# VFXForge Studio 项目文档

> 文档版本：1.1 · 状态：方案与执行基线 · 更新：2026-11-08

VFXForge Studio 是面向游戏 VFX 和 2D/2.5D 动效的程序化创作工具。本目录描述目标设计与建议执行方式；**除非另有实现记录，不代表相关功能已经开发或通过验收**。当前交付以产品与技术设计文档为主。

## 从这里开始

- 项目团队：先读[项目执行与技术美术规范](09_项目执行与技术美术规范.md)，确认首版目标、决策项、阶段产物和验收方式。
- 产品与设计：阅读[产品定位与工作流](01_产品定位与工作流.md)和[总体架构与项目模型](02_总体架构与项目模型.md)。
- 节点/运行时开发：阅读[节点系统与数据语义](03_节点系统与数据语义.md)、[编译渲染与运行时](04_编译渲染与运行时.md)及[Rust 工程与路线图](08_Rust工程与路线图.md)。
- AI 与工具开发：先阅读[手动与 AI 双模编辑规格](11_手动与AI双模编辑规格.md)，再阅读[AI 驱动与 Agent 架构](05_AI驱动与Agent架构.md)和[AI 工具协议与事务安全](06_AI工具协议与事务安全.md)。
- UI、插件与交付：阅读[Provider、UI、插件与导出](07_Provider_UI_插件导出.md)。
- 数据契约：查看 [`schemas/`](schemas/) 下的 Graph、Node、Tool、Transaction、[Graph Patch](schemas/graph-patch.schema.json)、[Project Manifest](schemas/project-manifest.schema.json)、[Export Profile](schemas/export-profile.schema.json)、[Atlas Manifest](schemas/atlas-manifest.schema.json) 和 [AI Task Record](schemas/ai-task-record.schema.json)。
- 效果验收：按[游戏特效验收记录模板](templates/effect-acceptance.md)为代表效果留存目标端验证数据。

## 文档地图

| 文档 | 内容 | 主要读者 |
|---|---|---|
| [01 产品定位与工作流](01_产品定位与工作流.md) | 目标用户、使用流程、产品边界 | 产品、设计、全体 |
| [02 总体架构与项目模型](02_总体架构与项目模型.md) | 模块分层、三种图、项目持久化 | 架构、核心开发 |
| [03 节点系统与数据语义](03_节点系统与数据语义.md) | Socket、Geometry、Field、Simulation | 图引擎、技术美术 |
| [04 编译渲染与运行时](04_编译渲染与运行时.md) | 编译执行、CPU/GPU、预览质量 | 渲染、图引擎 |
| [05 AI 驱动与 Agent 架构](05_AI驱动与Agent架构.md) | 上下文、计划、生成与诊断 | AI、产品 |
| [06 AI 工具协议与事务安全](06_AI工具协议与事务安全.md) | 工具契约、权限、事务与错误码 | AI、平台、安全 |
| [07 Provider、UI、插件与导出](07_Provider_UI_插件导出.md) | Provider、编辑器工作区、插件、资产和导出 | UI、平台、技术美术 |
| [08 Rust 工程与路线图](08_Rust工程与路线图.md) | 技术栈、阶段依赖、原型和验收 | 技术负责人、开发 |
| [09 项目执行与技术美术规范](09_项目执行与技术美术规范.md) | MVP、决策项、内容规范、性能预算、交付门禁 | 全体，执行基线 |
| [10 UI 技术选型与 BUI 原型](10_UI技术选型与BUI原型.md) | BUI Rust 候选、许可证风险、节点画布与 wgpu 视口原型门禁 | UI、架构、产品 |
| [11 手动与 AI 双模编辑规格](11_手动与AI双模编辑规格.md) | 共用工程模型、AI 预览/事务、冲突处理、权限与验收 | 产品、UI、AI、核心开发 |
| [12 实现级技术设计](12_实现级技术设计.md) | Rust 模块边界、项目包、事务接口、BYOK、P0 Runtime/导出契约 | 架构、核心开发 |
| [13 MVP 开发任务](13_MVP开发任务.md) | 有依赖和完成定义的阶段任务与发布阻断条件 | 项目负责人、开发、QA/TA |
| [ADR-0001 产品基线](decisions/ADR-0001-product-baseline.md) | 平台、导出、发行和 BYOK 决策 | 产品、项目负责人 |
| [ADR-0002 BUI UI](decisions/ADR-0002-ui-bui.md) | BUI 原型与最终接受门禁 | UI、架构、项目负责人 |

## 当前基线与完成度

当前交付包含产品/架构规格、路线图、TA 验收模板、BUI 评估、实现级设计草案和 MVP 任务表，不是已实现软件。已确认首发基线：Windows 10/11 x64、引擎无关 PNG 序列/Sprite Sheet + Atlas JSON、开源且接受 GPL 兼容义务、BYOK 云端 AI。仍需完成最低硬件实测、Provider 服务商/model 选择、BUI 固定依赖法律/许可证签收与 Windows 节点画布/视口原型门禁。详细任务见[实现级技术设计](12_实现级技术设计.md)、[MVP 开发任务](13_MVP开发任务.md)和 decisions/ 下 ADR。节点目录、runtime 语义、Schema 正反例、Atlas consumer parser、实施/QA 结果仍须在编码和测试后闭合。

产品硬要求：P0 同时交付可手动自定义编辑和可实际调用的 AI 驱动编辑；两者共用一份工程和同一 Undo/Revision/校验链路。AI 断线不阻断手动流程。不得把基础 AI 推迟为“以后有空再做”。

## 统一约定

- 文档里的“必须”表示契约或验收要求；“建议”表示方案选项；未确认事项明确标记为“待决策”。
- 新增功能先更新适用的需求、数据契约和验收标准，再进入实现；实施后记录状态、测试方式和已知限制。
- 影响序列化、节点语义、插件或工具协议的改动必须标注版本与迁移策略。
- 面向游戏生产的效果验收必须包含可复现输入、目标平台/画布、检查步骤和结果；仅有示意图不算通过。
- 架构内容以决策记录为准；每项决策记录状态、负责人、日期、备选项和影响范围。未有记录时按待决策处理，不擅自视为已定案。

## Schema

`schemas/` 是数据契约草案，包含 Project Manifest、Graph、Node、Tool、Transaction、Graph Patch、Export Profile、Atlas Manifest 和 AI Task Record。开始实现前需为每种契约补充示例和正反向测试，并用 JSON Schema Draft 2020-12 validator 验证；Schema 仅能检查结构，节点引用完整性、Socket 兼容性、图环路、Revision 冲突和权限仍由应用逻辑检查。
