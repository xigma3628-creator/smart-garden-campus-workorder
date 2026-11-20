## 1. Build system in use

The repository uses a **multi-language, unorchestrated** lightweight build approach; each submodule manages its own dependencies and start scripts:

- **Backend service (`server/`)**: Node.js / Express; dependencies declared in `package.json`, with npm scripts providing two commands: `start` (`node src/index.js`) and `simulate` (device simulator).
- **Inference service (`inference-service/`)**: Python / FastAPI; runtime dependencies pinned in `requirements.txt` (fastapi, uvicorn, ultralytics, etc.). There is no separate build step — an external container or environment runs `app.py` directly.
- **Desktop deliverables (`智慧园林桌面版-交付/`)**: the prebuilt binary installer `智慧园林监测系统-1.4.0-x64-安装包.exe`, shipped with a SHA256 checksum file. The Electron shell project lives outside the repository at `sg-desktop`, version `1.4.0`, repackaged via `npm run dist`.

## 2. Key files

| Path | Purpose |
|---|---|
| `server/package.json` | Backend npm manifest; declares Express/MQTT/WS dependencies and start scripts |
| `server/package-lock.json` | npm dependency-tree lockfile |
| `inference-service/requirements.txt` | Python inference service runtime dependencies |
| `智慧园林桌面版-交付/安装说明.txt` | Desktop install/usage instructions; also records the source project location and repackaging procedure |
| `智慧园林桌面版-交付/校验值-SHA256.txt` | Installer integrity checksums |
| `.gitignore` | Git ignore rules (no dedicated build-artifact ignore entries observed) |

## 3. Architecture and conventions

- **Module decoupling**: each deployable unit (backend, inference service, desktop) maintains its own dependency manifest; there are no cross-module shared build scripts or Makefile.
- **Decentralized versioning**: `server/package.json` is fixed at version `0.1.0`; the desktop installer version `1.4.0` is controlled by the external Electron project. The two version tracks are not synchronized — they are independent release channels.
- **Build is run**: the backend has no build phase; `npm start` executes the source directly. The inference service likewise has no compilation step.
- **Desktop packaging is external**: the Electron shell project is not in this repository; the install instructions explicitly point to the local path `sg-desktop`. This repository therefore only produces source code and the final binary deliverables.
- **Out-of-the-box**: the desktop edition bundles SQLite, MQTT, and REST services plus a basemap tile proxy, and needs no extra database or message middleware.

## 4. Conventions and constraints

- Backend dependency changes must update `server/package.json` and regenerate `package-lock.json` via npm (the lockfile is committed, indicating the dependency tree is controlled).
- Inference service dependencies are managed via `requirements.txt`, where the ONNX runtime is commented out as optional (enabled only with `MODEL_BACKEND=onnx`).
- The desktop installer defaults to the current user directory and **requires no administrator privileges** (install instructions, item 1).
- The installer is not code-signed; SmartScreen warnings are expected behavior, and users should verify integrity via `校验值-SHA256.txt` (install instructions, item 2).
- Repackaging must follow the order in install instructions item 4: build the frontend with `npm run build` in `digital-farmland`, use the backend source in `server`, then bump the version in `sg-desktop` and run `npm run dist`.
- The repository root has **no** Makefile, Dockerfile, CI configuration, or unified build/test orchestration script; all build and release activities happen separately in the submodules or external projects listed above.