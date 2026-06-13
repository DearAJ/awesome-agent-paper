# Agent Memory 专题 — HuggingFace 月榜（2025-11 至 2026-06）

> **数据范围**：HuggingFace Papers **月榜** 2025-11、2025-12、2026-01、2026-02、2026-03、2026-04、2026-05、2026-06（共 8 个月，每月 top50）
> **筛选焦点**：**Agent Memory**，尤其是 **working memory（工作记忆）**——即 **agent 在多轮 / 长程任务中，随着 transcript / 上下文不断增长，如何表示、更新、使用这部分"当前状态"**。兼收紧邻的 long-context 记忆管理、外化（externalization）、记忆 skill 化等工作。
> **分析三维度（贯穿全专题）**：
>
> **① 怎么表示（Representation）**：记忆以什么形式存在
>
> growing transcript / 环境侧结构化状态（candidate pool、evidence links）/ 文件系统 / 演进报告 / KV-cache / 固定大小关联状态矩阵 / 参数（adapter）/ markdown skill。
>
> **② 怎么更新（Update）**：记忆何时、由谁、按什么策略写入
>
> 与修订LLM 自己边推理边记 / 环境侧自动 bookkeeping / 周期性重构 / delta-rule 在线更新 / patch 历史 / RL 学到的 memorization policy / Sleep 阶段离线巩固。
>
> **③ 怎么使用（Usage）**：记忆如何回到上下文供推理
>
> budget-aware context rendering / 把 stale 观测 mask 掉 / 运行时 deep-research 式按需检索 / 低秩修正注意力 / 分阶段 rubric 召回。

long-horizon agent 的长上下文是一个需要主动管理的、不断增长的状态：

1. **已看过什么**（检索回来的海量 observation）；
2. **哪些证据有用**（evidence）；
3. **哪些约束还没满足**（open constraints）；
4. **哪些断言已被核实**（verified claims）。

