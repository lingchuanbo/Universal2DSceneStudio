# 13 MVP 开发任务与依赖

> 状态：可排期草案。未填写负责人/工期前不可视为已承诺排期。任务按依赖编号；产品 P0 同时包含 Manual 与 AI 编辑。开发前决策见 [ADR-0001](decisions/ADR-0001-product-baseline.md)、[ADR-0002](decisions/ADR-0002-ui-bui.md) 与 [实现级技术设计](12_实现级技术设计.md)。

## 阶段 0：工程与原型门禁

| ID | 任务 | 依赖 | 完成定义 |
|---|---|---|---|
| T000 | 建立 Rust workspace、Windows CI、格式/lint/test 门禁 | 无 | MSVC 构建；CI 固定 Rust/CMake/toolchain；保存 artifact/log；干净 Runner 可重复 |
| T001 | 建立许可证/SBOM 策略与开源发布模板 | 无 | SPDX 扫描、依赖来源、THIRD_PARTY_NOTICES、生成物/license 记录纳入 CI；检查通过 ADR-0001 兼容模式 |
| T002 | BUI Windows x64 构建与 IME/keyboard/input Spike | T000,T001 | BUI 固定 commit；Windows 启动、DPI、多窗口/对话框、中文 IME/剪贴板/焦点走查记录；见 ADR-0002 |
| T003 | BUI 节点 Canvas 与交互 Spike | T002 | pan/zoom/selection/node drag/socket connect/box select 示范；1k 节点测试记录；无现成 Node Editor 时自研成本评估 |
| T004 | BUI + wgpu 同窗预览 Spike | T002 | 固定 seed 动效、resize/restore/HiDPI/device loss、帧延迟和复制路径实测；UI 不阻塞；通过 ADR-0002 决定继续/替换 |
| T005 | 固定 Windows 首发目标与最低设备基线 | T000,T002,T004 | Windows 10/11 build/DPI/CPU/GPU/RAM 清单、CI 实机、性能场景和性能基线批准；记录 CPU Preview 为 fallback 基线及 GPU 是否纳入 P0 的决议 |
| T006 | 选择 BYOK Provider/model/API profile | T001 | 评估候选 vendor/model 的 OpenAI-compatible API/JSON schema 能力、区域/隐私、价格/限额/terms；记录正式选择与取消条件。真实连接测试在 T031 执行 |

**Gate G0：** T001/T002/T004 的 license、输入法及视口门禁必须签收。失败时停止向 BUI 写业务 UI，选择替代路线，Core 不受阻继续。

## 阶段 1：数据与核心

| ID | 任务 | 依赖 | 完成定义 |
|---|---|---|---|
| T010 | 定义 vfx_types 与稳定 ID/Revision/单位 | T000 | ID 生成、比较、serde、finite numeric/units、错误码有测试；不依赖 BUI/provider |
| T011 | 落地 Graph/Node Descriptor v1 和最小节点目录 | T010 | Graph schema、P0 node descriptor 与兼容规则冻结；节点版本缺失可保留诊断 |
| T012 | 图验证/诊断与固定拓扑顺序 | T011 | node/socket 存在、类型、普通 DAG cycle、输出可达性、稳定 diagnostics 覆盖正反例 |
| T013 | Project model/manifest/asset index | T010,T011 | Project/Effect/Graph/Asset 稳定 IDs；manifest schema；引用完整性校验 |
| T014 | `.vfxproj` ZIP64 open/save/migration/recovery | T013 | 原子写、旧版本迁移、未来版本只读、autosave recover、path traversal/Zip bomb/hash 测试 |
| T015 | Project Query API/Selection/Editor read models | T013 | 供 Manual UI/AI context 共用的只读摘要；不暴露内部可变引用 |
| T016 | P0 CPU Runtime 节点实现与 golden test harness | T011,T012 | Image/Particle/Trail/Color/Size/Sprite/Output 基础链路；固定 Seed 重复运行稳定；不可用诊断完整 |
| T017 | Frame output/Preview API | T016,T005 | preview frame image + diagnostics + timings；参数变更请求可取消/合并；无 Project mutation |

## 阶段 2：同一事务和人工编辑

