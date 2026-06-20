# 研究点提案：PhyloSearch —— 进化-系统发生式树记忆的 Deep Research Agent

> **一句话**：把 deep research agent 的工作记忆做成一棵**会进化的"证据-假设树"**——每个节点是一个**自包含的 Markovian 状态**（压缩 summary + 最新检索结果），用**生物系统发生学/进化算子**（分支/变异、选择/剪枝、谱系上卷、趋同验证）在搜索过程中维护这棵树，从而在长程检索任务上既**省 context** 又**可溯源**、还能**并行 + 可恢复**。
> **状态**：研究点提案（idea → method → 实验设计），不含实现。
> **关系**：补上 [`00-survey`](00-survey-tree-memory-evolution.md) 指出的空白——"检索式 search + 树状记忆 + 搜索中演化"。

---

## 1. 动机与空白（Why）

### 1.1 现状的两条独立主线，从未拼到一起

- **检索式 deep research 的记忆是"扁平"的**：Harness-1（环境侧 candidate pool / evidence links，arXiv 2606.02373）、GAM（JIT page-store，2511.18423）、IterResearch（单条演进报告 evolving report，2511.07327）、SearchSwarm（单层委派，2606.09730）——它们解决了"长程 context 膨胀"，但记忆**不是一棵树**，更**不在搜索中按进化方式演化**。
- **"树即记忆 + 进化"只在非检索场景出现**：Arbor（hypothesis tree `⟨h, ι, μ⟩` + 四操作进化，2606.11926）证明这套范式在 long-horizon 任务极强，但它搜索的是**代码/模型的改动空间**（MLE-Bench），**没有 corpus/web retrieval**。

> **空白**：没有人做"**检索式 deep research agent，其工作记忆是一棵随检索演化的证据-假设树**"。本提案就填这个空。

### 1.2 用户的核心 idea = 本提案的技术内核

> **"每个树节点是一个独立状态，包括了压缩的 summary + 最新检索结果。"**

这一句同时踩中两个已被验证有效、但从未结合的设计：
- **Arbor 的"节点承载语义记忆"**——节点的 insight 是 *"compact semantic memory"*（不是原始 transcript）；
- **IterResearch 的"Markovian state reconstruction"**——每个状态**自包含**、不靠 context 累积，周期性把发现合成进紧凑状态。

**把"节点 = 自包含 Markovian 状态"做成树**，带来三个 Arbor / IterResearch 各自都没有的性质：
1. **可并行**——节点状态自包含 ⇒ 不同分支可独立、并行扩展（IterResearch 是单线、Arbor 的并行靠 git worktree 隔离代码）；
2. **可恢复 / 可回溯**——任一节点状态完整 ⇒ 可从任意祖先节点 resume、可回退到任意分支（扁平记忆做不到）；
3. **可剪枝而不丢信息**——剪掉子树前先把其证据上卷到父节点 ⇒ "压缩 summary"保证剪枝无损（解决 Masking 2606.00408 揭示的"删观测有时有害"的 regime 依赖）。

---

## 2. 方法（How）—— PhyloSearch

### 2.1 数据结构：进化证据树（Phylogenetic Evidence Tree）

一棵树 $T$，节点 $v$ 是一个**自包含 Markovian 状态**：

$$v = \langle\, q_v,\; S_v,\; R_v,\; E_v,\; \phi_v,\; \pi_v \,\rangle$$

| 字段 | 含义 | 借鉴 |
| --- | --- | --- |
| $q_v$ | **子问题 / 局部假设**（这条分支在查什么） | Arbor $h_n$ |
| $S_v$ | **压缩 summary**——到本节点为止该分支已确立的紧凑结论（语义记忆，非 transcript） | Arbor $\iota$（compact semantic memory）+ IterResearch evolving report |
| $R_v$ | **最新检索结果**——本节点这一步 search/browse/grep 拿回的原始证据片段 | （用户 idea 的"最新检索结果"） |
| $E_v$ | **证据指针集**——指向原始来源（URL / doc span），材料本身不复制进树 | Arbor μ 的"reference 不复制材料" |
| $\phi_v$ | **适应度**——本节点证据的支持度 / verification 分数 / 置信度 | 自然选择的显式版 |
| $\pi_v$ | **谱系指针**——parent + 关键祖先 insight 引用（溯源链） | 生物 lineage tracking |

