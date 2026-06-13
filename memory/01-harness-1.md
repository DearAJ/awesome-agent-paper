# Harness-1 — 把"工作记忆"从 policy 搬到环境侧（State-Externalizing Harness）

> **arXiv**：2606.02373（2026.06）｜**机构**：Chroma（chromadb）
> **HF 月榜**：2026-06 月榜 #18，52↑
> **关键词**：working memory · state externalization · candidate pool · evidence links · verification records · budget-aware context rendering · curated recall
> **GitHub**：[pat-jj/harness-1](https://github.com/pat-jj/harness-1)（548★）

---

## 1. 这篇论文为什么重要

**一句话**：搜索 agent 通常被训练成"在一段**不断增长的 transcript** 上的策略"——模型既要决定**怎么搜**，又要同时记住**看过什么、哪些证据有用、哪些约束还没满足、哪些断言已被核实**。Harness-1 指出这把**太多例行状态管理塞进了 policy**，于是把这部分**可恢复的"工作记忆（working memory）"外化到环境侧的 harness**，让强化学习只去优化真正的语义决策。

这是本 **memory 专题的核心论文**，因为它最干净地回答了用户关心的三个问题——**working memory 怎么表示、怎么更新、怎么使用**——并且把答案统一在"**state externalization（状态外化）**"这一条原则下：

- **问题的本质是"目标错配"**：当 candidate pool、evidence、verification 这些**可由环境可靠维护**的东西也压在 policy 身上时，RL 被迫**同时**优化两类完全不同的能力——① 语义搜索决策（高价值、难学），② 可恢复的记账（routine、本不该学）。后者吃掉样本与容量，还损害泛化。
- **解法是把②交给环境**：harness 负责维护结构化工作记忆并**按预算渲染**回上下文；policy 只保留①。
- **结果验证了这条原则**：8 个检索基准平均 curated recall **0.730**，比次优开源搜索子 agent **+11.4 分**，且**在 held-out transfer 基准上增益尤其强**——说明"在显式搜索状态上做 RL"能学到**跨域泛化**的检索行为。

它与 [[03-fs-researcher]]（把工作区外化到文件系统）、IterResearch（把记忆外化为演进报告，见 `deep-research/02`）、GAM（把完整历史外化到 page-store，见 `deep-research/11`）属于同一条主线：**working memory 不该被压死在 policy 的上下文里，而应外化成环境可维护、可按需渲染的结构化状态。** [[10-externalization-efficiency]] 则把这条主线上升为统一理论框架（externalization）。

---

## 2. 核心方法

### 2.1 诊断：policy over growing transcript 的两类负担

把搜索 agent 形式化为"在增长 transcript 上的策略"，policy 实际背了两副担子：

| 负担 | 内容 | 该由谁承担 |
| ---- | ---- | ---------- |
| **语义决策（semantic）** | 搜什么 query、留/弃哪些文档、验证哪条断言、何时停止 | **policy**（必须学，高价值） |
| **可恢复记账（recoverable bookkeeping）** | 记住看过什么、哪些是有用证据、哪些约束未满足、哪些已核实、上下文怎么排布 | **环境**（可靠可维护，不必学） |

> 核心论点：**"reinforcement learning is forced to optimize both semantic search decisions and recoverable bookkeeping that the environment can maintain more reliably."** ——把第二副担子留在 policy 里，是样本效率与泛化的双重浪费。

### 2.2 Harness-1：环境侧维护的"工作记忆"

Harness-1 是一个 **20B 的检索子 agent（retrieval subagent）**，在一个 **stateful search harness** 内用 RL 训练。harness **维护环境侧 working memory**，包含六类结构化状态：

| # | 环境侧工作记忆组件 | 作用 |
| - | ------------------ | ---- |
| 1 | **candidate pool（候选池）** | 汇集检索回来的候选文档/片段 |
| 2 | **importance-tagged curated set（重要性打标的精选集）** | 从候选池中筛出、按重要性标注的"该留"集合 |
| 3 | **compact evidence links（紧凑证据链接）** | 指向支撑证据的紧凑引用，而非全文堆叠 |
| 4 | **verification records（验证记录）** | 记录哪些断言已被实际核实 |
| 5 | **compressed & deduplicated observations（压缩去重的观测）** | 把海量原始观测压缩、去重后保存 |
| 6 | **budget-aware context rendering（预算感知的上下文渲染）** | 在 token 预算内把上述状态**渲染**回 policy 的上下文 |

policy 则**只保留语义决策**：what to search、which documents to keep or discard、what to verify、when to stop。

$$
\underbrace{\text{Policy（20B）}}_{\text{语义决策：搜/留弃/验/停}}\;\rightleftarrows\;\underbrace{\text{Harness（环境侧）}}_{\substack{\text{candidate pool · curated set · evidence links}\\\text{verification records · dedup observations}\\\text{+ budget-aware context rendering}}}
$$

**关键直觉**：把"记账"交给一个**确定性、可靠**的环境组件，比让一个随机性的 LLM policy 顺手维护要稳得多；同时 policy 的上下文不再被原始观测淹没（对抗 context suffocation）。

### 2.3 用户关心的三维度拆解：表示 / 更新 / 使用

| 维度 | Harness-1 的做法 |
| ---- | ---------------- |
| **① 怎么表示（Representation）** | working memory = **环境侧的结构化对象集合**（candidate pool / curated set / evidence links / verification records / 压缩去重观测），而非 policy 上下文里的自然语言 transcript。表示是**显式、带类型、带重要性标签**的。 |
| **② 怎么更新（Update）** | 由 **harness（环境）自动维护**：检索结果入 candidate pool；按重要性筛入 curated set；证据转成 compact links；核实结果写入 verification records；观测压缩去重。policy **不负责**这些写操作，只通过语义动作（留/弃/验）**间接**影响状态。 |
| **③ 怎么使用（Usage）** | 通过 **budget-aware context rendering**：在 token 预算约束下，把当前最相关的 curated 状态**渲染**回 policy 上下文供下一步推理。使用是**按预算、按重要性筛选后注入**，而非全量回灌。 |

> 摘要未披露：harness 各组件的具体数据结构、importance 打标与压缩去重的算法、budget-aware rendering 的排序/截断策略、RL 的具体算法（reward 设计、是否 GRPO 类）。——**需读 PDF / 源码**。

---

## 3. 关键实验结果

| 评测设置 | 结果 | 说明 |
| -------- | ---- | ---- |
| **8 个检索基准**（web / finance / patents / multi-hop QA）平均 **curated recall** | **0.730** | 比"次优开源搜索子 agent"**+11.4 分** |
| 与 frontier-model 搜索器对比 | **competitive**（保持竞争力） | 用 20B 体量逼近大得多的前沿模型搜索器 |
| **held-out transfer 基准** | **增益尤其强** | 支持"在显式搜索状态上做 RL → 检索行为跨域泛化"的论点 |

> **核心实验信号**：泛化增益**集中在 held-out transfer**，这正是"把记账外化、让 RL 专注语义决策"应当带来的效果——学到的是**可迁移的搜索策略**，而非过拟合训练域的记账技巧。
> 摘要未披露：八个基准的逐项分数、与具体基线（哪些开源/前沿搜索器）的对照表、ablation（去掉某个 harness 组件的影响）、训练数据规模与 RL 细节——**需读 PDF**。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **把"working memory"确立为 agent 设计的一等公民**：明确区分"policy 该学的语义决策"与"环境该维护的可恢复记账"，并给出**六类具体的环境侧工作记忆组件**——这为后续 agent harness 设计提供了可直接借鉴的清单。
2. **state externalization 作为 RL 的"减负"原则**：把可恢复状态移出 policy，让 RL 的优化目标更纯粹，**held-out transfer 增益**是这条原则最有力的证据。与 [[10-externalization-efficiency]] 的理论框架互为表里。
3. **"curated recall"作为 working-memory 质量的可操作指标**：相比"最终答案对错"，curated recall 直接衡量**工作记忆把对的证据留下来的能力**，是更贴近 working memory 本质的度量。

### ⚠ 局限

- harness 的**确定性记账规则是人工设计的**——重要性打标、压缩去重、budget rendering 的具体策略若是 hand-crafted，则把"人类先验"从 policy 转移到了 harness，未必最优（对比 [[11-memskill-memento]] 主张把这些操作也**学出来**）。
- **budget-aware rendering 的"删减"是否总是安全**，本文未在 regime 层面分析——[[02-masking]] 恰恰证明"删 stale 观测"的收益强烈 regime-dependent，Harness-1 的 rendering 可能同样存在边界。
- 仅在**检索/搜索**域验证；对编码、计算机使用等其他长程域的可迁移性未知。
- 20B 单模型 + RL，**训练成本与 harness 工程复杂度**未量化。

### 🔮 揭示的趋势

1. **agent = policy + harness 的二分法**：能力越来越多地由"重组 policy 周围的运行时"获得，而非改权重（与 [[10-externalization-efficiency]] 的"weights → context → harness"演进一致）。
2. **working memory 的"环境侧化"**：candidate pool / evidence links / verification records 这类结构，可能成为长程 agent 的标准基础设施。
3. **下一步是"自演化的 harness"**：把 harness 的记账策略本身变成可学习/可优化的（[[11-memskill-memento]] 的 memory skills、[[#rho]] 的 Retrospective Harness Optimization 已指向此）。

### 📊 同方向工作

- [[03-fs-researcher]]：把工作区外化到**文件系统**——同样是"state externalization"，载体不同（文件系统 vs 结构化 harness 对象）。
- IterResearch（`deep-research/02`）：把记忆外化为**演进报告** + 周期性重构——更偏"周期性重置"而非"持续维护结构化状态"。
- GAM（`deep-research/11`）：把完整历史外化到 **page-store**，运行时按需检索——更偏"无损保管 + 运行时编译"。
- [[10-externalization-efficiency]]：把上述所有工作统一到 **externalization** 框架，并给出"weights → context → harness"的历史脉络。
- [[02-masking]]：Harness-1 的 budget rendering 做"该渲染什么"，Masking 研究"该删什么"——互补的 context 使用侧问题。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2606.02373
- **HF Papers**: https://huggingface.co/papers/2606.02373 （52↑）
- **机构**: Chroma（chromadb）
- **GitHub**: https://github.com/pat-jj/harness-1 （548★）
- **开源内容**: Harness-1 训练代码与 stateful search harness 实现
