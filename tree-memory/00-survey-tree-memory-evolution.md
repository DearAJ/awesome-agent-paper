# 调研：Search Agent Memory × Tree × Evolution（树结构记忆 + 进化）

> **调研问题**：想做「**search agent 的记忆 + 树结构 + 进化**」结合的研究——有没有相关工作？空白在哪？
> **数据范围**：本仓库 `evolve/` `memory/` `deep-research/` `agentic-rl/` 四个 00-summary（2025-11 ~ 2026-06 月榜）+ HuggingFace 2026-06 月榜全部 105 篇核对 + Arbor 全文（arXiv 2606.11926）。
> **一句话结论**：三者**真正合一**的只有 **Arbor**（hypothesis-tree 即记忆、四操作即进化）——但它做的是**自主科研/代码优化**，**不是检索式 search/deep research**。**「检索式 search agent + 树状工作记忆 + 在搜索中演化」这个交叉点，目前是空白。**
> **标记**：🔴 三者合一｜🟠 树+记忆｜🟡 树+进化｜🟢 记忆+进化｜🔵 单主题/背景

---

## 一、三条线的定义与交叉地图

把问题拆成三个维度，先说清各自指什么：

| 维度 | 在本调研里指什么 | 典型机制 |
| --- | --- | --- |
| **① Search-Agent Memory（搜索智能体记忆）** | 长程搜索 / deep research 中那团**不断增长的"当前状态"**：看过哪些页、哪些证据有用、哪些子问题/约束还没满足、哪些 claim 已被核实 | growing transcript / 环境侧结构化状态（candidate pool, evidence links, verification records）/ 文件系统 / 演进报告 / page-store |
| **② Tree（智能体用的树）** | agent 显式维护的**树/分层结构** | hypothesis tree / MCTS / Tree-of-Thought / skill network / E²R 树搜索 / 证据树 / 委派树 |
| **③ Evolution（进化）** | **变异 / 选择 / 淘汰 / 经验蒸馏 / 协同进化** —— 即"候选不断产生、好的保留、坏的剪掉、经验上卷复用" | mutation+crossover、prune 被证伪分支、insight 上卷、co-evolution、skill 库进化、patch 式演化 |

### 三圈维恩图（每篇论文落在哪个区）

```
                        ┌─────────────────────────────────────┐
                        │            ② TREE 树                 │
                        │   MCTS · Tree-of-Thought（背景）      │
                        │                                      │
          ┌─────────────┼──────────────┐      ┌────────────────┼─────────────┐
          │   🟠 树+记忆 │              │      │  🟡 树+进化     │             │
          │  GAM(2511.   │   🔴 三者合一 │      │  PSN(2601.03509)│             │
          │  18423)      │  ┌──────────┐│      │  CORAL(2604.    │             │
          │  IterResearch│  │ Arbor     ││      │  01658)         │             │
          │  (2511.07327)│  │(2606.11926)│      │                 │             │
          │              │  │ OneManCo. ││      │                 │             │
          └──────────────┼──│(2604.22446)│──────┼─────────────────┘             │
   ① SEARCH-AGENT        │  └──────────┘│      │          ③ EVOLUTION 进化     │
      MEMORY 记忆         │   🟢 记忆+进化 │      │  SkillRL/Skill1/MemSkill      │
   Harness-1(2606.02373) │  Harness-1·MIA│      │  EvoArena·EvoMem(2606.13681)  │
   SearchSwarm(2606.09730)│ EvoArena···· │      │  SCOPE/OpenSkill/Sleep        │
   MIA(2604.04503)       └───────────────┘      └───────────────────────────────┘
```

> **怎么读这张图**：
> - **🔴 正中（三圈交叠）只有 Arbor + OneManCompany**——但二者都**不做检索式 search**（Arbor 做代码/模型优化，OneManCompany 做软件公司式任务编排）。
> - **① 记忆圈里做"检索式 search"的（Harness-1 / SearchSwarm / GAM / MIA / IterResearch）全部不在 ② 树圈的核心**——它们的记忆是**扁平的环境对象 / 文件系统 / 演进报告 / page-store**，不是一棵树。
> - **所以真正的空白 = ①∩②∩③ 里"检索式"的那块**：把树状记忆 + 进化用到**对 corpus/web 做检索与证据收集**上。详见第四节。

---

## 二、交叉点逐篇梳理