- **边语义**：子节点是父 $q$ 的**细化 / 替代 / 互补检索方向**（对应 Arbor 的"refinement / alternative / correction"）。
- **关键不变量（Markovian self-containment）**：扩展节点 $v$ 只需读 $v$ 自己的 $(q_v, S_v, R_v)$ + 祖先的压缩 $S$，**不需要重放整条 transcript** ⇒ context 恒定、可并行、可 resume。

### 2.2 四个进化算子（生物 ↔ agent 对照）

整个搜索 = 在这棵树上反复施加四个算子，直到收敛/预算耗尽：

| 算子 | 生物对应 | 机制 | 借鉴 |
| --- | --- | --- | --- |
| **① Expand / Mutate（展开-变异）** | 突变 + 适应辐射 | 对一个未决/低置信节点，生成 $k$ 个**多样化**子检索方向（不同来源类型 / 不同角度），每个子节点独立执行一次检索得 $R$ | Arbor Ideate；**MAP-Elites 式多样性**（GigaEvo 2511.17592）防"分支塌缩" |
| **② Select / Prune（选择-剪枝）** | 自然选择 | 按 $\phi_v$ 剪掉被证伪/冗余子树；**剪前先把 $S_v$ 上卷到父**（无损剪枝） | Arbor prune + Masking 的"何时删"regime 意识 |
| **③ Coalesce / Abstract（谱系上卷）** | 系统发生 + coalescent | 沿叶→根把子节点 $S$ 抽象成父节点 $S$（$S_a \leftarrow \mathrm{Abstract}(\{S_c\})$）；多分支**汇合**到同一结论时合并谱系 | Arbor backprop+Abstract；coalescent 的"多样本回溯共祖" |
| **④ Verify-by-Convergence（趋同验证）** | 趋同演化 | **多条独立检索分支指向同一事实 ⇒ 置信度↑**；矛盾 ⇒ 标记冲突待裁决 | ARIS 跨模型验证（huggingface/07）；趋同演化作信号 |

> **与 Arbor 四操作的精确差异**：Arbor 的算子作用在"代码改动 + git worktree"，PhyloSearch 作用在"检索证据 + summary 状态"；Arbor 无 ②的"无损剪枝上卷"、无 ④的"趋同验证"（这两个是检索场景特有、且本提案的新机制）。

### 2.3 记忆的"使用"：budget-aware frontier 投影

policy 每步**不读整棵树**，只读一个**结构化投影**：活跃 frontier 节点的 $(q, S, R)$ + 根到 frontier 路径上的祖先 $S$ + 当前最优结论。
- 对应 Harness-1 的 budget-aware rendering + Arbor 的"structured projection"。
- 因为每个 $S$ 是压缩的、节点自包含 ⇒ 投影大小与树规模**解耦**（解决 mono-context 膨胀）。

### 2.4 控制循环（仿 Arbor 六步，检索化）

```
Observe(读 frontier 投影) → Ideate(为未决节点想检索方向)
→ Select(挑要扩展的节点) → Dispatch(并行执行子节点检索)
→ Coalesce+Verify(上卷 summary + 趋同验证 + 剪枝) → Decide(够不够答 / 继续)
```

---

## 3. 训练（可选两条路线）

- **路线 A：Training-free / prompt-orchestrated**（先做这个，低成本验证 idea）——树的维护全靠 prompt + 工具，policy 用现成 frontier LLM。对标 Arbor（它本身就是 orchestration、不训权重）。
- **路线 B：RL 化**——把"树的维护（剪枝/上卷/记账）"**外置给环境**（仿 Harness-1），RL 只学**语义决策**：扩展哪个节点、生成什么检索方向、何时停、采纳哪个结论。奖励 = 最终答案正确性 + 证据支持度 + 效率惩罚（仿 IterResearch EAPO 的几何折扣）。
  - 可叠加 SDAR / Agentic GRPO 的多轮信用分配（`agentic-rl/08`、`01`）。

