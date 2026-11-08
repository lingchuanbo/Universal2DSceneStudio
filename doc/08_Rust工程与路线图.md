# 08 Rust 工程结构、路线图与验收
## 技术栈
Rust：Project Core、Graph Engine、事务、CPU Runtime、AI Tool/Provider Host、导出管线。  
BUI Rust：首选桌面 UI 原型；正式接入受 [ADR-0002](decisions/ADR-0002-ui-bui.md) 门禁约束。  
wgpu/WGSL：实时预览渲染 spike 后决定 P0 使用范围，不默认要求 GPU Runtime。  
C/C++：P0 仅作为 BUI C kit 或必要外部 SDK 的构建依赖边界。  
Python/TypeScript：可用于内容工具/CI 辅助，不进入 P0 客户端必需 Runtime。  
SQLite：若资产索引性能/查询需要，作为可重建索引，不作为 Project 唯一数据来源。核心模型不绑定 UI/provider 类型。

## Workspace
```text
vfxforge/
├─ apps/vfxforge-studio
├─ crates/
│  ├─ vfx_types
│  ├─ vfx_project
│  ├─ vfx_graph
│  ├─ vfx_commands
│  ├─ vfx_transaction
│  ├─ vfx_runtime_cpu
│  ├─ vfx_render
│  ├─ vfx_preview
│  ├─ vfx_export
│  ├─ vfx_ai_provider
│  ├─ vfx_ai_agent
│  └─ vfx_ui_adapter
└─ doc/schemas/
```

## 开发阶段与依赖
阶段执行以[项目执行与技术美术规范](09_项目执行与技术美术规范.md)和[任务表](13_MVP开发任务.md)为准；以下只摘要 blocking dependency，不表示实现已经发生。

**阶段 A：工程与原型门禁。** Windows 10/11 x64、引擎无关 PNG/Atlas、GPL-compatible 开源发行和 BYOK 已由 ADR-0001 确认。阶段 A 实测最低硬件/性能、BUI 输入法/Node Canvas/wgpu 互操作、许可证依赖闭包，并选择具体 Provider/model。门禁任务按文档 13 执行；失败时替换 UI/provider Adapter，不重新打开已确认的产品方向。

**阶段 B：手动与 AI 垂直切片。** 建立 Project 文件往返、最小强类型 Graph、手动参数编辑、Sprite 或基础粒子预览和一种游戏目标导出；同时验证一个最小自然语言修改 Patch：读取当前 Effect/Revision、改一个受限参数、展示预览和变更摘要、明确确认后应用并 Undo；同时验证首次 AI CreateEffect 与首张 Graph 原子创建。先证明两条路径写入同一工程且可互相接续；暂缓通用插件系统和多后端抽象。

**阶段 C：可用核心（P0）。** 补齐 Undo/Redo、图诊断、基础粒子/拖尾、关键帧、纹理资源管理、AI 创建/修改/解释、受限 Preview/Approval/Apply、Revision 冲突与失败回滚、BYOK Adapter、PNG/Atlas 导出、示例工程和自动化回归。CPU Reference 作为语义基准；GPU Runtime 不自动属于 P0，仅在阶段 A 实测证明并经 ADR 追加后加入。

**阶段 D：离线模拟与性能扩展（P1）。** 实现固定时间步 Simulation Zone、状态快照、Bake/Reset、时间跳转恢复；测量代表场景，再按瓶颈增加 Field/Attribute、缓存、GPU 优化与质量降级。

**阶段 E：高级 AI 与扩展（P1）。** 扩展视觉对比、优化 Agent、多 Provider 路由、批量工作流、插件 SDK 与更多导出目标。AI Provider 不可用时手动编辑不受影响。双模产品需求和冲突契约见文档 11。

每阶段出口条件、资产验收及测试矩阵详见文档 09；本路线图不应替代阶段签收。

## 技术原型顺序
1. **端到端效果切片：** 按已确认的 Windows + 引擎无关序列帧/Atlas 目标，定义画布、贴图/色彩/Alpha、参数编辑、播放预览和可重读的导出 metadata。让首发用户任务驱动数据模型，而不是从完整 Geometry Node 目录起步。
2. **强类型图与 CPU 语义：** 建立 Node Descriptor、Graph 校验/诊断、序列化往返和一组可测核心节点。
3. **粒子/拖尾与关键帧：** 用固定 Seed 的代表性游戏效果检验生命周期、颜色/尺寸曲线、排序和视觉导出。
4. **性能与模拟：** 先建立目标设备基线，再实现缓存、固定时间步和 GPU 优化；报告质量/帧时间权衡。
5. **AI 创建/修改切片（P0）：** 选择当前效果或新建目标，执行 Read → Patch → Validate → Preview → Apply/Reject → Undo；包含原子 CreateEffect/Graph，注入 Revision 冲突和 Provider 失败，确认 AI 产物可被手工接续编辑。
6. **高级扩展（P1）：** 在 P0 稳定后评估视觉模型 A/B、多 Provider、本地模型、优化 Agent、Simulation 和引擎插件。

## P0 出口验收摘要
- 效果师可手动创建、保存、关闭重开、调整并导出一个代表效果；PNG/Atlas reference reader 解码后，帧序、颜色空间、Alpha、Pivot/Bounds 和有理数 FPS 检查通过。
- Graph 支持强类型连接、缺失节点/非法连接/普通 DAG 环路诊断，以及稳定序列化往返。
- 基础 AI 可从描述创建一个可编辑效果，并修改当前效果的参数/曲线；预览/确认/拒绝/撤销、过期 Revision、取消/失败回滚有测试。
- AI Apply 后仍可手动修改同一 Graph 并保存重开；无网络/无授权 Provider 时完整手动流程可用。
- 关键资产和示例可定位、可复现；效果至少包含固定 Seed 的参考时间点和导出产物。
- 测试覆盖项目往返、核心图编译/执行、AI Patch 与导出；目标硬件的性能预算须先由产品确认并记录实测。
- Simulation、MCP、多 Provider 自动路由/优化 Agent、引擎专属插件等只在纳入对应阶段后验收；P0 要求一个实际可用的 BYOK Provider Adapter，但不隐含指定某一供应商。

## 风险
首版范围过大：2D/2.5D 优先。节点类型失控：强类型与版本化 Descriptor。Field 复杂度：先做核心 Domain 和 CPU Reference。GPU 不一致：建立容差测试。AI 不稳定：检索节点组、Schema 校验和预览事务。工程损坏：Revision、原子 Transaction、Undo。Provider 锁定：Adapter 和用户路由设置。

## 最终原则
AI 负责理解、规划和加速；Graph Engine 负责确定性执行；用户始终保留专业控制权。Project Graph 是唯一事实来源，所有写操作必须进入 Transaction，所有核心节点必须可测试，所有 AI 行为必须可追溯。
