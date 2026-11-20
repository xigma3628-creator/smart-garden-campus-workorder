With `index.js` as the single entry point, three parallel runtimes are assembled uniformly:
- HTTP REST: the app is built with `express()`, routers are mounted by business phase (auth/workorder/kb/tree-assets/reports/drone/uploads/device-registry), and the `/ws` WebSocket channel is registered;
- MQTT ingestion layer: an embedded `aedes` broker (port configured by `MQTT_PORT`, default 1883) also acts as a client subscribing to `smart-garden/#`, parsing sensor status into the `state.js` in-memory Map and persisting to `sensor_data` via `db.js`;
- Scheduled tasks: offline detection (`makeOfflineWatcher`) every 5s, cleanup of sensor data older than 7 days every 60s.

Internal dependencies are layered top-down:
- The router layer (`routes.js`) exposes generic REST (devices, history, alarms, camera PTZ, AI recognition, irrigation control) and receives runtime capabilities via the `setPublishControl` / `setClientCounter` / `setAi` / `setIrrigation` callback injections;
- Domain engines (`rules.js` threshold alarms, `irrigation.js` closed-loop policy, `ai.js` inference pipeline, `uploads.js`+`multipart.js` image upload, `tree-assets.js` tree archives) are independent of the HTTP layer and depend only on `db.js` and `state.js`;
- The persistence layer (`db.js`) is based on Node's built-in `node:sqlite` `DatabaseSync`, using WAL mode and prepared statements; initial device catalog data comes from `devices.json` (ESM `with { type: 'json' }` import). `state.js` maintains the in-memory `devices` Map and provides snapshot export.

Cross-module collaboration is done via callback injection (not global singletons): after startup, `index.js` injects `broadcast`, `irrigation`, and `aiEngine` into the routes and rules, ensuring each submodule can be tested or replaced in isolation.