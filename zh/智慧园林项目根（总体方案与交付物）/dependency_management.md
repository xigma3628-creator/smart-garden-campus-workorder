## 1. 使用的工具
- Node.js 后端服务（`server/`）：使用 npm + `package.json` 声明依赖，并生成 `package-lock.json` 锁定版本。
- Python 推理服务（`inference-service/`）：使用 PIP 风格的 `requirements.txt` 声明依赖。
- 两个子项目各自独立维护依赖，仓库根目录没有统一的包管理器配置（无根级 `package.json`、`pyproject.toml`、`go.mod`、`Cargo.toml` 等）。

## 2. 关键文件
- `server/package.json`：定义项目名称、模块类型、启动脚本及五个运行时依赖。
- `server/package-lock.json`：npm 生成的锁文件，用于保证依赖树可复现。
- `inference-service/requirements.txt`：Python 推理服务的依赖清单。

## 3. 架构与约定
- **按子项目隔离依赖**：后端 Express 服务与 FastAPI 推理服务各自拥有独立的依赖清单，不存在跨子项目的共享依赖或 workspace 机制。
- **后端采用 ES Module**：`server/package.json` 中 `"type": "module"` 启用 ESM；`main` 指向 `src/index.js`，并通过 `scripts.start` 和 `scripts.simulate` 暴露运行入口。
- **依赖范围**：`server/package.json` 仅列出运行时依赖（`dependencies`），未出现 `devDependencies`、`peerDependencies` 或 `optionalDependencies`，所有依赖均被当作运行时依赖安装。
- **Python 可选依赖以注释形式标注**：`requirements.txt` 中以 `# onnxruntime>=1.18` 的形式标记 ONNX 后端为可选依赖，由环境变量 `MODEL_BACKEND=onnx` 控制（见推理服务 README）。

## 4. 约定与约束
- **版本号策略**：`server/package.json` 中所有依赖均采用 `^` 前缀（如 `express ^5.1.0`、`aedes ^0.51.3`、`ws ^8.18.0`），表示允许安装满足语义化版本的次级更新；`inference-service/requirements.txt` 使用 `>=` 下限约束（如 `fastapi>=0.115`、`uvicorn>=0.30`、`ultralytics>=8.3`）。
- **锁文件用于复现**：`server/package-lock.json` 存在，表明 npm 安装应通过该锁文件确保可重复构建；但未见 CI 中校验其新鲜度（如 `npm ci`）的脚本证据。
- **私有注册表 / GOPRIVATE**：未发现 `.npmrc`、`.yarnrc`、`pip.conf`、`setup.cfg`、`pyproject.toml` 或任何私有 registry 配置，也未发现 `GOPRIVATE` 相关设置。依赖来源均为公开源。
- **无 vendoring**：仓库中没有 `vendor/`、`node_modules`（忽略）、`__pypackages__` 等第三方库内联目录，依赖均在安装时拉取。
- **桌面版交付物不包含源码依赖**：`智慧园林桌面版-交付/` 仅提供安装包说明与 SHA256 校验值，不携带源码或依赖清单，属于已打包产物。
- **阶段任务文档不含依赖声明**：`阶段包/` 下各期提示词/验收包为 Markdown 文档，不声明任何代码依赖。