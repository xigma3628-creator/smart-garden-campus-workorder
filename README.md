# 智慧园林校园版带工单派遣 · 双语项目 Wiki

**Smart Garden Campus Edition (Work-Order Dispatch) — Bilingual Project Wiki**

面向**高校校园**数字孪生的智慧园林监测系统 Wiki：设备监测与告警、视频监控、AI 病虫害识别、园林知识库问答、工单派遣、苗木档案、报表驾驶舱、无人机巡检回放、设备自助注册。

本仓库是它的**项目自述与知识库**：中文 50 篇 + English 50 篇知识卡，按模块组织，docsify 站点可直接在线阅读。

- 中文站：<https://xigma3628-creator.github.io/smart-garden-campus-workorder/>
- English site：<https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/>
- 语言切换：站点侧栏顶部

## 系统组成 / System Composition

| 部分 | 技术栈 |
| --- | --- |
| 前端大屏 | Vue3 + Vite + Cesium + Element Plus（hash 路由，离线卫星底图） |
| 后端 | Node.js Express 5 + 内嵌 Aedes MQTT(1883) + ws + SQLite（REST :3100） |
| 推理服务 | FastAPI + YOLOv8n（:3101，不可用时自动降级为模拟） |
| 设备模拟器 | 18 台设备 2 秒级 MQTT 上报（土壤/气象/水质/灌溉/监控/虫情） |
| 桌面版 | Electron 45 + electron-builder（安装包 1.4.0） |
| 工人端 | H5 多页（待办/上报/上传）+ Capacitor Android APK |

## Wiki 目录 / Contents

每个模块 5 张卡：概述 / 技术栈 / 架构设计 / 编码规范 / 特殊配置与命令（项目根模块另有构建、配置、依赖、错误处理、日志 5 张）。

**中文（50 篇）**