### 🔴 三者合一（Tree + Memory + Evolution）

#### Arbor: Toward Generalist Autonomous Research via Hypothesis-Tree Refinement — arXiv 2606.11926（Microsoft / RUC，2026-06，100↑）

**这是目前唯一把「树 + 记忆 + 进化」真正做成一体的工作**，也是最值得用户细读的参照。但它的场景是**自主科研 / 代码与模型优化（MLE-Bench）**，**不是检索式 deep research**。

- **树即记忆（Hypothesis Tree = 工作记忆）**：每个节点 `n = ⟨h_n, ι_n, μ_n⟩`：
  - **h_n（hypothesis）**——一个可验证/可证伪的"对产物做某种改动会更好"的声明；**粒度随深度递进**：靠根的节点是宽方向，越深越是具体干预。
  - **ι_n（insight）**——论文明确称之为 **"a compact semantic memory"（紧凑语义记忆），而不是 execution transcript**；内部节点的 insight 是对其子节点的**抽象**。← 这正是"树承载记忆"的关键。
  - **μ_n（metadata）**——status、dev score、factual result、**一个 git branch/commit 引用**（实现指针，材料本身不复制进树）、可选 background。
  - **边语义**：内部节点=方向节点，叶=可执行干预；每个子节点是父 hypothesis 的"细化 / 替代 / 修正"。
- **进化（四操作 = 变异/选择/淘汰/采纳）**：coordinator 跑六步循环 `Observe → Ideate → Select → Dispatch → Backpropagate → Decide`，对应四个 refinement 操作：
  1. **Update tree（Backpropagate）**——executor 返回时把 `(score, result, insight, branch ref)` 写回叶节点；
  2. **Propagate reusable lessons（Abstract）**——沿"叶→根"路径**把子节点 insight 上卷成祖先 insight**（`ι_a ← Abstract({ι_c})`），一个叶级观察能升级为方向级约束、最终成"全局先验"。← 这是**经验泛化/变异**；
  3. **Refine search frontier（Ideate/Select/prune）**——展开有希望的叶、**剪掉被返回 insight 证伪的子树**。← 这是**选择/淘汰**；
  4. **Admit verified improvements（Decide/merge gate）**——候选只有在 **held-out evaluator 上确有提升**才 `merge` 进 `M_best`。← 这是**适应度门控**。
- **记忆如何被读**：coordinator **不做学习式检索/排序**，而是读树的"结构化投影"——active frontier 节点 + 近期证据 + 祖先 insight + 当前最优。选择是"**partial & delayed feedback 下的 frontier control**"（不是纯 score 最大化）。
- **执行**：每个 executor 在一个**从 M_best 初始化的全新 git worktree** 里实现被指派的 hypothesis（hypothesis-bound：只改实现 Δ、h_n 固定），返回 `(score, result, Distill(...), commit)`。
- **结果**：MLE-Bench Lite 用 GPT-5.5 拿 **86.36% Any Medal**（其对比中最强）；6 个真实研究任务（模型训练 / harness 工程 / 数据合成）**held-out 全部第一**，平均相对 held-out 增益 **> Codex 与 Claude Code 的 2.5 倍**。开源 `RUC-NLPIR/Arbor`。
- **🔑 与本调研的关系**：Arbor 证明了"**树状记忆 + 树的进化式精炼**"在 long-horizon 任务上极强——但它**没有 corpus/web retrieval**，搜索的是"改动空间"而非"信息空间"。**把这套树-记忆-进化机制迁到检索/证据收集，就是空白区。** 仓库交叉引用：`memory/00-summary`（A 类 Arbor）、`evolve/`（隐含）。

#### OneManCompany: Organising Heterogeneous Agents as a Real-World Company — arXiv 2604.22446（2026-04，121↑）

- **树**：核心是 **Explore-Execute-Review（E²R）树搜索**——任务自顶向下分解为"可问责单元"、结果自底向上聚合驱动审查精炼，**形式化保证终止性与无死锁**。
- **记忆**：Talent（封装 skill/tool/config 的可移植身份）+ Talent Market 构成**组织级共享记忆**（谁会什么）。
- **进化**：按需招募 / 动态重配 = 组织级自改进。PRDBench **84.67%（+15.48pp）**。
- **🔑 关系**：E²R 是"任务规划树"而非"检索记忆树"，做的是企业式任务编排不是 search。**详见 `evolve/00-summary`（2026-04 #28）**。

