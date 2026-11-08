# 08 Rust 工程结构、路线图与验收
## 技术栈
Rust：Project Core、Graph Engine、数据模型、Agent Tool Host、调度。  
wgpu/WGSL：GPU Compute 与 Rendering。  
C++：仅在必要的媒体编解码或成熟 SDK 边界使用。  
Python/TypeScript：开发工具、自动化、外部 Agent。  
SQLite：项目索引、资产元数据和历史记录；大文件独立存储。  
桌面 UI 首选原型为 BUI 的 Rust `blenderui` binding（`Proposed`，尚未通过许可证及集成门禁）；BUI 资料与原型门禁见[文档 10](10_UI技术选型与BUI原型.md)。核心数据模型不得绑定 UI 类型。

## Workspace
```text
vfxforge/
├─ apps/{vfxforge-studio,render-worker,ai-worker}
├─ crates/
│  ├─ vfx_types
│  ├─ vfx_project
│  ├─ vfx_transaction
│  ├─ vfx_graph
│  ├─ vfx_graph_ir
│  ├─ vfx_node_registry
│  ├─ vfx_geometry
│  ├─ vfx_field
│  ├─ vfx_attribute
│  ├─ vfx_particle
│  ├─ vfx_simulation
│  ├─ vfx_animation
│  ├─ vfx_compiler
│  ├─ vfx_runtime_cpu
│  ├─ vfx_runtime_gpu
│  ├─ vfx_render
│  ├─ vfx_ai_agent
│  ├─ vfx_ai_provider
│  ├─ vfx_tool_api
│  ├─ vfx_plugin_host
│  ├─ vfx_asset
│  ├─ vfx_cache
│  └─ vfx_export
└─ schemas/
```

## 开发阶段与依赖
阶段执行以[项目执行与技术美术规范](09_项目执行与技术美术规范.md)的门禁为准。本路线图强调工程依赖；以下项目是建议顺序，不代表已经实现。

**阶段 A：产品与平台验证。** 确认首发平台、BUI Rust 桌面/节点画布/视口互操作原型、GPL 分发评审、游戏运行时/导出目标、设备预算和代表性 VFX 验收场景。UI 门禁按文档 10 执行；先解决会改变数据模型和渲染接口的决策。

**阶段 B：手动与 AI 垂直切片。** 建立 Project 文件往返、最小强类型 Graph、手动参数编辑、Sprite 或基础粒子预览和一种游戏目标导出；同时验证一个最小自然语言修改 Patch：读取当前 Effect/Revision、改一个受限参数、展示预览和变更摘要、明确确认后应用并 Undo；同时验证首次 AI CreateEffect 与首张 Graph 原子创建。先证明两条路径写入同一工程且可互相接续；暂缓通用插件系统和多后端抽象。

**阶段 C：可用核心（P0）。** 补齐 Undo/Redo、图诊断、基础粒子/拖尾、关键帧、纹理资源管理、基础 AI 创建/修改/解释、预览事务、Revision 冲突与失败回滚、常用组合节点、示例工程及自动化回归。CPU Reference 作为语义基准；是否 P0 要求 GPU 运行时，以阶段 A 的平台决策为准。

**阶段 D：模拟与性能（P1）。** 实现固定时间步 Simulation Zone、状态快照、Bake/Reset、时间跳转恢复；测量代表场景，再按瓶颈增加 Field/Attribute、缓存、GPU 优化与质量降级。

**阶段 E：高级 AI 与扩展（P1）。** 扩展视觉对比、优化 Agent、多 Provider 路由、批量工作流、插件 SDK 与更多导出目标。AI Provider 不可用时手动编辑不受影响。双模产品需求和冲突契约见文档 11。

每阶段出口条件、资产验收及测试矩阵详见文档 09；本路线图不应替代阶段签收。

## 技术原型顺序
1. **端到端效果切片：** 明确画布、贴图/色彩/Alpha、参数编辑、播放预览及目标运行时导出。让首发用户任务驱动数据模型，而不是从完整 Geometry Node 目录起步。
2. **强类型图与 CPU 语义：** 建立 Node Descriptor、Graph 校验/诊断、序列化往返和一组可测核心节点。
3. **粒子/拖尾与关键帧：** 用固定 Seed 的代表性游戏效果检验生命周期、颜色/尺寸曲线、排序和视觉导出。
4. **性能与模拟：** 先建立目标设备基线，再实现缓存、固定时间步和 GPU 优化；报告质量/帧时间权衡。
5. **AI 修改切片（P0）：** 选择当前效果，运行受限参数修改的 Read → Patch → Validate → Preview → Apply/Reject → Undo，注入 Revision 冲突和 Provider 失败验证双模一致性。
6. **AI 创建与扩展（P0/P1）：** 核心节点稳定后生成可手工接续编辑的效果图；再扩展视觉评估、多 Provider 和优化 Agent。

## P0 出口验收摘要
- 效果师可手动创建、保存、关闭重开、调整并导出一个代表效果；目标端的时序、颜色空间、Alpha、Pivot/Bounds 和帧率通过检查。
- Graph 支持强类型连接、缺失节点/非法连接/普通 DAG 环路诊断，以及稳定序列化往返。
- 基础 AI 可从描述创建一个可编辑效果，并修改当前效果的参数/曲线；预览/确认/拒绝/撤销、过期 Revision、取消/失败回滚有测试。
- AI Apply 后仍可手动修改同一 Graph 并保存重开；无网络/无授权 Provider 时完整手动流程可用。
- 关键资产和示例可定位、可复现；效果至少包含固定 Seed 的参考时间点和导出产物。
- 测试覆盖项目往返、核心图编译/执行、AI Patch 与导出；目标硬件的性能预算须先由产品确认并记录实测。
- Simulation、GPU/CPU 数值一致性、MCP、高级多 Provider/优化 Agent、插件 SDK 等仅在纳入对应阶段后验收，不视作基础 P0 隐式承诺。

## 风险
首版范围过大：2D/2.5D 优先。节点类型失控：强类型与版本化 Descriptor。Field 复杂度：先做核心 Domain 和 CPU Reference。GPU 不一致：建立容差测试。AI 不稳定：检索节点组、Schema 校验和预览事务。工程损坏：Revision、原子 Transaction、Undo。Provider 锁定：Adapter 和用户路由设置。

## 最终原则
AI 负责理解、规划和加速；Graph Engine 负责确定性执行；用户始终保留专业控制权。Project Graph 是唯一事实来源，所有写操作必须进入 Transaction，所有核心节点必须可测试，所有 AI 行为必须可追溯。