| 模块 | 内容 | 源码 |
| --- | --- | --- |
| [项目根（总体方案与交付物）](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%A6%82%E8%BF%B0) | 总体方案、五层架构、六~十二期路线图、交付物索引 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89) |
| [后端服务（server）](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%A6%82%E8%BF%B0) | REST/MQTT/WS 网关，告警、工单、知识库、苗木、报表、设备注册 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89) |
| [AI 识别与园林知识库服务](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/AI%20%E8%AF%86%E5%88%AB%E4%B8%8E%E5%9B%AD%E6%9E%97%E7%9F%A5%E8%AF%86%E5%BA%93%E6%9C%8D%E5%8A%A1/%E6%A6%82%E8%BF%B0) | 病虫害/长势推理（可降级 mock）、知识条目 CRUD 与 FTS5 检索、问答 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/AI%20%E8%AF%86%E5%88%AB%E4%B8%8E%E5%9B%AD%E6%9E%97%E7%9F%A5%E8%AF%86%E5%BA%93%E6%9C%8D%E5%8A%A1) |
| [无人机巡检与设备模拟器](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%97%A0%E4%BA%BA%E6%9C%BA%E5%B7%A1%E6%A3%80%E4%B8%8E%E8%AE%BE%E5%A4%87%E6%A8%A1%E6%8B%9F%E5%99%A8/%E6%A6%82%E8%BF%B0) | 巡飞任务状态机、帧上传与回填、18 台设备 2s 上报模拟 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%97%A0%E4%BA%BA%E6%9C%BA%E5%B7%A1%E6%A3%80%E4%B8%8E%E8%AE%BE%E5%A4%87%E6%A8%A1%E6%8B%9F%E5%99%A8) |
| [后端运行时（Express + Aedes MQTT + SQLite）](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E8%BF%90%E8%A1%8C%E6%97%B6%EF%BC%88Express%20%2B%20Aedes%20MQTT%20%2B%20SQLite%EF%BC%89/%E6%A6%82%E8%BF%B0) | 应用启动、内嵌 MQTT、WS 广播、内存设备目录、SQLite 时序存储 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E8%BF%90%E8%A1%8C%E6%97%B6%EF%BC%88Express%20%2B%20Aedes%20MQTT%20%2B%20SQLite%EF%BC%89) |
| [服务端验收与端到端测试脚本集](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%AA%8C%E6%94%B6%E4%B8%8E%E7%AB%AF%E5%88%B0%E7%AB%AF%E6%B5%8B%E8%AF%95%E8%84%9A%E6%9C%AC%E9%9B%86/%E6%A6%82%E8%BF%B0) | 告警、无人机、灌溉、工单等场景的 HTTP/MQTT/WS 集成验证 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%AA%8C%E6%94%B6%E4%B8%8E%E7%AB%AF%E5%88%B0%E7%AB%AF%E6%B5%8B%E8%AF%95%E8%84%9A%E6%9C%AC%E9%9B%86) |
| [种子数据与运维辅助脚本集](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E7%A7%8D%E5%AD%90%E6%95%B0%E6%8D%AE%E4%B8%8E%E8%BF%90%E7%BB%B4%E8%BE%85%E5%8A%A9%E8%84%9A%E6%9C%AC%E9%9B%86/%E6%A6%82%E8%BF%B0) | 知识库导入、苗木种子、无人机模拟、问答评测等一次性脚本 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%90%8E%E7%AB%AF%E6%9C%8D%E5%8A%A1%EF%BC%88server%EF%BC%89/%E7%A7%8D%E5%AD%90%E6%95%B0%E6%8D%AE%E4%B8%8E%E8%BF%90%E7%BB%B4%E8%BE%85%E5%8A%A9%E8%84%9A%E6%9C%AC%E9%9B%86) |
| [帧级检测推理服务](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%B8%A7%E7%BA%A7%E6%A3%80%E6%B5%8B%E6%8E%A8%E7%90%86%E6%9C%8D%E5%8A%A1/%E6%A6%82%E8%BF%B0) | FastAPI + YOLOv8 单帧检测，权重缺失自动降级 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E5%B8%A7%E7%BA%A7%E6%A3%80%E6%B5%8B%E6%8E%A8%E7%90%86%E6%9C%8D%E5%8A%A1) |
| [桌面版交付物与分阶段任务包](https://xigma3628-creator.github.io/smart-garden-campus-workorder/#/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E6%A1%8C%E9%9D%A2%E7%89%88%E4%BA%A4%E4%BB%98%E7%89%A9%E4%B8%8E%E5%88%86%E9%98%B6%E6%AE%B5%E4%BB%BB%E5%8A%A1%E5%8C%85/%E6%A6%82%E8%BF%B0) | 安装包说明与 SHA256、六~十一期任务包/验收包/提示词三件套 | [目录](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/zh/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E9%A1%B9%E7%9B%AE%E6%A0%B9%EF%BC%88%E6%80%BB%E4%BD%93%E6%96%B9%E6%A1%88%E4%B8%8E%E4%BA%A4%E4%BB%98%E7%89%A9%EF%BC%89/%E6%99%BA%E6%85%A7%E5%9B%AD%E6%9E%97%E6%A1%8C%E9%9D%A2%E7%89%88%E4%BA%A4%E4%BB%98%E7%89%A9%E4%B8%8E%E5%88%86%E9%98%B6%E6%AE%B5%E4%BB%BB%E5%8A%A1%E5%8C%85) |

**English (50 pages)**

| Module | Scope | Source |
| --- | --- | --- |
| [Project Root (Master Plan & Deliverables)](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/overview) | Master plan, five-layer architecture, phase roadmap, deliverables | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root) |
| [Backend Service (server)](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/overview) | REST/MQTT/WS gateway: alarms, work orders, KB, trees, reports, device registry | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29) |
| [AI Recognition & Knowledge Base](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/AI%20Recognition%20and%20Knowledge%20Base/overview) | Pest/growth inference with mock fallback, KB CRUD, FTS5 retrieval, QA | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/AI%20Recognition%20and%20Knowledge%20Base) |
| [Drone Patrol & Device Simulator](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Drone%20Patrol%20and%20Device%20Simulator/overview) | Mission state machine, frame upload/backfill, 18-device 2s telemetry simulator | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Drone%20Patrol%20and%20Device%20Simulator) |
| [Backend Runtime (Express + Aedes MQTT + SQLite)](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Backend%20Runtime%20%28Express%20%2B%20Aedes%20MQTT%20%2B%20SQLite%29/overview) | App bootstrap, embedded MQTT, WS broadcast, in-memory device catalog, SQLite storage | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Backend%20Runtime%20%28Express%20%2B%20Aedes%20MQTT%20%2B%20SQLite%29) |
| [Server Acceptance & E2E Tests](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Server%20Acceptance%20and%20E2E%20Tests/overview) | HTTP/MQTT/WS integration checks for alarms, drones, irrigation, work orders | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Server%20Acceptance%20and%20E2E%20Tests) |
| [Seed Data & Ops Scripts](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Seed%20Data%20and%20Ops%20Scripts/overview) | One-shot scripts: KB import, tree seeding, drone simulation, QA eval | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Backend%20Service%20%28server%29/Seed%20Data%20and%20Ops%20Scripts) |
| [Frame Detection Inference Service](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Frame%20Detection%20Inference%20Service/overview) | FastAPI + YOLOv8 single-frame detection with graceful fallback | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Frame%20Detection%20Inference%20Service) |
| [Desktop Deliverables & Phase Packages](https://xigma3628-creator.github.io/smart-garden-campus-workorder/en/#/Smart%20Garden%20Project%20Root/Desktop%20Deliverables%20and%20Phase%20Packages/overview) | Installer notes & SHA256, per-phase task/acceptance/prompt packages | [tree](https://github.com/xigma3628-creator/smart-garden-campus-workorder/tree/main/en/Smart%20Garden%20Project%20Root/Desktop%20Deliverables%20and%20Phase%20Packages) |

## 本地预览 / Local Preview

```bash
npx docsify-cli serve .
# 中文 http://localhost:3000/ · English http://localhost:3000/en/
```

## 仓库结构 / Repository Layout

```
index.html        中文站入口（docsify，basePath zh/）
en/index.html     English site entry
zh/               中文知识卡（50 篇）
en/               English knowledge cards (50 pages)
.nojekyll         GitHub Pages 关闭 Jekyll
```

## 部署 / Deployment

GitHub Pages 已启用：Source = `main` 分支 `/(root)`。向 `main` 推送后约 1 分钟自动重新构建发布；站点即上面两个地址。

---

源工程：Vue3 + Cesium 大屏 · Node.js Express + Aedes MQTT + SQLite 后端 · FastAPI + YOLOv8 推理 · Electron 桌面打包 · Capacitor 工人端 APK。
