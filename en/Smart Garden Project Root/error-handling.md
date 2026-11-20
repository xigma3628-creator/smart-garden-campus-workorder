## 1. Approach and conventions adopted

The backend is Node.js + Express and **introduces no dedicated error library (such as `http-errors`, `pino-error`) or custom Error subclasses**. Error handling follows a mixed pattern:

- **HTTP layer**: routes return business errors directly via `res.status(code).json({ ok: false, error })`; uncaught exceptions fall through to the Express default error handler.
- **Async I/O layer**: external calls (inference service HTTP, file I/O, database operations) are wrapped in `try/catch`; on failure the code logs and returns a degraded result or a `pending` state instead of rethrowing.
- **Runtime protection**: global `setInterval` tasks, MQTT message callbacks, etc. swallow exceptions via `try/catch` + `console.error/warn`, so timers never terminate because of a single failure.
- **Degradation strategy**: when the AI inference service is unavailable, the engine automatically falls back to mock inference (`mockInference`) and broadcasts `ai.degraded` / `ai.recovered` events via the `consecutiveFailures` / `degraded` flags.

On the Python side, inference-service's `app.py` returns JSON through FastAPI's native exception mechanism, and the backend consumes it uniformly via `res.ok` + `body.error`. There are no custom Python exception classes at the project root — only framework defaults.

## 2. Key files

| File | Responsibility |
|---|---|
| `server/src/index.js` | Express app entry, CORS middleware, error backstops for global timers and MQTT callbacks |
| `server/src/db.js` | SQLite initialization (WAL, busy_timeout), SQL execution wrappers |
| `server/src/ai.js` | AI engine, inference degradation, alarm linkage, result persistence |
| `server/src/routers/drone.js` | Drone frame upload, inference compensation retries, multipart parse error handling |
| `server/src/multipart.js` | Hand-written multipart parser; throws objects carrying `{ status, message }` |
| `server/src/rules.js` | Alarm rule engine; internal try/catch swallows errors |
| `inference-service/app.py` | FastAPI inference service; returns standard JSON error bodies |

## 3. Architecture and conventions

### 3.1 Unified response contract
All REST endpoints return an `{ ok: boolean, ... }` structure; error paths add `error: string`. For example:

```js
return res.status(404).json({ ok: false, error: '任务不存在' })
```

Business-logic functions follow the same contract — e.g. `createAiEngine().recognize()` returns `{ ok: true, result }` or `{ ok: false, error }`.

### 3.2 "Recoverable failure" pattern for external calls
Calls to external services (inference service, file system) are always wrapped in `try/catch`; failures never interrupt the main flow:

- `inferFrame` (drone.js): on inference failure returns `{ ok: false, pending: true, error }`; the frame stays `pending` awaiting compensation retries.
- `inferFromModel` (ai.js): HTTP errors are converted to `Error(body.error || \`推理服务 HTTP ${res.status}\`)`; JSON parse failures are backstopped with `.catch(() => ({}))`.
- `startDroneInferenceRetry`: each round takes only 10 pending frames and `break`s on the first failure to avoid spinning.

### 3.3 Degradation and recovery
The AI engine maintains `consecutiveFailures` and `degraded` state: after ≥ `DEGRADE_AFTER_FAILURES` (constant 2) consecutive failures it switches to mock inference and broadcasts `ai.degraded`; one success clears the counter and broadcasts `ai.recovered`.

### 3.4 Global protection for timers and callbacks
`index.js` wraps timers in `try/catch` in several places:

```js
setInterval(() => {
  try { cleanupOld(7) } catch (e) { console.error('[cleanup]', e.message) }
}, 60_000)

mqttClient.on('message', (topic) => {
  if (!topic.includes('/soil/')) return
  try { irrigation.evaluate() } catch (e) { console.error('[irrigation]', e.message) }
})
```

### 3.5 Input validation before business logic
Routes validate parameters before executing business logic and return 400 directly — e.g. the drone frame upload validates `seq`, `lon/lat/alt`, `missionId`, and `score`.

## 4. Observed conventions and constraints

- **REST response format convention**: all APIs return `{ ok, ... }` with a string `error` field — implemented consistently by each route, with no enforced type checking observed.
- **External calls must be recoverable**: inference service, file system, and database I/O failures are all wrapped in `try/catch`; uncaught exceptions propagating to the Express default error handler are not allowed as a normal path.
- **AI inference degradation policy is hardcoded**: `DEGRADE_AFTER_FAILURES = 2`, timeouts `INFER_TIMEOUT_MS = 3000` (camera path) / `5000` (drone path); these are declared as `const` and not exposed in configuration.
- **SQLite robustness settings**: `db.js` always enables WAL mode and `busy_timeout = 5000` to prevent concurrent-write blocking.
- **Multipart parse errors carry HTTP status**: errors thrown by `multipart.js` have the shape `{ status: number, message: string }`; the drone router maps them via `err.status || 400`.
- **Alarm deduplication**: AI alarm keys use prefix + resource identifier (`ai-pest-${cameraId}`, `ai-pest-drone-${frameRow.id}`), naturally deduplicated by the `key UNIQUE` constraint in `upsertAlarm`.
- **Undefined behavior**: no global Express error middleware (such as `app.use((err, req, res, next) => {...})`) was found, nor custom Error subclasses or a unified error-code enum — errors are expressed mainly as string messages with no type-level distinction.

## 5. Out of current scope

- The frontend code is not in this repository, so its error-presentation strategy cannot be judged.
- The specific exception types of the Python inference-service were not expanded within the search scope, but judging from how the backend consumes it, it should return JSON containing an `error` field.