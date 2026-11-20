## 1. 使用的方案

仓库中**没有引入任何日志框架**。后端（server）和推理服务（inference-service）均使用语言原生输出：

- Node.js 后端：仅使用 `console.log` / `console.warn` / `console.error`。
- Python 推理服务：未导入 `logging` 或第三方库，仅通过 FastAPI 默认行为与业务代码中的 `print()` 式调用输出。

`server/package.json` 的依赖只有 `express`、`aedes`、`mqtt`、`ws`，无任何日志相关包（如 `winston`、`pino`、`log4js`、`bunyan`、`morgan`、`debug`）。Python 端也未发现 logging 相关 import。

## 2. 关键文件

- `server/src/index.js`：进程启动入口，集中打印 HTTP/WS/MQTT/DB/AI/Drone 等子系统初始化信息。
- `server/src/ai.js`：AI 识别主流程，使用 `[ai]` 前缀记录类加载、降级、异常。
- `server/src/auth.js`：JWT 密钥缺失告警及初始账号创建日志。
- `server/src/kb.js`：知识库 FTS5/LIKE 退化、LLM 调用失败、种子导入等日志。
- `server/src/routers/drone.js`、`server/src/routers/kb.js`：路由层错误回退日志。
- `server/src/uploads.js`、`server/src/workorders.js`、`server/src/irrigation.js`、`server/src/rules.js`：各子模块业务日志。
- `inference-service/app.py`：推理服务，无显式日志输出。

## 3. 架构与约定

- **无统一 logger 模块**：每个 `.js` 文件直接调用 `console.*`，不存在共享的 logger 实例或中间件。
- **前缀约定**：日志行以方括号标注子系统来源，例如 `[ai]`、`[auth]`、`[kb]`、`[drone]`、`[uploads]`、`[workorders]`、`[irrigation]`、`[cleanup]`、`[http]`、`[ws]`、`[db]`、`[mqtt]`。这是一种人工约定的分类方式，并非由框架强制。
- **级别区分**：正常信息用 `console.log`，可恢复异常或配置缺失用 `console.warn`，不可恢复错误用 `console.error`。级别语义是代码风格约定，非运行时控制。
- **结构化字段**：除少数包含 JSON 片段（如 `err.message`、`err`）外，日志均为纯文本拼接，无统一的 JSON 结构化格式。
- **输出目标**：所有日志最终进入 Node.js/FastAPI 进程的 stdout/stderr；仓库中没有日志轮转、文件 sink、远程收集器的配置。

## 4. 约定与约束

- **约定（观察到的模式）**：每条业务日志以 `[子系统名]` 方括号前缀开头，便于在控制台输出中快速定位来源。
- **约定（观察到的模式）**：可预期的降级路径（FTS5 不可用、LLM 调用失败、classes_map 未加载、JWT_SECRET 未设置）统一使用 `console.warn` 输出告警。
- **约束（由代码结构体现）**：新增模块若要输出日志，需自行调用 `console.log/warn/error`，因为不存在共享 logger API；当前所有业务日志都内联在各自文件中，未见独立日志配置入口。
- **约束（由依赖清单体现）**：生产环境无法通过 npm 包升级日志框架版本，也无法通过配置文件切换输出目标——如需结构化日志或文件落盘，需要引入新依赖并改写现有 `console.*` 调用。