### 🟠 树 + 记忆（缺"进化"这一维）

#### General Agentic Memory (GAM) — arXiv 2511.18423（BAAI，2025-11，171↑）
- **记忆 + 隐含层级**：借编译器 **JIT** 原则——universal **page-store 无损保管完整历史**（层级化），运行时由 Researcher "像做 deep research 一样"按需检索重建。记忆是**层级 page-store** 而非演化的树。**详见 `memory/00-summary`（A 类）、`deep-research/00-summary`（2025-11 #5）**。

#### IterResearch — arXiv 2511.07327（阿里 Tongyi，2025-11，80↑）
- **记忆**：维护 **evolving report（演进报告）** 作工作记忆，周期性 Markovian 重构（合成洞见→重置工作区）。但 report 偏**线性/扁平**，不是分支树。**详见 `deep-research/00-summary`（2025-11 #36）、`memory/00-summary`（A 类）**。

### 🟡 树 + 进化（缺"记忆即那棵树"）

#### PSN: Evolving Programmatic Skill Networks — arXiv 2601.03509（蒙特利尔大学，2026-01，88↑）
- **树/网 + 进化**：skill = 可执行符号程序，组成**可组合网络**随经验进化（REFLECT 故障定位 + maturity-aware gating + 规范化重构 + rollback 验证）。是 **skill 进化网络**，不是"搜索证据的记忆树"。发现"skill 进化 ≈ 神经网络训练"同构。**详见 `evolve/00-summary`（2026-01 #37）**。

#### CORAL: Autonomous Multi-Agent Evolution for Open-Ended Discovery — arXiv 2604.01658（MIT，2026-04，55↑）
- **层级 + 进化 + 共享记忆**：long-running agents 通过**共享持久记忆 + 异步多 agent 执行 + heartbeat 干预**做开放式发现。10 任务 SOTA，kernel 优化 **1363→1103 cycles**。多 agent 层级是组织结构，记忆是共享池非树。**详见 `evolve/00-summary`（2026-04 #78）**。

#### 背景基线（仓库外）
- **MCTS / Tree-of-Thought**——经典"用树做推理搜索"，但**树是临时推理结构、用完即弃，不是持久记忆，也不跨任务进化**。它们是本方向的思想源头，但缺"记忆持久化"与"进化复用"两维。

### 🟢 记忆 + 进化（缺"树"这一维）

这一区**最密集**——大量工作在做"会演化的搜索记忆/skill 库"，但记忆都是**扁平结构**（环境对象、列表、patch、skill 库），没人把它做成树。

| 论文 | arXiv | 记忆形态 | 进化机制 | 仓库位置 |
| --- | --- | --- | --- | --- |
| **Harness-1**（最接近"检索式"） | 2606.02373 | 环境侧 working memory：candidate pool / curated set / **evidence links** / verification records | RL 优化语义决策 + 环境维护可恢复状态 | `memory/01` `deep-research/`（2026-06 #45）`agentic-rl/13` |
| **SearchSwarm**（最接近"类树"） | 2606.09730 | 主 agent context + subagent 返回的 **summary** | 委派=**单层树**（主→子），summary 省 context 预算 | `deep-research/`（2026-06 #48）`agentic-rl/` |
| **MIA** | 2604.04503 | 非参数轨迹库 + 参数 Planner | **参/非参记忆双向转换循环** | `memory/00-summary`（A 类） |
| **EvoArena / EvoMem** | 2606.13681 | **patch-based 记忆**（结构化 update history） | 记忆随环境演化追加 patch | `memory/08` `evolve/`（2026-06 #14） |
| **SkillRL / Skill1 / MemSkill** | 2602.08234 / 2605.06130 / 2602.02474 | 层级 SkillBank / skill 库 / 记忆 skill | RL 递归进化 / 统一三能力 / 记忆操作可学化 | `evolve/` `memory/` |
| **SCOPE / OpenSkill / Need Sleep** | 2605.31433 / 2606.06741 / 2606.03979 | 自造任务-rubric / 自建 skill+verifier / 短期→长期参数 | 开放世界 self-play / bootstrap / 睡眠巩固 | `evolve/` `memory/` |

