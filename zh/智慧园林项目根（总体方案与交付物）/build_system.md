## 1. 使用的构建系统

仓库采用**多语言、无统一编排**的轻量构建方式，按子模块各自管理依赖与启动脚本：

- **后端服务 (`server/`)**：基于 Node.js / Express，通过 `package.json` 声明依赖，使用 npm scripts 提供 `start`（`node src/index.js`）和 `simulate`（设备模拟器）两个命令。
- **推理服务 (`inference-service/`)**：基于 Python / FastAPI，通过 `requirements.txt` 锁定运行时依赖（fastapi、uvicorn、ultralytics 等），没有独立构建步骤，由外部容器或环境直接运行 `app.py`。
- **桌面版交付物 (`智慧园林桌面版-交付/`)**：已预编译的二进制安装包 `智慧园林监测系统-1.4.0-x64-安装包.exe`，附带 SHA256 校验文件。Electron 壳工程位于仓库之外的 `sg-desktop`，版本号为 `1.4.0`，通过 `npm run dist` 重新出包。

## 2. 关键文件

| 路径 | 作用 |
|---|---|
| `server/package.json` | 后端 npm 项目清单，声明 Express/MQTT/WS 依赖及启动脚本 |
| `server/package-lock.json` | npm 依赖树锁文件 |
| `inference-service/requirements.txt` | Python 推理服务运行时依赖 |
| `智慧园林桌面版-交付/安装说明.txt` | 桌面版安装/使用说明，同时记录源工程位置与重新打包流程 |
| `智慧园林桌面版-交付/校验值-SHA256.txt` | 安装包完整性校验值 |
| `.gitignore` | Git 忽略规则（未见专门的构建产物忽略条目） |

## 3. 架构与约定

- **模块解耦**：每个可部署单元（后端、推理服务、桌面版）各自维护其依赖清单，不存在跨模块共享的构建脚本或 Makefile。
- **版本分散**：`server/package.json` 中版本号固定为 `0.1.0`；桌面版安装包版本 `1.4.0` 由外部 Electron 工程控制，两者版本不同步，属于独立的发布口径。
- **构建即运行**：后端没有 build 阶段，`npm start` 直接执行源码；推理服务同样无编译环节。
- **桌面版打包外置**：Electron 壳工程不在本仓库内，安装说明明确指向本地路径 `sg-desktop`，因此本仓库只负责产出源码与最终二进制交付物。
- **开箱即用**：桌面版内置 SQLite、MQTT、REST 服务与底图瓦片代理，无需额外安装数据库或消息中间件。

## 4. 约定与约束

- 后端依赖变更需更新 `server/package.json`，并通过 npm 生成 `package-lock.json`（仓库已提交锁文件，表明依赖树是受控的）。
- 推理服务依赖通过 `requirements.txt` 管理，其中 ONNX 运行时被注释为可选（需设置 `MODEL_BACKEND=onnx` 才启用）。
- 桌面版安装包安装路径默认在当前用户目录，**无需管理员权限**（安装说明第 1 条）。
- 安装包未做代码签名，SmartScreen 提示属预期行为，用户需通过 `校验值-SHA256.txt` 验证完整性（安装说明第 2 条）。
- 重新打包必须遵循安装说明第 4 条的顺序：前端在 `digital-farmland` 执行 `npm run build`，后端在 `server` 源码，最后到 `sg-desktop` 修改版本号后执行 `npm run dist`。
- 仓库根目录**没有** Makefile、Dockerfile、CI 配置文件或统一的构建/测试编排脚本；所有构建与发布活动均在上述子模块或外部工程中分别进行。