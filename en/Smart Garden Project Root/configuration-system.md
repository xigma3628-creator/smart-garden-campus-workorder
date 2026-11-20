## 1. Approach adopted

This repository uses no dedicated configuration framework (such as dotenv, config, or yaml/toml config files); it relies **entirely on process environment variables** for runtime configuration. Both the Express backend and the Python inference service read parameters via `process.env` / `os.environ`, and all defaults are hardcoded in the source.

- Backend: Node.js (`server/src/index.js`, etc.) uses the `process.env.X || default` pattern.
- Inference service: Python (`inference-service/model_adapter.py`) uses the `os.environ.get("X", default)` pattern.
- There are no external config files such as `.env`, `config.yaml`, `application.properties`, or `settings.json`; the scripts in `server/package.json` also run `node src/index.js` directly without `dotenv`.

## 2. Key files

| Module | File | Purpose |
|---|---|---|
| Backend entry | `server/src/index.js` | Loads PORT, MQTT_PORT, FRONTEND_DIST and other runtime parameters |
| Auth | `server/src/auth.js` | Loads JWT_SECRET |
| AI engine | `server/src/ai.js` | Loads INFERENCE_URL, AI_MODE, CLASSES_MAP_PATH, AI_SCAN_SECONDS |
| Knowledge-base LLM | `server/src/kb.js` | Loads KB_LLM_URL / KB_LLM_KEY / KB_LLM_MODEL / KB_SEED_FILE |
| Camera frame grabbing | `server/src/frame-grabber.js` | Loads FFMPEG_PATH / FFPROBE_PATH |
| Database path | `server/src/db.js` | Loads GARDEN_DATA_DIR |
| Drone router | `server/src/routers/drone.js` | Reuses INFERENCE_URL |
| Inference adapter | `inference-service/model_adapter.py` | Loads MODEL_PATH / CONF_THRESHOLD / MODEL_BACKEND |

## 3. Architecture and conventions

### 3.1 Variable naming convention
All configurable items are UPPER_SNAKE_CASE process environment variables in the form `ENV_VAR_NAME` (e.g. `INFERENCE_URL`, `KB_LLM_URL`, `MODEL_BACKEND`, `CONF_THRESHOLD`).

### 3.2 Value-reading patterns
- Node.js side uniformly uses `process.env.X || default`, or parses after `(process.env.X || '').trim()`.
- Python side uniformly uses `os.environ.get("X", default)`; string variables are `.strip()`ed, numeric ones converted with `float()` / `Number()`.
- The sensitive value (JWT_SECRET) logs a warning when missing and falls back to the development default `'garden-dev-secret-change-me'`.

### 3.3 Layered structure
Configuration is not loaded centrally — each subsystem reads the environment variables it needs on demand, a "decentralized, per-module self-service" pattern. Adding a new config item usually means adding one `process.env.X` read in the relevant module.

### 3.4 Relationship to deployment
- The embedded MQTT broker listens on `MQTT_PORT`; a comment notes it can be replaced by EMQX in production (only the connection address changes).
- `FRONTEND_DIST` controls whether the frontend static directory is mounted (desktop packaging mode vs development mode).
- `AI_MODE=mock` switches AI inference to the mock implementation.
- `KB_SEED_FILE` auto-imports seed data at startup.
- The inference service switches between yolo / onnx backends via `MODEL_BACKEND`, sets the weights path via `MODEL_PATH`, and tunes the detection confidence threshold via `CONF_THRESHOLD`.

## 4. Observed conventions and constraints

- **No central configuration service or config file**: there is no `config/` directory, `.env*` file, or any YAML/TOML configuration in the repository. Configuration is environment variables.
- **Sensitive configuration must be injected via environment variables**: `JWT_SECRET` only prints a warning and falls back to the development default when unset, prompting users to set it explicitly in production.
- **All defaults are inline in code**: every `process.env` / `os.environ.get` call carries a default, so services still start with no environment variables set.
- **String environment variables are always trimmed**: `kb.js` calls `.trim()` on `KB_LLM_URL`, `KB_LLM_KEY`, and `KB_LLM_MODEL` individually to avoid request failures caused by trailing whitespace.
- **Numeric environment variables are explicitly coerced**: PORT, MQTT_PORT, AI_SCAN_SECONDS, etc. all pass through `Number()`, ensuring later comparisons and timer parameters are numbers.
- **Optional URLs are tested against the empty string**: when `KB_LLM_URL` is empty, LLM functionality is skipped (`if (!config) return null`), so the knowledge-base feature can be disabled independently.
- **The inference service default port is fixed at 3101**: `INFERENCE_URL` defaults to `http://127.0.0.1:3101`, as verified by the start command in the README.

This design is simple, direct, and dependency-free — a good fit for the current scale of two sub-services (Express + FastAPI). However, it lacks schema validation, hierarchical merging, and hot reload, and new config items must be declared scattered across modules, which hurts global discoverability.