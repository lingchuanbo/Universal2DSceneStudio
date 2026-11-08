# 05 AI Native：AI 真正驱动软件
## 总原则
AI 是自然语言操作层，必须理解当前项目、选中对象、节点图、参数、预览和错误状态。它不能直接写内部内存，只能通过版本化 Tool API 和 Transaction 操作。

## Agent 执行链
User Intent → Intent Understanding → Context Builder → Planner → Task Graph → Permission Check → Preview Transaction → Validation/Render Preview → User Approval → Apply Transaction → Result + Undo Handle。

## AI 功能
1. **自然语言建效果：** 检索节点组、组装节点、填参数、输出可编辑图。
2. **修改当前效果：** 根据当前选中对象定位最相关参数，尽量只改必要部分。
3. **解释节点图：** 说明数据流、关键参数、视觉结果来源和性能影响。
4. **自动排错：** 诊断类型错误、缺资源、空输出、模拟缓存和 GPU 问题。
5. **性能优化：** 分析节点耗时、缓存命中、粒子数和重复计算，提供 A/B 预览。
6. **参数智能调节：** 根据目标描述调整参数；不确定时生成 Variant。
7. **工作流自动化：** 批量建效果、改色、导出、检查缺失纹理。
8. **教学辅助：** 解释节点、推荐后续节点、生成可复用 Node Group。

## Agent Skills 与产品分期
基础 Graph Builder、当前效果修改、解释/诊断及受控 Preview Patch 是首个可用版本的核心能力，并与手动编辑共用 Project Graph、Transaction、Undo 和 Revision；交互与验收详见[文档 11](11_手动与AI双模编辑规格.md)。推荐采用一个 Agent Host + 多 Skill，不部署多个独立 Agent 服务。

高级 Optimization、Vision A/B 评估、批量自动化、多 Provider 智能路由和 Director/Worker Agents 属于扩展阶段。手动编辑不依赖 AI 服务，但这不代表 AI 编辑从首版范围中移除。

## Context Builder
按需提供最小充分上下文：项目摘要与 Revision、选中 Effect/Node/Parameter、Graph 摘要、Node Catalog、预览缩略图（用户授权）、错误和性能统计、当前目标。不要把整个工程无差别塞给模型。上下文标注来源、版本和可信等级。

## Graph Builder 流程
解析目标 → 识别当前 Effect/Node/Parameter 选择 → 构建最小充分上下文并绑定 Project Revision → 检索 Node Catalog/Node Group → 生成 Graph Plan/Patch → 校验 Socket/版本/环路/权限/预算 → 创建隔离 Preview → 编译渲染并展示差异摘要 → 必要时澄清或迭代 → 再检查 Revision → 用户确认 Apply → 以原子 Transaction 提交到同一 Project Graph → 返回 Undo Handle 并保存 AI Task Record。

Revision 过期时不直接重放旧 Patch；必须拒绝或在重新读取、重新校验和重新预览后请求确认。具体冲突规则见文档 11。

## 模型输出约束
默认只允许结构化计划、Tool 调用、Graph Patch、Parameter Patch、解释和诊断。禁止默认执行任意 Rust/C++/WGSL/Shell。自定义代码节点必须独立授权、沙箱运行并限制资源。