> **关键观察**：**SearchSwarm 的委派树**（主→子 agent）和 **Harness-1 的 evidence links** 是最接近"检索 + 类树 + 记忆"的两块拼图——但 SearchSwarm 只有**单层**委派、且子 agent 返回的是 summary 不是持久树节点；Harness-1 的 evidence links 是**扁平集合**不是分支树。**没有人把"检索证据 + 子问题"组织成一棵会剪枝/上卷/演化的树。**

---

## 三、2026-06 月榜补充扫描结论

已用本地缓存核对 HuggingFace 2026-06 月榜**全部 105 篇**（不止 top50），逐篇筛 tree / memory / search / evolution 关键词：

- **tree + memory + 检索式 search 三者交叉的新论文，只有 Arbor（2606.11926）一篇**。
- 其余相关命中均属"已在仓库"或"单/双主题"：EvoArena(2606.13681)、Harness-1(2606.02373)、SearchSwarm(2606.09730)、FORT-Searcher(2606.12087)、Masking Stale Observations(2606.00408)、Where-Do-DR-Go-Wrong/DRIFT(2606.02060)、Agentic Environment Engineering Survey(2606.12191)、LatentSkill(2606.06087)、LongTraceRL(2605.31584)、Echo-Memory(2606.05080)、Language Models Need Sleep(2606.03979)、OpenSkill(2606.06741)、SCOPE(2605.31433)、TaskMem(2605.31075)、SkillAdaptor(2606.01311)。
- **结论**：这个交叉点**确实新、确实空**——即便在论文密度最高的 2026-06，也没有"检索式 search agent + 树状工作记忆 + 搜索中演化"的工作。

---

## 四、空白区与机会（核心结论）

### 已覆盖 vs 空白的组合

| 组合 | 状态 | 代表 |
| --- | --- | --- |
| 树 + 记忆 + 进化（**非检索**，做优化/科研） | ✅ 有 | **Arbor**、OneManCompany |
| 树 + 记忆（无进化） | ✅ 有 | GAM、IterResearch |
| 树 + 进化（记忆非树） | ✅ 有 | PSN、CORAL、MCTS/ToT |
| 记忆 + 进化（无树） | ✅ 很多 | Harness-1、SearchSwarm、EvoArena、SkillRL… |
| 检索式 search + 记忆（无树、无进化或弱进化） | ✅ 有 | Harness-1、GAM、SearchSwarm、MIA |
| **检索式 search + 树状工作记忆 + 搜索中演化** | ❌ **空白** | —— |

### 空白点的精确定义

> **没有人做：一个对 corpus/web 做检索的 deep-research / search agent，其工作记忆本身是一棵「证据-子问题树」，并在搜索过程中按进化方式演化——展开新检索分支（变异）、按证据支持度剪枝/提升分支（选择/淘汰）、把叶级证据上卷成父级结论（经验泛化）、只把被核实的 claim 采纳进答案（适应度门控）。**

三块拼图各自存在、但从未拼到一起：
- **Arbor** 有"树即记忆 + 四操作进化"，但搜索的是**改动空间**（代码/模型），**没有 retrieval**。
- **Harness-1 / GAM / IterResearch** 有"检索式 search + 工作记忆"，但记忆是**扁平**的（环境对象 / page-store / 演进报告），**不是树、不进化**。
- **SearchSwarm** 有"检索 + 类树（委派）"，但**只单层、子 agent 返回 summary 即弃**，不是持久演化的树。

### 可探索的设计角度（作为"机会"列出，非完整 method）

1. **节点 schema**：树节点 = ⟨子问题/假设，检索到的证据集 + 来源，verification 状态，置信度⟩——直接借 Arbor 的 `⟨h, ι(语义记忆), μ⟩` 但把 ι 换成"证据摘要"、μ 里存证据来源指针而非 git commit。
2. **变异 = 展开检索分支**：对一个未决子问题，生成多个互补的检索方向作为子节点（对应 Arbor 的 Ideate，但工具是 search/grep/browse 而非 code edit）。
3. **选择/淘汰 = 按证据剪枝**：被证据证伪或冗余的子树剪掉（对应 Arbor 的 prune + Masking 的"删过期观测"，但删的是树枝不是 token）。
4. **经验上卷 = 证据→结论**：叶级证据 `Abstract` 成父级中间结论、再上卷成答案要点（对应 Arbor 的 insight 上卷 + IterResearch 的"合成洞见"）。
5. **记忆使用 = budget-aware 渲染 frontier**：只把树的活跃 frontier + 祖先结论渲染回 policy 上下文（对应 Harness-1 的 budget-aware rendering + Arbor 的"结构化投影"），解决长程 context 膨胀。
6. **与 RL 结合**：可像 Harness-1 把"树的维护（剪枝/上卷/记账）"外置给环境、让 RL 只学"搜什么/展开哪个分支/何时停"的语义决策。

