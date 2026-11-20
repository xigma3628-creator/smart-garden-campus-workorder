## 1. 采用的方式与约定

本仓库后端基于 Node.js + Express，**未引入专门的错误库（如 `http-errors`、`pino-error`）或自定义 Error 子类**。错误处理采用以下混合模式：

- **HTTP 层**：路由内直接通过 `res.status(code).json({ ok: false, error })` 返回业务错误；由 Express 默认错误处理器处理未捕获异常。
- **异步 I/O 层**：使用 `try/catch` 包裹外部调用（推理服务 HTTP、文件读写、数据库操作），失败时记录日志并返回降级结果或 `pending` 状态，而非向上抛出。
- **运行时保护**：全局 `setInterval` 任务、MQTT 消息回调等通过 `try/catch` + `console.error/warn` 吞掉异常，保证定时任务不因单次失败而终止。
- **降级策略**：AI 推理服务不可用时自动回退到模拟推理（`mockInference`），并通过 `consecutiveFailures` / `degraded` 标志广播 `ai.degraded` / `ai.recovered` 事件。

Python 端 inference-service 的 `app.py` 以 FastAPI 原生异常机制返回 JSON，后端通过 `res.ok` + `body.error` 统一消费。项目根目录无 Python 自定义异常类定义，仅依赖框架默认行为。

## 2. 关键文件

| 文件 | 职责 |
|---|---|
| `server/src/index.js` | Express 应用入口、CORS 中间件、全局定时器与 MQTT 回调的错误兜底 |
| `server/src/db.js` | SQLite 初始化（WAL、busy_timeout）、SQL 执行封装 |
| `server/src/ai.js` | AI 引擎、推理降级、告警联动、结果落库 |
| `server/src/routers/drone.js` | 无人机帧上传、推理补偿重试、multipart 解析错误处理 |
| `server/src/multipart.js` | 手写 multipart 解析，抛出自带 `{ status, message }` 对象 |
| `server/src/rules.js` | 告警规则引擎，内部 try/catch 吞错 |
| `inference-service/app.py` | FastAPI 推理服务，返回标准 JSON 错误体 |

## 3. 架构与约定

### 3.1 统一的响应契约
所有 REST 接口返回 `{ ok: boolean, ... }` 结构，错误路径附加 `error: string`。例如：

```js
return res.status(404).json({ ok: false, error: '任务不存在' })
```

业务逻辑函数也遵循相同契约，如 `createAiEngine().recognize()` 返回 `{ ok: true, result }` 或 `{ ok: false, error }`。

### 3.2 外部依赖调用的“可恢复失败”模式
对外部服务（推理服务、文件系统）的调用一律包裹 `try/catch`，失败后不中断主流程：

- `inferFrame`（drone.js）：推理失败返回 `{ ok: false, pending: true, error }`，帧保持 `pending` 等待补偿重试。
- `inferFromModel`（ai.js）：HTTP 错误转 `Error(body.error || \`推理服务 HTTP ${res.status}\`)`；JSON 解析失败用 `.catch(() => ({}))` 兜底。
- `startDroneInferenceRetry`：每轮只取 10 条 pending 帧，遇到失败即 `break`，避免空转。

### 3.3 降级与恢复
AI 引擎维护 `consecutiveFailures` 和 `degraded` 状态：连续失败 ≥ `DEGRADE_AFTER_FAILURES`（常量 2）后切换到 mock 推理并广播 `ai.degraded`；成功一次后清除计数并广播 `ai.recovered`。

### 3.4 定时任务与回调的全局保护
`index.js` 中多处 `try/catch` 包裹定时任务：

```js
setInterval(() => {
  try { cleanupOld(7) } catch (e) { console.error('[cleanup]', e.message) }
}, 60_000)

mqttClient.on('message', (topic) => {
  if (!topic.includes('/soil/')) return
  try { irrigation.evaluate() } catch (e) { console.error('[irrigation]', e.message) }
})
```

### 3.5 输入校验优先于业务逻辑
路由在执行业务前进行参数校验，直接返回 400，如 drone 帧上传对 `seq`、`lon/lat/alt`、`missionId`、`score` 的校验。

## 4. 观察到的约定与约束

- **REST 响应格式约定**：所有 API 返回 `{ ok, ... }` 结构，错误字段为字符串 `error`——由各路由一致实现，未见强制的类型检查。
- **外部调用必须可恢复**：推理服务、文件系统、数据库 I/O 的失败均被 `try/catch` 包裹，不允许未捕获异常传播到 Express 默认错误处理器作为常态路径。
- **AI 推理降级策略是硬编码**：`DEGRADE_AFTER_FAILURES = 2`，超时 `INFER_TIMEOUT_MS = 3000`（摄像头路径）/ `5000`（无人机路径），这些值以 `const` 声明且未在配置中暴露。
- **SQLite 健壮性设置**：`db.js` 固定启用 WAL 模式与 `busy_timeout = 5000`，防止并发写入阻塞。
- **multipart 解析错误携带 HTTP 状态**：`multipart.js` 抛出的错误对象形如 `{ status: number, message: string }`，drone 路由据此映射 `err.status || 400`。
- **告警去重**：AI 告警 key 使用前缀+资源标识（`ai-pest-${cameraId}`、`ai-pest-drone-${frameRow.id}`），利用 `upsertAlarm` 的 `key UNIQUE` 约束天然去重。
- **未定义的行为**：未发现全局 Express 错误中间件（如 `app.use((err, req, res, next) => {...})`），也未发现自定义 Error 子类或统一错误码枚举——错误主要通过字符串消息表达，类型上不做区分。

## 5. 不在当前范围内的部分

- 前端代码不在本仓库中，无法判断其错误呈现策略。
- Python inference-service 的具体异常类型未在检索范围内展开，但从后端消费方式看，它应返回包含 `error` 字段的 JSON。
