# 07 Provider、UI、插件与导出
## AI Provider
P0 采用用户自带云端 API Key（BYOK），首个 Adapter 以 OpenAI-compatible Chat Completions + JSON mode/schema 子集验证结构化 Plan/Patch；具体服务商和 model 待确认并实测。Key 用 Windows Credential Manager/OS 等效 Secret Store 保存，不写入 Project、Activity、日志或导出。P0 不承诺 Vision、图像/视频生成、Embedding、本地推理、多 Provider 自动路由；后续可在 Provider Adapter 稳定后扩展。

P0 不启用 Provider Router：用户显式选择单一 Provider/model 和连接配置，未经同意不得切换或把 Context 发往其他服务。视觉反馈以运行时渲染预览和用户检查为准；Vision 模型评价属于后续阶段。模型输出只是结构化编辑提议，实际节点语义和编译诊断由软件校验。

每次 AI Task 仅保存隐私受限的意图摘要、context/patch digest、Provider/Model、共享数据类别、revision、transaction/undo 关联、Preview/Apply/Reject/失败状态、耗时和错误码。原始 Prompt 默认不持久化；API Key、HTTP headers/body 永不写入工程/Activity。记录格式见 [`ai-task-record.schema.json`](schemas/ai-task-record.schema.json)。

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

P0 固定为引擎无关的透明 PNG 序列与 Sprite Sheet + Atlas JSON，数据结构见 `schemas/export-profile.schema.json` 和 `schemas/atlas-manifest.schema.json`，实现约定见文档 12。Atlas 记录有理数 FPS、帧区间、canvas、pivot/bounds、颜色/Alpha、纹理 hash、packing 与每帧 rect/trim metadata。Unity/Unreal/Godot 原生 Runtime 插件及 Effect Runtime Data JSON 延后，不将序列帧导出描述为引擎内实时粒子效果。
