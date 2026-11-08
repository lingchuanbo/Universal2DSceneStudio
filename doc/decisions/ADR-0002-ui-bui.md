# ADR-0002：桌面 UI 与 BUI

- 状态：Proposed（首选候选；不得在原型门禁前视为最终 Accepted）
- 日期：2026-11-08
- 决策人：产品方向已确认，技术/法律验收待签收
- 依据：[BUI 原型规格](../10_UI技术选型与BUI原型.md)、[ADR-0001](ADR-0001-product-baseline.md)

## 背景

产品需要 Rust 桌面 UI、节点编辑器、BUI 风格参数控件，以及和独立 wgpu 预览视口同窗运行。产品选择开源并接受 GPL 兼容发行，但这只使 GPL 义务成为可接受候选，不自动证明 BUI 的完整依赖栈适配发行策略。

## 决策

以固定 commit 的 BUI Rust binding 做 Windows x64 主窗口/参数控件原型。BUI 只负责 UI 展示、布局和输入；不持有 Project 真相、不编译图、不拥有效果 Runtime Device。Project Core、Graph Engine、AI Service 与 Export API 不依赖 BUI 类型。

应用视口由 VFXForge 渲染层拥有。优先验证同窗 Surface/共享资源；如果 BUI host 与 wgpu 互操作成本过高，可采用 Windows 子窗口或受控帧复制，并量化延迟。不得因“同为 GPU”而默认共享 Device 或 swap chain。

## 必须通过的门禁

- Cargo 依赖与 BUI source 固定到 release/commit 并入 SBOM；扫描所有许可证和非代码资产。
- Windows 输入测试：中文 IME 组合输入、Ctrl/Shift 快捷键、文本剪贴板、键盘焦点、DPI 125/150/200%、多显示器移动、弹窗/文件对话框。
- 节点画布原型验证 1k 节点平移/缩放/拖拽/连线/框选，实测目标设备延迟和帧时间。
- wgpu 视口持续播放固定 Seed VFX、resize/restore/device lost、透明画面、无 GPU 降级、AI 面板开启时不卡帧。
- 至少一名 TA 完成核心任务走查，记录无法使用的键盘/输入法行为和替代方案。
- 许可证律师/项目负责人审核固定版本与整个应用的 GPL-compatible 分发方式并签字。

任何关键门禁失败时回到 UI 替代方案评审；核心模块仍保持与 UI toolkit 解耦。
