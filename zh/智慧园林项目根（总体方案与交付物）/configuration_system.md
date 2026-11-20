## 1. 采用的方案

本仓库没有使用任何专门的配置框架（如 dotenv、config、yaml/toml 配置文件），而是**完全依赖进程环境变量**来管理运行时配置。后端 Express 服务与 Python 推理服务均通过 `process.env` / `os.environ` 读取参数，所有默认值均在代码中以硬编码形式给出。

- 后端：Node.js (`server/src/index.js` 等) 使用 `process.env.X || default` 模式。
- 推理服务：Python (`inference-service/model_adapter.py`) 使用 `os.environ.get("X", default)` 模式。
- 无 `.env`、`config.yaml`、`application.properties`、`settings.json` 等外部配置文件；`server/package.json` 中的脚本也直接执行 `node src/index.js`，不引入 `dotenv`。

## 2. 关键文件

| 模块 | 文件 | 作用 |
|---|---|---|
| 后端入口 | `server/src/index.js` | 加载 PORT、MQTT_PORT、FRONTEND_DIST 等运行参数 |
| 鉴权 | `server/src/auth.js` | 加载 JWT_SECRET |
| AI 引擎 | `server/src/ai.js` | 加载 INFERENCE_URL、AI_MODE、CLASSES_MAP_PATH、AI_SCAN_SECONDS |
| 知识库 LLM | `server/src/kb.js` | 加载 KB_LLM_URL / KB_LLM_KEY / KB_LLM_MODEL / KB_SEED_FILE |
| 摄像头抓取 | `server/src/frame-grabber.js` | 加载 FFMPEG_PATH / FFPROBE_PATH |
| 数据库路径 | `server/src/db.js` | 加载 GARDEN_DATA_DIR |
| 无人机路由 | `server/src/routers/drone.js` | 复用 INFERENCE_URL |
| 推理适配层 | `inference-service/model_adapter.py` | 加载 MODEL_PATH / CONF_THRESHOLD / MODEL_BACKEND |

## 3. 架构与约定

### 3.1 变量命名规范
所有可配置项均为大写蛇形命名的进程环境变量，采用 `ENV_VAR_NAME` 形式（例如 `INFERENCE_URL`、`KB_LLM_URL`、`MODEL_BACKEND`、`CONF_THRESHOLD`）。

### 3.2 取值模式
- Node.js 端统一使用 `process.env.X || default` 或 `(process.env.X || '').trim()` 后解析。
- Python 端统一使用 `os.environ.get("X", default)`，字符串型变量会 `.strip()`，数值型用 `float()` / `Number()` 转换。
- 敏感值（JWT_SECRET）在缺失时输出警告日志并回退到开发默认值 `'garden-dev-secret-change-me'`。

### 3.3 分层结构
配置不是集中式加载的——每个子系统按需读取自己需要的环境变量，属于“分散式、按模块自取”的模式。新增配置项通常只需在对应模块中增加一行 `process.env.X` 读取。

### 3.4 与部署的关系
- 嵌入式 MQTT Broker 监听 `MQTT_PORT`，注释说明生产环境可替换为 EMQX（仅改连接地址）。
- `FRONTEND_DIST` 控制是否挂载前端静态目录（桌面打包模式 vs 开发模式）。
- `AI_MODE=mock` 切换 AI 推理为模拟实现。
- `KB_SEED_FILE` 启动时自动导入种子数据。
- 推理服务通过 `MODEL_BACKEND` 切换 yolo / onnx 后端，`MODEL_PATH` 指定权重路径，`CONF_THRESHOLD` 调整检测置信度阈值。

## 4. 观察到的约定与约束

- **不存在统一的配置中心或配置文件**：仓库中没有 `config/` 目录、`.env*` 文件或任何 YAML/TOML 配置。配置即环境变量。
- **敏感配置必须通过环境变量注入**：`JWT_SECRET` 仅在未设置时打印警告并回退到开发默认值，提示使用者在生产环境中显式设置。
- **默认值全部内联于代码**：每个 `process.env` / `os.environ.get` 调用都附带默认值，保证服务在无环境变量时仍可启动。
- **字符串型环境变量一律 trim**：`kb.js` 中对 `KB_LLM_URL`、`KB_LLM_KEY`、`KB_LLM_MODEL` 分别调用 `.trim()`，避免尾部空白导致请求失败。
- **数值型环境变量强制类型转换**：PORT、MQTT_PORT、AI_SCAN_SECONDS 等均通过 `Number()` 转换，确保后续比较和定时任务参数为数字。
- **可选 URL 以空串判断**：`KB_LLM_URL` 为空时跳过 LLM 功能（`if (!config) return null`），使知识库功能可独立禁用。
- **推理服务默认端口固定为 3101**：`INFERENCE_URL` 默认值为 `http://127.0.0.1:3101`，由 README 中的启动命令验证。

该设计简单直接、零依赖，适合当前两个子服务（Express + FastAPI）的规模；但缺乏 schema 校验、层级合并、热重载能力，且新配置项需在各模块散落声明，不利于全局发现。