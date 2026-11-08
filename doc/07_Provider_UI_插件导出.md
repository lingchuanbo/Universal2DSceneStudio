# 07 Provider、UI、插件与导出
## AI Provider
统一 Provider Interface：Chat/Reasoning、Vision、Image Generation、可选 Video Generation、Embedding/Retrieval、Local Inference。云端适合复杂推理和高质量生成；本地适合隐私、低延迟和离线任务。支持 ONNX Runtime 或外部推理 Worker。

Router 根据任务类型、质量、延迟、成本、隐私和模型能力选模型。用户可锁定 Provider；不得后台切换到未授权服务。视觉反馈闭环：生成 Graph → Render Preview → Vision 检查颜色/密度/层次 → 参数迭代 → 用户确认。模型判断只是建议，真实参数和编译诊断以软件为准。

每次 AI Task 保存用户请求、Context Snapshot、Provider/Model、Prompt 版本、Tool Calls、Graph Patch、Preview、Apply/Reject、耗时和可用成本统计。

## UI 布局
```text
┌────────────────────────────────────────────────────────────┐
│ Project / Undo / Redo / Preview / Export / Manual | AI     │
├──────────────┬─────────────────────────────┬───────────────┤
│ Asset/Node   │           Viewport          │ Inspector     │
│ Catalog      │                             │ Parameters    │
├──────────────┴─────────────────────────────┴───────────────┤
│ Node Graph / Timeline / Curve Editor                      │
├────────────────────────────────────────────────────────────┤
│ AI Edit: Prompt / Plan / Diff / Preview / Apply / Undo     │
└────────────────────────────────────────────────────────────┘
```
建议渐进提供工作模式：Quick Create 聚焦模板和常用参数；Professional Edit 提供 Node Graph、Timeline、Keyframes、Inspector；Advanced 提供 Graph IR、Simulation Debug、GPU Profiling、Attribute Spreadsheet 和 Tool API。桌面 UI 首选以 BUI Rust binding 做原型，状态和适配边界见[文档 10](10_UI技术选型与BUI原型.md)。Manual 与 AI 是首个可用版本的并列编辑入口；AI 工作区可收起，但基础创建/修改不能依赖隐藏的未来扩展。

AI Edit 面板显示当前理解/假设、引用对象、修改计划、参数差异、预览对比、Provider/数据发送范围、Apply/Reject/Cancel/Undo 和工具诊断。用户可以在同一工程同时使用节点画布/Inspector 和 AI 面板；AI Preview 不应锁死手动编辑，但用户在预览期间手工改动后必须使旧预览失效并处理 Revision 冲突。

## 插件（后续阶段）
候选扩展包括 Effect、Node、Generator、Importer/Exporter、Tool、UI Extension 和 AI Skill Plugin。Native Rust/C++、WASM、Python/TypeScript 的安全与发布模型需先经过平台原型和威胁评审；不属于最小垂直切片。插件不得直接访问 Project 内存或 GPU Device，须通过稳定公开 SDK。Manifest 目标字段包括 plugin_id、version、api_version、capabilities、permissions、dependencies、platforms 和完整性信息。

## 资产与导出
目标资产类型包括 Image/Texture、Sprite Sheet、SVG、Curve、LUT、Font、Node Group、Preset、Simulation Cache；最小版本优先保证实际用到的纹理和效果资源可追踪、可迁移。

首个导出格式须由目标游戏运行时决策确定。候选项：透明图像序列、Sprite Sheet + Atlas JSON、Effect Runtime Data JSON，以及后续按需求支持的 PNG/EXR、MP4/WebM、GIF。所有运行时数据明确帧率、图集、Pivot、Bounds、事件、颜色/Alpha 约定、参数曲线和依赖版本。
