## 1. Tools in use
- Node.js backend service (`server/`): npm + `package.json` for dependency declarations, with `package-lock.json` generated to lock versions.
- Python inference service (`inference-service/`): PIP-style `requirements.txt` for dependency declarations.
- The two subprojects maintain dependencies independently; the repository root has no unified package-manager configuration (no root-level `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.).

## 2. Key files
- `server/package.json`: defines the project name, module type, start scripts, and five runtime dependencies.
- `server/package-lock.json`: npm-generated lockfile ensuring a reproducible dependency tree.
- `inference-service/requirements.txt`: dependency manifest of the Python inference service.

## 3. Architecture and conventions
- **Dependencies isolated per subproject**: the Express backend and the FastAPI inference service each own an independent dependency manifest; there is no cross-subproject shared dependency or workspace mechanism.
- **Backend uses ES Modules**: `"type": "module"` in `server/package.json` enables ESM; `main` points to `src/index.js`, and `scripts.start` / `scripts.simulate` expose the run entries.
- **Dependency scope**: `server/package.json` lists only runtime dependencies (`dependencies`); there are no `devDependencies`, `peerDependencies`, or `optionalDependencies` — everything is installed as a runtime dependency.
- **Python optional dependencies are marked by comments**: `requirements.txt` marks the ONNX backend as optional with `# onnxruntime>=1.18`, controlled by the environment variable `MODEL_BACKEND=onnx` (see the inference service README).

## 4. Conventions and constraints
- **Version policy**: all dependencies in `server/package.json` use the `^` prefix (e.g. `express ^5.1.0`, `aedes ^0.51.3`, `ws ^8.18.0`), allowing semver-compatible minor updates; `inference-service/requirements.txt` uses `>=` lower bounds (e.g. `fastapi>=0.115`, `uvicorn>=0.30`, `ultralytics>=8.3`).
- **Lockfile for reproducibility**: `server/package-lock.json` exists, indicating npm installs should go through the lockfile for repeatable builds; however, no CI script evidence of freshness checks (such as `npm ci`) was found.
- **Private registries / GOPRIVATE**: no `.npmrc`, `.yarnrc`, `pip.conf`, `setup.cfg`, `pyproject.toml`, or any private registry configuration was found, nor any `GOPRIVATE`-related settings. All dependencies come from public sources.
- **No vendoring**: the repository contains no `vendor/`, `node_modules` (ignored), `__pypackages__`, or other inlined third-party library directories; dependencies are fetched at install time.
- **Desktop deliverables carry no source dependencies**: `智慧园林桌面版-交付/` provides only installer instructions and SHA256 checksums — no source code or dependency manifests; it is a packaged artifact.
- **Phase task documents declare no dependencies**: the per-phase prompts/acceptance packages under `阶段包/` are Markdown documents and declare no code dependencies.