---

## 4. 实验设计（Experiments）

### 4.1 任务与基准

- **主战场（长程检索 DR）**：**BrowseComp / BrowseComp-ZH**（最对口，强调多跳深检索）、**K-BrowseComp**（2606.02404，多语言抗饱和）、**GAIA**、**FRAMES**、**HotpotQA / MuSiQue / 2Wiki**（多跳 QA，可控验证）。
- **过程级评测**：用 **DRIFT / TELBench**（2606.02060）做 span 级错误定位——验证"树结构是否真的减少了 unsupported/conflicting claim"。
- **可溯源性**：报告 citation precision（每个 claim 能否追溯到 $E_v$ 来源）——这是树结构相对扁平记忆的**独特卖点**。

### 4.2 基线（Baselines）

| 类别 | 基线 | 说明 |
| --- | --- | --- |
| 扁平记忆 DR | ReAct、IterResearch（演进报告）、Harness-1（环境侧记忆） | 验证"树 vs 扁平" |
| 树/搜索但非演化记忆 | MCTS / Tree-of-Thought 风格搜索、SearchSwarm（单层委派） | 验证"演化树 vs 普通树搜索" |
| 树+进化但非检索 | Arbor 思路迁到检索的 naive 版（无②④新算子） | 验证本提案的②无损剪枝、④趋同验证的增量 |

### 4.3 关键消融（Ablations）—— 每个都对应一个 claim

1. **去掉树（退化成单条 evolving report）** → 验证"分支结构"的价值（多路并行 vs 单线）。
2. **去掉②无损剪枝上卷**（直接删子树）→ 验证"剪枝前上卷 summary"是否真的无损（对照 Masking 的有害 regime）。
3. **去掉③谱系上卷**（叶证据不抽象到父）→ 验证长程结论质量是否退化。
4. **去掉④趋同验证**（不利用多分支一致性）→ 验证 citation precision / 事实正确性下降。
5. **去掉 MAP-Elites 式多样性**（①贪心单方向展开）→ 验证是否出现"分支塌缩"（对应 VideoDR 的 agentic floor effect）。
6. **节点状态是否真自包含**：测"从任意节点 resume / 回退分支"的能力——这是扁平记忆**做不到**的对照实验。

### 4.4 度量

- **质量**：答案正确率 / item F1（BrowseComp 类）、多跳 EM（HotpotQA 类）。
- **效率**：达到同等质量的 token / 工具调用数（对标 IterResearch 的 efficiency scaling）。
- **可溯源**：citation precision、first-error step（DRIFT）。
- **结构性**：平均树深/宽、剪枝率、趋同验证触发率 vs 正确率的关系（分析性图表，类似 IterResearch 的"交互次数 vs 性能"曲线）。

---

## 5. 新颖性与定位（Novelty —— 投稿时怎么讲）

> **One-liner**：*"We are the first to make a deep-research search agent's working memory an **evolving phylogenetic evidence tree** of self-contained Markovian states, where retrieval branches are mutated, selected, and coalesced via biology-inspired operators."*

| vs 谁 | 它有什么 | **我们多了什么** |
| --- | --- | --- |
| **Arbor** (2606.11926) | 树即记忆 + 四操作进化 | **检索/证据场景**（它做代码优化无 retrieval）+ **无损剪枝上卷** + **趋同验证** |
| **IterResearch** (2511.07327) | Markovian 自包含状态 | **把单条状态做成树**（可并行/可恢复/可回溯）+ 进化算子 |
| **Harness-1** (2606.02373) | 环境侧 working memory + RL | **结构是树而非扁平 candidate pool** + 谱系溯源 |
| **MCTS / ToT** | 树搜索 | **树是持久演化记忆**（非用完即弃）+ **节点是语义状态**（非单步推理）+ 检索化 |
| **GigaEvo / FunSearch / CSE** | 进化算子 / QD | **作用在检索证据树**（非代码/数学解）+ 与 DR 工作记忆合一 |