> 以上仅为"可做的方向"，若要展开成完整 method/实验设计，可另起一份"研究点提案"。

---

## 五、资源与交叉引用表

| 论文 | arXiv | 交叉类型 | 仓库内位置 |
| --- | --- | --- | --- |
| Arbor | [2606.11926](https://arxiv.org/abs/2606.11926) | 🔴 树+记忆+进化 | `memory/00-summary` A 类 |
| OneManCompany | [2604.22446](https://arxiv.org/abs/2604.22446) | 🔴 树+记忆+进化 | `evolve/00-summary` 2026-04 #28（仅 summary，无独立精读） |
| GAM | [2511.18423](https://arxiv.org/abs/2511.18423) | 🟠 树+记忆 | `deep-research/11-general-agentic-memory`、`memory/00-summary` A 类 |
| IterResearch | [2511.07327](https://arxiv.org/abs/2511.07327) | 🟠 树+记忆 | `deep-research/02-iterresearch`、`memory/00-summary` A 类 |
| PSN | [2601.03509](https://arxiv.org/abs/2601.03509) | 🟡 树+进化 | `evolve/07-psn` |
| CORAL | [2604.01658](https://arxiv.org/abs/2604.01658) | 🟡 树+进化 | `evolve/00-summary` 2026-04 #78 |
| Harness-1 | [2606.02373](https://arxiv.org/abs/2606.02373) | 🟢 记忆+进化（最接近检索式） | `memory/01-harness-1`、`deep-research/09`、`agentic-rl/13-harness-1` |
| SearchSwarm | [2606.09730](https://arxiv.org/abs/2606.09730) | 🟢 记忆+进化（最接近类树） | `deep-research/12-searchswarm-and-wideseek` |
| MIA | [2604.04503](https://arxiv.org/abs/2604.04503) | 🟢 记忆+进化 | `memory/00-summary` A 类 |
| EvoArena / EvoMem | [2606.13681](https://arxiv.org/abs/2606.13681) | 🟢 记忆+进化 | `memory/08-evoarena-evomem`、`evolve/18-evoarena` |
| SkillRL | [2602.08234](https://arxiv.org/abs/2602.08234) | 🟢 记忆+进化 | `evolve/05-skillrl`、`agentic-rl/14-skillrl` |
| Skill1 | [2605.06130](https://arxiv.org/abs/2605.06130) | 🟢 记忆+进化 | `evolve/06-skill1` |
| MemSkill | [2602.02474](https://arxiv.org/abs/2602.02474) | 🟢 记忆+进化 | `memory/11-memskill-memento`、`evolve/` |

### 与生物进化树的概念呼应

这个方向和**生物遗传进化的"树"**在思想上同源——
- **系统发生树 / 进化树（phylogenetic tree）** 和 **coalescent（合并理论，反向时间追溯共同祖先）** 表达的都是"**分支、可累积的历史**"，正如 Arbor 的 hypothesis tree 把"研究历史"沉淀成一棵可累积的树；
- **变异 / 选择 / 淘汰**（生物进化算子）正对应 Arbor 四操作里的"展开分支 / 适应度门控 / 剪枝被证伪子树"；
- 区别在于：生物进化树是**事后重建**的历史记录，而这里的"证据树"是 agent **运行时主动生长并剪枝**的工作记忆——把"进化"从描述性（重建过去）变成生成性（驱动搜索）。

---

## 六、一句话总结

> **想做的「search agent memory + 树 + 进化」，最接近的参照是 Arbor（树即记忆、四操作即进化），但它做优化不做检索；检索式 search 这边（Harness-1 / GAM / SearchSwarm）记忆又是扁平的、不进化。把 Arbor 的「hypothesis-tree-as-evolving-memory」迁到「对 corpus/web 检索 + 证据收集」场景——让证据-子问题组成一棵会展开分支、按证据剪枝、把证据上卷成结论的演化树——就是当前明确的空白与机会。**
