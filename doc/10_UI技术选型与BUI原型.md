# 10 UI 技术选型与 BUI 原型

> 决策状态：`Proposed`（BUI Rust binding 为 Windows x64 首选候选；用户已确认开源并接受 GPL-compatible 发行，但固定依赖法律审查、IME、节点画布/视口工程原型仍未签收）。技术决策记录见 [ADR-0002](decisions/ADR-0002-ui-bui.md)。
>
> 上游信息检查日期：2026-11-08；本次核对的 `main` commit：`e8126cab68f1b542676e83909671bc803dedb36c`。合入项目时需固定到经审核的 release/commit，不跟踪浮动 `main`。

## 1. 方向

将 [BUI](https://github.com/ppmpreetham/BUI) 作为 VFXForge Studio 桌面 UI 的**首选原型和暂定技术方向**，使用其 Rust `blenderui` 绑定承载应用 Shell、菜单、面板、参数控件、时间线/列表和主题。产品工程以 Rust 为主时，避免先引入浏览器/WebView 或再搭一套独立组件框架。

此决定只覆盖应用 UI 框架的评估方向，不表示已批准分发 BUI，也不表示节点编辑器和实时视口已解决。只有在许可证、Rust 控件覆盖、视口集成和目标平台原型门禁通过后，才能把状态从 `Proposed` 改为 `Accepted`。

## 2. 已核对的上游信息

- 上游定位为从 Blender UI 交互/控件中拆出的独立库，核心是 C kit，并提供 Rust `blenderui` 绑定；Rust 使用 gpui 风格 API、HTML/CSS 与 Tailwind/`rsx!`，布局使用 Taffy。
- Rust crate 需 Rust 1.88+、CMake 和 C 编译器；上游列出 Win32、Linux X11 和实验中的 macOS Cocoa。**本产品 P0 仅将 Windows 10/11 x64 Win32 host 纳入支持与验收范围**；其余平台不构成首版承诺。
- 上游将软件光栅作为默认 UI 渲染路径，GPU UI 渲染器当前列出 Windows Direct3D 11 和 Linux OpenGL 3.3。该 UI 渲染器不是 wgpu，也不能假设与项目 VFX 视口共用 GPU Device/纹理。
- 上游 README 说明 45 个 Blender widgets/editors 中仅有 10 个可从 Rust/JavaScript 使用；其节点编辑器示例展示在 Python 路径。不能推断 Rust crate 已提供可直接投入生产的 Node Editor。
- BUI README 明列无障碍支持和输入法支持尚未完成。中文输入、屏幕阅读器、键盘导航和高 DPI 操作必须进行实测。
- GitHub 仓库元数据标识 GPL-3.0；Rust workspace 标识 GPL-3.0-or-later。以固定版本源代码中的 LICENSE、workspace/crate metadata 和依赖清单为准，入库时执行许可证扫描。
- 上游性能表为特定机器/场景数据，README 明确这些测量不能视为所有设备的承诺；项目需自行以目标硬件测量。

## 3. 架构边界

```text
BUI Application Shell
  ├─ 菜单 / 工具栏 / 项目资源与节点目录
  ├─ 参数 Inspector / 时间线 / 诊断 / AI Workspace
  └─ 视口宿主控件（待原型确认画面合成路径）

VFXForge Rust Core（不依赖 BUI）
  ├─ Project / Assets / Transactions / Undo
  ├─ Graph / Compiler / CPU & GPU Runtime
  └─ Editor Commands + Read Models

Viewport Renderer（独立所有权）
  └─ wgpu Device / Surface / Simulation Preview / Frame Scheduling
```

核心数据模型、事务、编译器、运行时和渲染后端不得依赖 BUI 类型。UI 只能经明确的 Command/Query 接口访问核心；这样可在许可证、UI API 或平台条件改变时替换壳层，而不迁移项目文件和图语义。

不要让 BUI 与 wgpu 分别创建窗口后假定能互相嵌入，也不要把 BUI 内部 C kit 对象写入 Project。优先验证单窗口原生布局中的视口 Surface；如果直接合成不稳定，评估受控的原生子窗口/纹理复制方案，并记录帧延迟、资源所有权、HiDPI 和平台差异。

## 4. 阶段 A 原型计划

### 原型 A：Rust 桌面基础壳

实现菜单、可停靠/可调整尺寸的主分栏、资源树、属性面板、可折叠区段、快捷键和主题切换；验证 BUI Rust binding 的构建、窗口关闭/恢复、DPI、中文输入法、剪贴板、键盘焦点和可访问性替代方案。Docking 行为若无现成稳定 API，应列为自研范围，不预设 BUI 已提供。

### 原型 B：节点画布

用 BUI Rust 当前稳定 API 做可交互节点画布：缩放/平移、节点选择与拖动、Socket 连线、框选、快捷菜单、撤销、图层排序、网格/曲线绘制和 1k 节点压力场景。先确认 Rust binding 是否有成熟的 Node Editor 组件；若无，则评估其 Canvas/Painter 和事件能力，记录自研输入/虚拟化成本。

### 原型 C：实时视口互操作

验证渲染 Surface 能否嵌入 BUI 主视口。P0 参考路径为 CPU Runtime 产帧供 UI 显示；若评估 wgpu，则记录其作为候选后端，不把 GPU 标为首版硬依赖。测试固定 Seed 动画、参数响应、Alpha/色彩空间、缩放/HiDPI、窗口 resize/最小化/恢复、设备丢失与无 GPU 降级。只对 Windows 10/11 x64 测试；记录资源复制/共享方式与 P50/P95 输入到画面延迟。

### 原型 D：目标用户走查

由至少一名游戏 VFX 使用者完成“打开示例、改颜色/持续时间、播放预览、撤销、保存重开、导出”的任务。记录完成时间、误操作、无法发现的控件、快捷键冲突及中文输入问题。

## 5. 原型出口门禁

- 在选定首发操作系统的干净环境可重复构建和打包；记录 Rust/CMake/C 编译器版本、构建耗时、产物大小和依赖许可证。
- 上述关键桌面输入、焦点和文件对话框行为通过测试；所有 P0 UI 工作流均可仅用键盘或提供可用替代方式。
- 节点画布核心操作达到效果师走查可接受标准，若依赖自研 Canvas，完成独立风险评估。
- wgpu 视口在目标分辨率/帧率下稳定显示，达到阶段 A 确认的延迟和性能预算；BUI UI 渲染器不能无依据替代 VFX runtime renderer。
- 许可证及开源分发方式得到项目负责人/法务书面确认，覆盖 BUI 固定版本与全部依赖、资源。若 GPL 兼容性条件不能满足，暂停产品集成并评估替代 UI；不得以静态/动态链接形式的未经审查假设绕过。
- 原型截图、录屏、测试记录、构建配置、依赖锁文件和 ADR 一并存档后，才将 UI 决策标记为 `Accepted`。

## 6. 许可证与第三方依赖门禁

GPL-3.0-or-later 是强 Copyleft 许可证。若把 BUI/Rust crate 与应用组合并向外部分发，可能对整个组合程序的许可证和对应源代码提供义务产生影响；实际边界需由熟悉软件许可的法律顾问根据链接方式、交付形式、修改情况和各依赖许可判断。本说明不是法律意见。

本项目当前发行方向是开源且接受 GPL 兼容义务，但仍须对固定依赖栈完成正式许可证审查。在决定每个版本分发前，应：

1. 核对固定版本所有复制源码的 SPDX/版权头、LICENSE、`THIRD_PARTY_NOTICES.md`、Cargo metadata 和传递依赖。
2. 生成 SBOM 与许可证清单，扫描生成资源、字体和图片的授权。
3. 明确产品是开源还是闭源、是否单一可执行文件、是否提供客户内部分发/插件 SDK，并取得书面审查结论。
4. 若不能接受 GPL 义务，取得权利人明确的商业许可，或换用许可证兼容的 UI；在此之前仅允许内部技术评估，不并入可分发产品构建。

## 7. 上游资料

- [BUI GitHub 仓库](https://github.com/ppmpreetham/BUI)
- [Rust binding README](https://github.com/ppmpreetham/BUI/tree/main/bindings/rust/blenderui)
- [Rust 使用指南](https://github.com/ppmpreetham/BUI/blob/main/docs/rust.md)
- [BUI 架构说明](https://github.com/ppmpreetham/BUI/blob/main/ARCHITECTURE.md)
- [BUI 许可证文件](https://github.com/ppmpreetham/BUI/blob/main/LICENSE)
- [BUI Rust workspace manifest](https://github.com/ppmpreetham/BUI/blob/main/bindings/rust/Cargo.toml)