**三个可声明的 contribution**：
1. **PhyloSearch 框架**——首个"演化证据树即工作记忆"的检索式 DR agent；节点 = 自包含 Markovian 状态（压缩 summary + 最新检索）。
2. **两个检索场景特有的进化算子**——无损剪枝上卷（Coalesce-before-Prune）+ 趋同验证（Verify-by-Convergence）。
3. **系统实验**——在 BrowseComp / GAIA / 多跳 QA 上 vs 扁平记忆 / 普通树搜索 / Arbor-naive 的全面对比 + 6 个消融，并报告**可溯源性**这一树结构独有指标。

---

## 6. 与生物进化树的概念对照（投稿 intro 的"灵感"段）

| 生物 | PhyloSearch |
| --- | --- |
| 系统发生树（分支累积历史） | 证据树（检索历史沉淀为可累积结构） |
| 突变 + 适应辐射 | 多样化展开检索分支（① MAP-Elites 式） |
| 自然选择 | 按证据支持度剪枝（②） |
| coalescent（回溯共祖） | 多分支汇合到同一结论 + 谱系溯源（③） |
| 趋同演化（独立谱系得相似性状=强信号） | 多路独立检索得同一事实 = 高置信（④） |
| lineage tracking | 每个 claim 可溯源到 $E_v$ |

> **关键区别（也是卖点）**：生物系统发生树是**事后重建**的描述性历史；PhyloSearch 的证据树是 agent **运行时正向生长 + 主动剪枝**的**生成性**工作记忆——把"进化/系统发生"从"还原过去"变成"驱动搜索"。

---

## 7. 风险与缓解（Risks）

| 风险 | 缓解 |
| --- | --- |
| **Abstract/上卷丢关键细节**（压缩 summary 的老问题，DreamGym/IterResearch 都有） | $E_v$ 保留原始来源指针，必要时可"展开"回原文；消融测压缩率 vs 质量 |
| **树膨胀 / 维护开销** | budget-aware frontier 投影 + 激进剪枝；报告树规模 vs 收益曲线 |
| **趋同验证被"共同错误源"骗**（多分支都引同一个错网页） | 趋同要求**来源多样性**（不同 domain/类型才算独立证据），借 ARIS 跨源思想 |
| **新算子收益不显著**（可能只是 Arbor-naive 就够好） | 消融 2/4 专门隔离②④的增量；若不显著，contribution 退守"框架 + Markovian 树节点"仍成立 |
| **训练成本（路线 B）** | 先交付路线 A（training-free）证明 idea，RL 作为加分项 |

---

## 8. 最小可行验证路径（建议执行顺序）

1. **MVP（training-free，2-3 周）**：prompt 编排实现证据树 + 四算子，在 **HotpotQA / 2Wiki**（便宜、可自动判分）上 vs ReAct / IterResearch-style 单报告，先看"树 + ④趋同验证"是否提升多跳 EM + citation precision。
2. **扩到 BrowseComp / GAIA**：验证长程真实检索下的 token 效率（对标 IterResearch 的 scaling 曲线）。
3. **消融 1-6**：逐个验证 claim。
4. **（可选）RL 化（路线 B）**：把树维护外置给环境，RL 学语义决策。
5. **过程级评测（DRIFT）+ 写作**。

---

## 附：与本仓库的交叉引用

- 空白论证：[`tree-memory/00-survey`](00-survey-tree-memory-evolution.md)
- Arbor（节点 schema / 四操作）：`memory/00-summary` A 类、本目录 survey 第二节
- IterResearch（Markovian state）：`deep-research/02-iterresearch`、`agentic-rl/11-iterresearch`
- Harness-1（状态外置 + RL）：`memory/01-harness-1`、`agentic-rl/13-harness-1`
- Masking（何时删观测的 regime）：`memory/02-masking`、`deep-research/`（2026-06 #28）
- GigaEvo / CSE / BES（进化算子 / QD）：`evolve/13-cse`、`evolve/00-summary`（GigaEvo / BES 见 2025-11 / 2026-05）
- ARIS（趋同/跨源验证）：`huggingface/07-aris`
- DRIFT（过程级评测）：`deep-research/13-dr-eval-and-error`
