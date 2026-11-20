## 1. Approach adopted

The repository **introduces no logging framework**. Both the backend (server) and the inference service (inference-service) use native language output:

- Node.js backend: only `console.log` / `console.warn` / `console.error`.
- Python inference service: no `logging` or third-party library imported; output comes only from FastAPI's default behavior and `print()`-style calls in business code.

`server/package.json` depends only on `express`, `aedes`, `mqtt`, `ws` — no logging-related packages (such as `winston`, `pino`, `log4js`, `bunyan`, `morgan`, `debug`). No logging-related imports were found on the Python side either.

## 2. Key files

- `server/src/index.js`: process start entry; centrally prints initialization info for HTTP/WS/MQTT/DB/AI/Drone subsystems.
- `server/src/ai.js`: AI recognition main flow; logs class loading, degradation, and exceptions with the `[ai]` prefix.
- `server/src/auth.js`: JWT secret-missing warning and initial account creation logs.
- `server/src/kb.js`: knowledge-base FTS5/LIKE fallback, LLM call failures, seed import logs.
- `server/src/routers/drone.js`, `server/src/routers/kb.js`: router-level error fallback logs.
- `server/src/uploads.js`, `server/src/workorders.js`, `server/src/irrigation.js`, `server/src/rules.js`: business logs of each submodule.
- `inference-service/app.py`: inference service; no explicit log output.

## 3. Architecture and conventions

- **No unified logger module**: every `.js` file calls `console.*` directly; there is no shared logger instance or middleware.
- **Prefix convention**: log lines mark the subsystem source in square brackets, e.g. `[ai]`, `[auth]`, `[kb]`, `[drone]`, `[uploads]`, `[workorders]`, `[irrigation]`, `[cleanup]`, `[http]`, `[ws]`, `[db]`, `[mqtt]`. This is a human-agreed classification, not enforced by a framework.
- **Level distinction**: normal info uses `console.log`, recoverable exceptions or missing configuration use `console.warn`, unrecoverable errors use `console.error`. Level semantics are a code-style convention, not runtime control.
- **Structured fields**: apart from a few JSON fragments (such as `err.message`, `err`), logs are plain-text concatenation with no unified JSON structured format.
- **Output destination**: all logs end up on the Node.js/FastAPI process stdout/stderr; the repository has no log rotation, file sink, or remote collector configuration.

## 4. Conventions and constraints

- **Convention (observed pattern)**: every business log line starts with a `[subsystem]` bracket prefix, making it easy to locate the source in console output.
- **Convention (observed pattern)**: expected degradation paths (FTS5 unavailable, LLM call failure, classes_map not loaded, JWT_SECRET unset) uniformly emit warnings via `console.warn`.
- **Constraint (reflected in code structure)**: a new module that wants to log must call `console.log/warn/error` itself, because no shared logger API exists; all business logs are currently inline in their own files, with no separate logging configuration entry.
- **Constraint (reflected in the dependency manifest)**: in production you cannot upgrade a logging framework via npm or switch output destinations via a config file — structured logging or file persistence would require new dependencies and rewriting the existing `console.*` calls.