朴素做法是把这些全堆进一个**单一、膨胀的 transcript**，让策略模型（policy）一边决定"下一步搜什么"，一边还要充当"记账员"记住上面这一切。多篇论文（[[01-harness-1]]、[[#2-iterresearch（见-deep-research02）]]）共同指出这会导致：

- **context suffocation（上下文窒息）**：有效信息被无关 token 稀释，推理容量被吃满；
- **noise contamination（噪声污染）**：早期失败路径、过时观测一直挂在上下文里干扰决策；
- **RL 目标错配**：强化学习被迫同时优化"语义搜索决策"和"本可由环境可靠维护的记账"，样本效率低、泛化差。

于是 **working memory** 成了独立研究对象：**不是问"agent 长期记住了什么知识"（long-term / episodic / factual memory），而是问"在当前这个长任务里，那团正在增长的状态该怎么表示、怎么更新、怎么用"。** 这正是与传统"agent memory / RAG"的区别——后者重 long-term 知识库，本专题重 **运行时的 working set**。

> **概念坐标**（forms × functions × dynamics 框架）：
>
> - **功能维度**：factual（事实）/ experiential（经验）/ **working（工作）**
> - **形式维度**：token-level / parametric / latent——working memory 主要落在 **token-level（上下文里的结构化状态）**，但 [[07-delta-mem]] 探索了 **latent（固定状态矩阵）**、[[06-qwenlong-l15]] 触及 **parametric**。
> - **动态维度**：formed → evolved → retrieved

---

## 一、按"表示 / 更新 / 使用"三维度归类（核心地图）

> 同一篇论文可能在三个维度都有贡献，下表按其**最具代表性的贡献维度**归位；完整三维拆解见各精读笔记的"表示/更新/使用"小节。

### ① 怎么**表示**（Representation）——记忆以什么形式承载

| 表示形式                                                                                               | 代表论文                                | 一句话                                                        |
| ------------------------------------------------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------- |
| **环境侧结构化工作记忆**（candidate pool + curated set + evidence links + verification records） | **[[01-harness-1]]** ⭐           | 把"记账"从 policy 搬到 harness（环境侧），policy 只留语义决策 |
| **演进报告（evolving report）作为记忆**                                                          | IterResearch（见 `deep-research/02`） | 每轮重建工作区，把洞见合成进一份不断演进的 report             |
| **文件系统（hierarchical knowledge base）作为外部记忆**                                          | **[[03-fs-researcher]]** ⭐       | 文件系统当持久工作区，知识库可远超 context length             |
| **持久树（hypothesis tree）链接假设/证据/洞见**                                                  | [[#hypothesis-tree-arbor]] 📄           | 长程科研中用一棵跨时间的树承载策略与证据                      |
| **固定大小关联状态矩阵（associative memory state）**                                             | **[[07-delta-mem]]** ⭐           | 8×8 在线状态矩阵压缩历史，耦合进注意力                       |
| **KV-cache 作为分层记忆**                                                                        | [[#hermes]] 📄 / MSA 📄                 | 把 KV-cache 重新概念化为多粒度记忆                            |
| **结构化/可演化的 markdown skill 作为记忆**                                                      | [[11-memskill-memento]] ⭐              | 记忆 = 可学习、可演化的"记忆技能"或 skill 文件                |
| **page-store + 轻量记忆（JIT）**                                                                 | GAM（见 `deep-research/11`）          | 离线无损保管完整历史，运行时再现场编译                        |

### ② 怎么**更新**（Update）——何时、由谁、按什么策略写入/修订

| 更新机制                                                     | 代表论文                                                         | 一句话                                   |
| ------------------------------------------------------------ | ---------------------------------------------------------------- | ---------------------------------------- |
| **环境侧自动 bookkeeping**（去重、压缩、重要性打标）   | **[[01-harness-1]]** ⭐                                    | 环境可靠维护可恢复状态，RL 不再学记账    |
| **周期性工作区重构**（Markovian state reconstruction） | IterResearch（`deep-research/02`）                             | 把长程重构为 MDP，周期性合成→重置工作区 |
| **delta-rule 在线更新**（不动 backbone 权重）          | **[[07-delta-mem]]** ⭐                                    | 用 delta 规则在线更新固定状态矩阵        |
| **patch 化更新历史**（记录环境演化）                   | **[[08-evoarena-evomem]]** ⭐                              | 以结构化 update history 记录环境变化     |
| **RL 学到的 memorization policy**（学"该记什么"）      | [[#taskmem]] 📄                                                  | 用 RL 让 agent 动态决定记什么            |
| **Sleep / Dreaming 离线巩固**（短期→长期参数）        | **[[10-externalization-efficiency]]** 关联 / [[#sleep]] ⭐ | 睡眠阶段把脆弱短期记忆蒸馏进稳定长期参数 |
| **Read–Write 反思式学习**（边做边写 skill 库）        | [[11-memskill-memento]] ⭐                                       | 读阶段选 skill、写阶段更新/扩充 skill 库 |
| **Retrospective 自监督优化**（用过去轨迹改 harness）   | [[#rho]] 📄                                                      | 无需标注，自我偏好挑选 harness 更新      |

### ③ 怎么**使用**（Usage）——记忆如何回到上下文供推理

| 使用方式                                                     | 代表论文                                 | 一句话                                          |
| ------------------------------------------------------------ | ---------------------------------------- | ----------------------------------------------- |
| **budget-aware context rendering**（按预算渲染上下文） | **[[01-harness-1]]** ⭐            | 在 token 预算内把 curated 状态渲染回上下文      |
| **mask 掉 stale 观测**（token-for-turn 权衡）          | **[[02-masking]]** ⭐              | 把模型已不再 attend 的旧观测移出上下文          |
| **task-aware 自适应剪枝**（按当前目标 skim）           | **[[05-swe-pruner]]** ⭐           | 给定任务目标，神经 skimmer 动态选相关行         |
| **运行时 deep-research 式按需检索**                    | GAM（`deep-research/11`）/ [[#mia]] 📄 | 把"记忆检索"当成一次深度研究                    |
| **单遍推理 ⇄ 迭代记忆处理融合**                       | **[[06-qwenlong-l15]]** ⭐         | >4M token 时在单遍与记忆 agent 间无缝切换       |
| **低秩修正注意力**（readout 改 attention）             | **[[07-delta-mem]]** ⭐            | 记忆 readout 对注意力做低秩修正                 |
| **把散落 turn 编译成显式长上下文监督**                 | **[[04-acc]]** ⭐                  | 把跨 turn 的证据编译成 long-context QA 训练信号 |
| **分阶段 rubric 召回**                                 | [[#rubricem]] 📄                         | rubric 同时充当执行/评判/记忆的统一接口         |

> **一句话总览**：2026 H1 的 working-memory 研究，**表示**上从"单一 transcript"转向"环境侧结构化状态 / 文件系统 / 固定状态矩阵"；**更新**上从"LLM 顺手记"转向"环境自动维护 / 周期重构 / 在线 delta / RL 学策略 / 离线巩固"；**使用**上从"全量塞回"转向"按预算渲染 / mask / 剪枝 / 按需检索 / 低秩注入"。三者合流指向同一句话——**不要让 policy 既当大脑又当记账员，把可恢复的状态外化出去、把上下文当成需要主动调度的资源。**

---

## 二、按类别列出（所有摘要均基于 arXiv 原文）

> 下列按**合理主题类别**组织（同一篇若跨类，归入最贴切的一类）；每条保留月份/票数与 arXiv 编号。⭐ = 精读，📄 = 简短描述。

### A. 记忆即运行时重构（Memory as Runtime Reconstruction）

> 不在离线把记忆压死，而是运行时按需检索、整合、重建上下文。

- **General Agentic Memory Via Deep Research** ⭐(已收录 `deep-research/11`) [BAAI, 171↑｜2025-11] — arXiv 2511.18423
  - 借编译器 **JIT** 原则：离线只留轻量记忆 + universal page-store（**无损保管**完整历史），运行时由 **Researcher** 像做 deep research 一样**按需检索整合**。
  - **三维定位**：表示=page-store+轻量记忆；更新=离线只打高亮（不压缩）；使用=运行时现场编译上下文。详见 `deep-research/11-general-agentic-memory.md`，本专题作为"记忆 = 运行时重构"主线的源头交叉引用。
- **IterResearch: Rethinking Long-Horizon Agents via Markovian State Reconstruction** ⭐(已收录 `deep-research/02`) [80↑｜2025-11] — arXiv 2511.07327
  - 诊断 **mono-contextual paradigm → context suffocation + noise contamination**；把长程重构为 **MDP + 周期性工作区重建**，维护**演进报告**作记忆。
  - **三维定位**：表示=evolving report；更新=周期性重构（Markovian）；使用=每轮把合成洞见重置进干净工作区。详见 `deep-research/02-iterresearch.md`。
- **Memory Intelligence Agent (MIA)** 📄 [786★｜2026-04] — arXiv 2604.04503
  - Manager-Planner-Executor 架构：非参数记忆存压缩历史轨迹 + 参数记忆 agent 产搜索计划 + **参/非参记忆双向转换循环**做记忆演化。"记忆 = 运行时检索 + 双向演化"代表（非精读，偏 DRA）。
- **Toward Generalist Autonomous Research via Hypothesis-Tree Refinement (Arbor)** 📄 [RUC-NLPIR, 100↑｜2026-06] — arXiv 2606.11926
  - long-lived coordinator + short-lived executors + **Hypothesis Tree Refinement**：一棵跨时间的**持久树**链接假设/产物/证据/洞见，把自治科研从"局部尝试序列"变成"累积过程"。working memory 的 **tree 形式 + 跨时间传播**（非精读，偏 auto-research）。

### B. 状态外化 & Harness 工程（State Externalization & Harness）

> 把可恢复的"记账"从 policy 搬到环境侧的 harness / 文件系统，policy 只留语义决策。

- **Harness-1: Reinforcement Learning for Search Agents with State-Externalizing Harnesses** ⭐ [Chroma, 52↑｜2026-06] — arXiv 2606.02373
  - **本专题核心论文**。论点：把搜索 agent 当成"在不断增长的 transcript 上的策略"，是把**太多例行状态管理塞进了 policy**。Harness-1（20B）在 **stateful harness** 里训练，**环境侧维护 working memory**——candidate pool、importance-tagged curated set、compact evidence links、verification records、压缩去重的 observations、**budget-aware context rendering**；policy 只保留语义决策（搜什么、留/弃哪些文档、验什么、何时停）。8 个检索基准平均 curated recall **0.730**（+11.4 超次优开源），**held-out transfer 增益尤强**。已成精读 [[01-harness-1]]。
- **FS-Researcher: Test-Time Scaling for Long-Horizon Research Tasks with File-System-Based Agents** ⭐ [52↑｜2026-02] — arXiv 2602.01566
  - **文件系统 = 持久外部记忆 + 跨 agent/session 协调介质**：Context Builder 当图书管理员写结构化笔记、归档原始源到可远超 context length 的分层知识库；Report Writer 据此逐节成文。working-memory **表示=文件系统**代表。已成精读 [[03-fs-researcher]]。
- **Retrospective Harness Optimization (RHO)** 📄 [MSR, 52↑｜2026-06] — arXiv 2606.05922
  - 仅用**过去轨迹**自监督优化 agent harness：挑难例 coreset 并行重解，靠 self-validation/self-consistency/自我偏好选 harness 更新。SWE-Bench Pro 59%→78%。working-memory 之外的"**harness 自我演化**"代表（非精读）。

### C. 上下文管理：压缩 / mask / 剪枝 / 切换（Context Management）

> 上下文是需要主动调度的有限资源——删、缩、切换、编译成监督。

- **Masking Stale Observations Helps Search Agents — Until It Doesn't: A Regime Map and Its Mechanism** ⭐ [McAuley-Lab, 62↑｜2026-06] — arXiv 2606.00408
  - 系统扫描 4B–284B backbone × 3 retriever：**mask stale 观测**的增益呈**非对称倒 U**——弱 retriever 下平台、强 retriever 遇中等模型时峰值、模型饱和时骤崩。机制是 **token-for-turn 权衡**（移走模型已不 attend 的旧页换更多 turn）。把 context management 重新定义为**regime-dependent**。working-memory **使用侧"何时该删"**精读。已成精读 [[02-masking]]。
- **SWE-Pruner: Self-Adaptive Context Pruning for Coding Agents** ⭐ [92↑｜2026-01] — arXiv 2601.16746
  - 模仿程序员"选择性 skim"：给定任务目标（如"focus on error handling"），用 **0.6B 神经 skimmer** 动态选相关代码行。agent 任务上 **23–54% token 缩减**、单轮任务最高 **14.84× 压缩**且性能几乎无损。working-memory **使用侧**（task-aware 剪枝）代表作。已成精读 [[05-swe-pruner]]。
- **QwenLong-L1.5: Post-Training Recipe for Long-Context Reasoning and Memory Management** ⭐ [Tongyi, 113↑｜2025-12] — arXiv 2512.12967
  - 三大件：长上下文数据合成 + 稳定化 RL（AEPO）+ **Memory-Augmented Architecture**——承认"再长的窗口也装不下任意长序列"，用**多阶段融合 RL** 把**单遍推理与迭代记忆处理**无缝结合，支撑 **>4M token**。已成精读 [[06-qwenlong-l15]]。
- **ACC: Compiling Agent Trajectories for Long-Context Training** ⭐ [USTC, 59↑｜2026-05] — arXiv 2605.21850
  - 洞见：agent 解题产生的**海量轨迹**里，答案证据**散落在多个 turn**，而标准 agent SFT **mask 掉 tool 响应**造成监督盲区。**ACC** 把 search/SWE/DB agent 的轨迹**编译成 long-context QA**（原问题 + 跨 turn 的 tool 响应/环境观测），显式训练**跨段长上下文推理**。MRCR +18.1、GraphWalks +7.6。working-memory **使用侧（把散落状态编译成监督）**精读。已成精读 [[04-acc]]。
- **Mindscape-Aware RAG (MiA-RAG)** 📄 [Tencent, 115↑｜2025-12] — arXiv 2512.17220
  - 给 RAG 注入**全局语义表示（mindscape）**做层次摘要，retrieval 与 generation 都条件于该全局视图。长上下文"全局表示"一例（非精读，偏 RAG）。

### D. 记忆架构：latent / KV-cache / 稀疏注意力（Memory Architectures）

> 把"记忆"做进模型内部——固定状态矩阵、KV-cache、稀疏注意力，与序列长度解耦。

- **δ-mem: Efficient Online Memory for Large Language Models** ⭐ [declare-lab, 128↑｜2026-05] — arXiv 2605.12357
  - 给**冻结**的 full-attention backbone 加一个**紧凑在线关联记忆**：用 **delta-rule** 把历史压进 **8×8 固定状态矩阵**，readout 对注意力做**低秩修正**。无需 full fine-tuning / 换 backbone / 扩上下文；MemoryAgentBench 上达 1.31×。working memory 的 **latent 形式 + 在线更新**精读。已成精读 [[07-delta-mem]]。
- **MSA: Memory Sparse Attention for Efficient End-to-End Memory Model Scaling to 100M Tokens** 📄 [3469★, EverMind｜2026-03] — arXiv 2603.23516
  - 端到端可训练的**百万亿级记忆模型框架**：scalable sparse attention + document-wise RoPE，16K→100M token 退化 <9%。working memory 的**架构级**极致扩展（非精读，偏底层架构）。
- **HERMES: KV Cache as Hierarchical Memory for Efficient Streaming Video Understanding** 📄 [OpenMOSS, 75↑｜2026-01] — arXiv 2601.14724
  - training-free 地把 **KV-cache 概念化为分层记忆**，复用紧凑 KV-cache 做流式视频理解（TTFT 快 10×）。working memory 的 **KV-cache 形式**代表（非精读，偏视频）。
- **Latent Collaboration in Multi-Agent Systems（LatentMAS）** 📄 [127↑｜2025-11] — arXiv 2511.20639
  - 多 agent 在**潜空间**协作，提出 **latent working memory**（潜工作记忆）做信息交换。working memory 的 **latent 形式**一例（非精读，偏 MAS）。

### E. 可学习 / 可演化的记忆（Learnable & Evolving Memory）

> "记什么、怎么记、怎么巩固"不再硬编码，而是可学习、可演化、可离线巩固。

- **MemSkill: Learning and Evolving Memory Skills for Self-Evolving Agents** ⭐ [NTU, 63↑｜2026-02] — arXiv 2602.02474
  - 把"提取/巩固/剪枝记忆"的固定操作**重构为可学习、可演化的 memory skills**；controller 选 skill、executor 产记忆、designer 复盘难例并演化 skill 集。working-memory **更新策略可学化**代表。已成精读 [[11-memskill-memento]]（与 Memento-Skills 合并）。
- **Memento-Skills: Let Agents Design Agents** ⭐ [UCL, 58↑｜2026-03] — arXiv 2603.18743
  - **memory-based RL + stateful prompts**：可复用 skill（结构化 markdown）充当**持久、演化的记忆**；**Read–Write 反思式学习**——读阶段 skill router 选 skill、写阶段更新/扩充 skill 库；**不更新 LLM 参数**，全靠外化 skill/prompt 演化。已成精读 [[11-memskill-memento]]（与 MemSkill 合并）。
- **EvoArena: Tracking Memory Evolution for Robust LLM Agents in Dynamic Environments** ⭐ [MIT, 90↑｜2026-06] — arXiv 2606.13681
  - 现有评测多假设**静态环境**；EvoArena 把环境变化建模为 terminal/software/social 上的**渐进 update 序列**。**EvoMem**——**patch-based 记忆**，以结构化 update history 记录记忆演化。当前 agent 仅 39.6%；EvoMem 在 GAIA/LoCoMo 上 +6.1%/+4.8%，chain-level +3.7%。working-memory **更新（记录演化）+ 评测**精读。已成精读 [[08-evoarena-evomem]]。
- **Language Models Need Sleep: Learning to Self-Modify and Consolidate Memories** ⭐ [Google, 29↑｜2026-06] — arXiv 2606.03979
  - 提出 **Sleep 范式**：把脆弱**短期记忆**经 replay **蒸馏进稳定长期参数**（Knowledge Seeding：小自我→大网络的向上蒸馏），并用 **Dreaming**（RL 自生成课程）自我精进。working-memory 与 **long-term 参数化记忆的桥接（更新侧·离线巩固）**精读。已成精读见 [[10-externalization-efficiency]] 关联讨论与本条。
- **Task-Focused Memorization for Multimodal Agents (TaskMem)** 📄 [ByteDance-Seed, 38↑｜2026-05] — arXiv 2605.31075
  - 把记忆生成**重构为可学习的 memorization policy**，两阶段 RL 学"怎么记"与"该记什么"。working-memory **更新策略可学化**（多模态侧，非精读）。
- **RubricEM: Meta-RL with Rubric-guided Policy Decomposition** 📄 [Google, 79↑｜2026-05] — arXiv 2605.10899
  - 主张 **rubric 应同时充当执行接口 / 评判反馈 / agent memory** 三位一体；分阶段 GRPO + 反思 meta-policy 把判过的轨迹蒸成可复用指导。"rubric 即记忆接口"一例（非精读，偏 RL）。

### F. 综述 & 统一框架（Surveys & Frameworks）

- **Memory in the Age of AI Agents** ⭐ [157↑｜2025-12] — arXiv 2512.13564
  - **agent memory 最新综述**：明确把 agent memory 与 LLM memory / RAG / context engineering 区分；用 **forms（token-level / parametric / latent）× functions（factual / experiential / working）× dynamics（formed / evolved / retrieved）** 三透镜统一全领域。**本专题的概念锚点与阅读地图**。已成精读 [[09-memory-survey]]。
- **Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering** ⭐ [SJTU, 52↑｜2026-04] — arXiv 2604.08224
  - 用 **externalization（外化）** 统一透镜回顾 agent 基础设施：**memory 把状态跨时间外化**、skills 把过程外化、protocols 把交互外化、harness 是协调它们的统一层。提出"从 weights → context → harness"的历史演进。**本专题的理论框架**。已成精读 [[10-externalization-efficiency]]（与 Toward Efficient Agents 合并）。
- **Toward Efficient Agents: Memory, Tool learning, and Planning** ⭐(合并) [Shanghai AI Lab, 57↑｜2026-01] — arXiv 2601.14192
  - 从 memory/tool/planning 三组件审视 agent **效率**；记忆侧的共识：**bounding context via compression and management（用压缩与管理界定上下文）**。与 [[10-externalization-efficiency]] 合并精读（效率/外化双视角）。

### G. 评测 & 训练数据（Benchmarks & Data）

- **HaluMem: Evaluating Hallucinations in Memory Systems of Agents** 📄 [95↑｜2025-11] — arXiv 2511.03506
  - 提出记忆系统**幻觉**评测：fabrication / error / conflict / omission 四类。聚焦"记忆质量"评测维度，本专题作为记忆**可靠性**评测列出（非精读）。
- **LMEB: Long-horizon Memory Embedding Benchmark** 📄 [73↑｜2026-03] — arXiv 2603.12572
  - 长程记忆**检索/embedding 评测**基准，区分 episodic 等记忆类型。本专题作为记忆**使用侧（检索）评测**列出（非精读）。
- **daVinci-Agency: Unlocking Long-Horizon Agency Data-Efficiently** 📄 [GAIR, 53↑｜2026-02] — arXiv 2602.02619
  - 用 **Pull Request 序列**天然的长依赖结构合成长程监督数据。偏"长程 agent 训练数据合成"，与 working memory 的"跨阶段一致性"相关（非精读）。

---

## 三、精读论文清单（11 篇 / 9 个文件）

| 文件                                        | 论文                                      | 月/票               | 三维主贡献                   | 一句话                                     |
| ------------------------------------------- | ----------------------------------------- | ------------------- | ---------------------------- | ------------------------------------------ |
| **[[01-harness-1]]**                  | Harness-1                                 | 06 / 52↑           | 表示·更新·使用（全）       | 环境侧 working memory，policy 只留语义决策 |
| **[[02-masking]]**                    | Masking Stale Observations                | 06 / 62↑           | **使用**（何时删）     | mask 增益呈 regime-dependent 倒 U          |
| **[[03-fs-researcher]]**              | FS-Researcher                             | 02 / 52↑           | **表示**（文件系统）   | 文件系统当持久外部记忆 + 协调介质          |
| **[[04-acc]]**                        | ACC                                       | 05 / 59↑           | **使用**（编译成监督） | 把散落 turn 编译成 long-context 训练信号   |
| **[[05-swe-pruner]]**                 | SWE-Pruner                                | 01 / 92↑           | **使用**（剪枝）       | task-aware 神经 skimmer 自适应剪枝         |
| **[[06-qwenlong-l15]]**               | QwenLong-L1.5                             | 12 / 113↑          | 表示·使用                   | 单遍 ⇄ 迭代记忆融合，撑 >4M token         |
| **[[07-delta-mem]]**                  | δ-mem                                    | 05 / 128↑          | 表示·更新·使用             | 8×8 在线状态矩阵 + delta 更新 + 低秩注入  |
| **[[08-evoarena-evomem]]**            | EvoArena / EvoMem                         | 06 / 90↑           | **更新**（记录演化）   | patch-based 记忆记录环境演化 + 评测        |
| **[[09-memory-survey]]**              | Memory in the Age of AI Agents            | 12 / 157↑          | 三维框架本身                 | forms×functions×dynamics 统一透镜        |
| **[[10-externalization-efficiency]]** | Externalization Review + Efficient Agents | 04·01 / 52↑·57↑ | 表示·更新（理论）           | externalization 框架 + 效率 Pareto         |
| **[[11-memskill-memento]]**           | MemSkill + Memento-Skills                 | 02·03 / 63↑·58↑ | **更新**（策略可学）   | 记忆操作 = 可学习/可演化的 skill           |

> **去重交叉引用**（已在 `deep-research/` 精读）：
>
> - **GAM**（`deep-research/11`）——JIT 记忆，运行时按需检索整合。
> - **IterResearch**（`deep-research/02`）——Markovian 工作区重构，演进报告作记忆。

---

## 四、2025-11 → 2026-06 关键趋势速览

> 围绕 **表示 / 更新 / 使用** 三维度提炼。

1. **【表示】working memory 正在"离开 transcript"**：从单一膨胀上下文，转向**环境侧结构化状态**（Harness-1 的 candidate pool/evidence links）、**文件系统**（FS-Researcher）、**演进报告**（IterResearch）、**持久树**（Arbor）、**固定状态矩阵**（δ-mem）。共识——**可恢复的状态不该塞在 policy 里**。
2. **【更新】"谁来记、何时记"成为设计核心**：环境自动 bookkeeping（Harness-1）、周期性重构（IterResearch）、在线 delta-rule（δ-mem）、patch 化演化历史（EvoMem）、**RL 学到的 memorization policy**（TaskMem/MemSkill）、**离线 Sleep 巩固**（LMs Need Sleep）。趋势——**把"记什么/怎么记"从硬编码人类先验，变成可学习、可演化、可离线巩固的过程**。
3. **【使用】上下文从"全量塞回"变成"主动调度的资源"**：budget-aware rendering（Harness-1）、mask stale 观测（Masking，但**有 regime 边界**）、task-aware 剪枝（SWE-Pruner）、运行时 deep-research 式检索（GAM/MIA）、低秩注入注意力（δ-mem）、把散落 turn 编译成监督（ACC）。关键发现——**没有银弹**：Masking 证明 context 删减的收益强烈依赖 retriever 召回 × 模型隐式过滤容量的交互。
4. **【RL 视角】把记账从 policy 剥离，RL 才学得动**：Harness-1 与多篇共同主张——强化学习不该既学语义决策又学可恢复记账；**state externalization 让 held-out transfer 增益尤强**。
5. **【评测视角】从"静态环境"走向"演化环境"**：EvoArena 指出真实部署是动态的，记忆必须**记录环境演化**；HaluMem 关注记忆**幻觉**（fabrication/conflict/omission）；LMEB 评长程记忆检索——"最终任务对"已不足以衡量记忆系统。
6. **【统一框架成型】externalization 与 forms×functions×dynamics**：[[09-memory-survey]] 与 [[10-externalization-efficiency]] 给出两套互补的统一语言——**功能上区分 factual/experiential/working，形式上区分 token/parametric/latent，动态上区分 formed/evolved/retrieved**；working memory 是其中"形式偏 token、动态最活跃"的一支。

**仍待解决**：① working memory 的**最优粒度**（行级？观测级？语义块级？）缺乏统一结论；② **更新的稳定性**——在线 delta / patch / RL policy 在超长历史下是否漂移；③ **使用的 regime 依赖**（Masking）尚无自适应开关；④ working memory 与 long-term 参数记忆的**巩固边界**（Sleep）仍是 proof-of-concept；⑤ **评测**仍多用 LLM-judge，记忆幻觉的客观度量未统一。

---

## 五、数据来源说明

- 论文元数据来自 **HuggingFace Papers 月榜**（`huggingface.co/papers/month/YYYY-MM`），2025-11 ~ 2026-06 共 8 个月、每月 top50。
- 笔记内容基于 **arXiv 摘要**（部分含 GitHub README / 全文）撰写；模型/数据规模数字以论文摘要为准，摘要未披露处统一标注"**需读 PDF**"。
- 与 `deep-research/`、`evolve/`、`huggingface/` 三个既有专题**去重**：重叠论文仅交叉引用、不重复精读。
