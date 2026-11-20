# Smart Garden · Project Wiki

Campus digital twin for a university campus — architecture and module knowledge base of the Smart Garden monitoring system.

- **Big-screen frontend**: Vue3 + Cesium (`digital-farmland`)
- **Backend service**: Node.js ESM + Express + embedded Aedes MQTT + SQLite (`server`)
- **AI inference**: Python FastAPI + YOLOv8/ONNX (`inference-service`)
- **Desktop edition**: Electron-packaged, works out of the box (offline satellite basemap included)

## Module navigation

| Module | Contents |
|---|---|
| [Project Root (master plan & deliverables)](Smart%20Garden%20Project%20Root/overview.md) | Five-layer architecture, build/config/dependency/error/logging systems |
| [Backend Service (server)](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/overview.md) | Express entry, MQTT ingestion, alarms & irrigation, unified API |
| [AI Recognition & Knowledge Base](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/AI%20Recognition%20and%20Knowledge%20Base/overview.md) | Pest/growth inference (real model + mock fallback), KB CRUD and LLM Q&A |
| [Drone Patrol & Device Simulator](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/Drone%20Patrol%20and%20Device%20Simulator/overview.md) | Mission CRUD, frame upload & inference backfill, MQTT simulation |
| [Backend Runtime](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/Backend%20Runtime%20(Express%20+%20Aedes%20MQTT%20+%20SQLite)/overview.md) | Process assembly, WebSocket broadcast, in-memory catalog + SQLite dual-write |
| [Server Acceptance & E2E Tests](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/Server%20Acceptance%20and%20E2E%20Tests/overview.md) | HTTP/MQTT/WS three-channel acceptance scripts |
| [Seed Data & Ops Scripts](Smart%20Garden%20Project%20Root/Backend%20Service%20(server)/Seed%20Data%20and%20Ops%20Scripts/overview.md) | KB import, tree archive generation, Q&A evaluation |
| [Frame Detection Inference Service](Smart%20Garden%20Project%20Root/Frame%20Detection%20Inference%20Service/overview.md) | FastAPI + YOLOv8/ONNX single-frame detection |
| [Desktop Deliverables & Phase Packages](Smart%20Garden%20Project%20Root/Desktop%20Deliverables%20and%20Phase%20Packages/overview.md) | Installer notes, SHA256 checksums, phase 6–11 task packages |

Each module has 5 knowledge cards: **Overview / Tech Stack / Architecture / Coding Conventions / Special Config & Commands** (the project root additionally carries 5 system cards: build, configuration, dependency, error handling, logging).

## Local preview

```bash
npx docsify-cli serve .
# or any static server, e.g. npx serve .
```

Open `http://localhost:3000/en/` for English; `http://localhost:3000/` for Chinese.