| ID | 任务 | 依赖 | 完成定义 |
|---|---|---|---|
| T020 | Command model/Manual edit command registry | T013,T015 | CreateEffect/AddNode/Connect/SetParameter/SetKeyframe 等 action 有验证器和序列化测试 |
| T021 | Canonical read/write set 推导 | T020 | Host 从 read snapshot/commands 推导集合；声明不匹配拒绝；对象/属性粒度明确 |
| T022 | Transaction prepare/preview/atomic commit | T017,T020,T021 | revision check、patch digest、preview TTL、approval token、原子提交、revision+1；故障注入无半提交 |
| T023 | Undo/Redo 与 History | T022 | command inversion 或补偿事务；期望 revision 校验；后续编辑不会被旧 Undo 覆盖 |
| T024 | 手动图/Inspector/timeline 参数交互 | T002/T003,T015,T020,T023 | 实际 UI command 流程完成增点、连线、参数/关键帧修改、预览、Undo/Redo |
| T025 | 保存状态、dirty indicator、关闭/恢复流程 | T014,T023,T024 | Save/Save As、dirty、外部文件错误、未保存确认和 crash recovery smoke test |

## 阶段 3：AI BYOK 双模编辑与导出

| ID | 任务 | 依赖 | 完成定义 |
|---|---|---|---|
| T030 | Credential Store 与 Provider Settings | T000,T005 | OS secret store；Key 新增/替换/删除；日志、工程、导出和错误文本扫描不到 Key |
| T031 | Provider trait + mock + HTTP Adapter | T030,T006 | 选定 Provider/API 通过结构化输出、TLS、timeout、cancel、size limit、429/5xx/坏 JSON 测试；对固定任务记录有效率/失败率与费用 |
| T032 | Context projection/consent UI | T015,T024,T030 | 预览每次发送 Provider/model/数据类别；用户可拒绝预览图/资产；Context 有 revision/source digest |
| T033 | Graph Patch schema/parser/semantic validation | T011,T021 | P0 schema 正/反向测试；CreateEffect/AddNode/Connect/SetParameter/SetKeyframe 操作闭集；limits enforced |
| T034 | AI Plan -> preview -> approval -> Apply/Undo workflow | T022,T023,T031,T032,T033 | AI 可创建 effect/graph 并修改现有效果；所有写入必须确认；Reject/Cancel/timeout/offline/stale revision 无写入；Apply 后同图可手工继续编辑 |
| T035 | AI Task Record/Activity UI | T031,T034 | 可追 provider/model、数据类别、revision、patch digest、状态和错误；无 raw key/headers/body；支持显示撤销态 |
| T036 | Export profile / PNG sequence | T016,T017 | fps/frame range、透明通道、颜色空间、hash、staging+atomic replace、碰撞默认 fail 通过测试 |
| T037 | Atlas packer + Atlas JSON schema/reference parser | T036 | bounds/padding/extrude/trim/metadata/hash；图集无越界/帧缺失/rect 碰撞；独立消费端 reference parser round-trip；P0 只确认引擎无关协议，不交付 Unity/Unreal 插件 |

## 阶段 4：端到端验收

| ID | 任务 | 依赖 | 完成定义 |
|---|---|---|---|
| T040 | 双模同工程端到端 TA 场景 | T025,T034,T036,T037 | AI 创建 -> 手工改节点 -> AI 改参数 -> 手工曲线微调 -> 保存重开 -> 导出；反向手动建图 -> AI 修改路径也通过 |
| T041 | Revision/事务/安全对抗测试 | T022,T034 | 预览中手工并发修改、取消 race、过期 approval、读写集合不匹配、资源超限均无覆盖/半提交 |
| T042 | VFX 内容验收包 | T016,T036,T037 | 至少 3 个代表效果（如 projectile/impact、trail、looping ambient）；源工程、纹理、固定 seed、参考帧、导出 PNG/Atlas、材质/Alpha 约定和 TA acceptance record 齐全 |
| T043 | Windows clean install/release/license audit | T001,T005,T040,T042 | 干净 VM 安装、运行、卸载；SBOM/source/notices/license/版本信息发布物完整 |

## MVP 发布阻断条件

- 手动完整路径成立但 AI 只展示架构/聊天，不算完成。
- AI 写入与手动编辑若采用不同 Project model、Undo 或序列化路径，不算完成。
- AI 写入无需用户确认、或 Provider 失败可污染工程，不算完成。
- Atlas JSON 没有独立 schema/reference reader、Frame 对不上导出内容，不算完成。
- P0 BUI/依赖许可证审查未签收，不允许分发构建。
- 任一首发决策变化必须更新 ADR、接口/schema 和受影响任务，不靠口头变更。

## 排期模板

每个 task 进入迭代前填写负责人、预估/实际、起止、风险、PR、测试 artifact 和验收人。表中依赖代表 blocking dependency；不要以并行开工替代阶段门禁。
