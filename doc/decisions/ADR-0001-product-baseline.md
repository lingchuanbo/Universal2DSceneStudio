# ADR-0001：首发产品基线

- 状态：Accepted（产品方向；UI 实现仍受 ADR-0002 原型门禁约束）
- 日期：2026-11-08
- 决策人：产品负责人（用户确认）
- 相关规格：[实现级技术设计](../12_实现级技术设计.md)、[MVP 开发任务](../13_MVP开发任务.md)

## 背景

需要固定首发平台、导出目标、发行模型和 AI 调用模式，以便拆解桌面 UI、序列化、导出与安全任务。此决策不代表软件已经实现，也不替代 BUI 依赖许可证清单或实现验收。

## 决策

| 项目 | 决策 | 约束/未决子项 |
|---|---|---|
| 首发平台 | Windows 10/11 x64 桌面应用 | 最低 CPU/GPU/RAM、最低 Windows build 与 DPI 范围由硬件验证任务确定；不以未经测量的性能数字作承诺 |
| 首发交付目标 | 引擎无关的 PNG 帧序列与 Sprite Sheet + Atlas JSON | 定义 FPS、时长、Pivot、Bounds、颜色空间、Alpha 模式、帧索引和资源哈希；Unity/Unreal/Godot 原生插件不纳入 P0 |
| 发行模型 | 开源发行，接受 GPL 兼容义务 | 逐项审核所有直接/传递依赖及资源许可证；选择 GPL-compatible 方式并发布 Notices/SBOM/对应源代码。具体法律义务由合格法律顾问复核 |
| 桌面 UI | BUI Rust binding 为首选实现原型 | BUI crate 与版本必须固定；完成 Windows 原型、Cocoa/X11 不在首发范围；许可证兼容性、输入法/键盘与视口集成通过门禁后才正式 Accepted |
| AI 调用 | 用户自带云端 API Key（BYOK） | Key 不进入工程、日志、崩溃报告或导出；用户明确选择 Provider/模型及数据发送范围；断网时手动编辑完整可用 |
| 首版 AI 写入 | 预览后由用户确认 | 首版不提供自动应用；写入走受限 Graph Patch、统一 Transaction、Revision 检查和 Undo |

## 后果

- 先实现 Windows 桌面包、帧序列/图集导出；目标游戏引擎专用 Runtime Adapter 延后。
- 工程核心保持 Rust 且与 UI/Provider/导出格式解耦；必须在初始阶段冻结最小序列化格式和迁移规则。
- 因接受 GPL-compatible 发行，BUI 可继续作为候选，但仍须锁定依赖版本并检查其依赖；GPL 兼容不等于免除署名、源码提供、许可证文本或修改标注义务。
- BYOK 降低服务端运营范围，但需要安全凭据管理、网络错误处理、Provider 兼容性测试和明确的数据披露。
- “支持任意兼容 API”不作为默认承诺；先指定一个可验证的 OpenAI-compatible Chat Completions + JSON 输出实现作为 Adapter 原型，具体服务/模型仍待产品确认。

## 验收与复审

- Windows 10/11 x64 干净环境可构建、安装、启动和卸载；最低硬件表来自实测。
- 一份工程能导出 PNG 序列和 Sprite Sheet/Atlas JSON；工具可读回并验证元数据、帧数、哈希和 alpha。
- 许可证扫描与法律评审覆盖 BUI 固定版本、Cargo/npm/build dependencies、字体/图标/测试素材；发布前将结果附于版本记录。
- Key 可新增/更新/删除，磁盘位置使用 Windows Credential Manager 或等效 OS 凭据存储；项目文件和诊断日志中无 Key。
- AI 写入由 Patch Schema 强制 `requires_approval: true`；拒绝/取消/Provider 失败均不修改工程。

## 变更条件

首发平台、引擎集成、发行许可模式或 AI 服务经营方式变化时创建新 ADR；不得静默修改本文决策。
