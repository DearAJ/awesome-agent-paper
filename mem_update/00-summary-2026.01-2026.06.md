# Agent 记忆更新（非增量 / 冲突 / 矛盾）专题 — 2026.01–2026.06

> **专题焦点**：**记忆更新**，尤其是**非增量更新**——当新信息**与已存记忆冲突 / 矛盾 / 使其过时（stale）****时，agent / LLM 该怎么办：覆盖？门控保留？版本化？还是保留为并行分支？**
> **数据范围**：HuggingFace Papers 月榜 + arXiv，2026-01 ~ 2026-06（6 个月）；补充必引的经典 prior art（ROME/MEMIT、知识冲突综述、AGM/TMS 等）。
> **标记**：⭐ = 精读｜📄 = 摘要描述｜机制标签：**OVERWRITE / GATE / VERSION / BRANCH**

---

## 〇、「记忆更新」核心难点

`memory/` 专题讲的是 **working memory 怎么表示/更新/使用**（上下文增长怎么管）。本专题把其中「**更新**」一维**单独放大**，并专门聚焦最难的一类——**非增量（矛盾）更新**。

**为什么这是独立且关键的问题**：

- **加性更新**（新事实与旧记忆不矛盾）= 简单追加，几乎人人会做。
- **非增量更新**（新事实**推翻**旧记忆）= 必须决定**对旧记忆做什么**：删？改？降权？保留备查？而且 **依赖旧记忆的下游结论也得跟着变**。这才是真问题。

**全领域的矛盾处理只塌缩成 4 种机制**（本专题的分析骨架，贯穿每篇）：

| 机制                            | 一句话                                                           | 代表                                                               | 现状                                        |
| ------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------- |
| **OVERWRITE（覆盖）**     | 原地改写旧事实（含参数级权重编辑）                               | ROME/MEMIT、MOSE、LM Need Sleep                                    | 🔴 最成熟最卷，**但"覆盖有害"被实证** |
| **GATE（门控）**          | 先判该不该更新；倾向**保留原始证据**、不轻易覆盖           | STALE、CBM、Useful-Memories-Faulty、EvolveMem                      | 🟢**2026 H1 的明显转向**              |
| **VERSION（版本/patch）** | 记**变更历史 + 时间戳**，旧值降权而非删除                  | EvoMem(patch)、MemForest(树)、RoMem(相位降级)、DCPM(supersedes 链) | 🟡 中等，已有卡位                           |
| **BRANCH（分支）**        | 矛盾证据**并行保留为活分支**，靠后续证据**收敛裁决** | TOKI(audit 行保留)、ATMS(经典)                                     | ⚪**几乎空**                          |

**三条贯穿 6 个月的核心结论**：

1. **覆盖（overwrite）正在被证伪**。最强实证 **"Useful Memories Become Faulty…"（2605.12978）**：LLM 持续 consolidation 覆盖记忆**反而有害**——效用掉到**无记忆基线以下**，保留原始 episode 反而**翻倍**。→ 全领域从 OVERWRITE 转向 **GATE**。
2. **级联失效（cascade / dependent-invalidation）是公认未解难题**。更新 X 后，**依赖 X 的下游记忆没跟着失效**。这个洞**参数侧与记忆侧同时栽**：参数侧 **RippleEdits / MQuAKE** 证明编辑"recall 得到但传播不到"（"hopping-too-late"）；记忆侧 **MEME** 把它测到谷底（**Cascade ~3%、Absence ~1%**）。**谁解决谁有论文。**
3. **BRANCH 近乎空地**。把"矛盾证据保留为**并行活分支**、靠收敛裁决"——几乎没人做。最接近的是 **TOKI**（把落败事实留在 audit 行）、**DCPM**（supersedes 链）和经典 **ATMS**（多 environment 并行），但**都不是"并行活检索分支 + 收敛"**。**这正是 ReTrace 的位置。**

> 经典理论坐标（related-work 的外衣）：**AGM belief revision**（expansion/revision/contraction + 最小改动）、**TMS/JTMS**（Doyle 1979：justification 依赖网络 + **矛盾时 dependency-directed backtracking** —— 这就是"回溯到矛盾源、失效依赖链"的经典原型）、**ATMS**（de Kleer 1986：多 assumption-set 并行 = branch/version）。LLM 时代复活：**BeliefBank**（MaxSAT 求解器修订冲突信念）。

---

## 一、经典 / 必引 prior art（主流方法的骨架，pre-2026）

> 写作与引用时的"地基"。本专题精读 06/07/08 会展开，这里先给全景。

### 知识编辑（参数级 OVERWRITE，最主流也最卷）

| 方法                | arXiv/venue               | 怎么编辑                                                          | 机制                         | 已知失败模式                                                 |
| ------------------- | ------------------------- | ----------------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| **ROME**      | 2202.05262 (NeurIPS'22)   | causal trace 定位决定性中层 MLP，rank-1 更新插入新 (主语→新宾语) | OVERWRITE 单条               | 一次一条；**涟漪失败**（蕴含事实不变）；改写不传播多跳 |
| **MEMIT**     | 2210.07229 (ICLR'23)      | 把 ROME 扩到**上千条**，最小二乘跨多层铺开                  | OVERWRITE 批量               | 编辑量↑退化快；渗漏邻居；不传播多跳                         |
| **MEND**      | 2110.11309 (ICLR'22)      | 超网络把单样本梯度变换成局部权重增量                              | OVERWRITE 习得               | 多序列编辑遗忘/破坏 locality                                 |
| **SERAC**     | 2206.06520 (ICML'22)      | scope 分类器把 query 路由到"反事实模型"或冻结 base                | GATE+KEEP（外存，base 冻结） | 受 scope 分类器精度上限制约；编辑不可组合                    |
| **GRACE**     | 2211.11031 (NeurIPS'23)   | codebook 包一层，query 落入 deferral 半径才触发习得值             | GATE+KEEP（离散码本）        | 纯检索式覆盖（不推理）；码本随编辑膨胀                       |
| **WISE**      | 2405.14768 (NeurIPS'24)   | 旁路 side-memory 存编辑 + router 选主/侧 + 知识分片               | **BRANCH+GATE**        | 路由错误；分片数有上限                                       |
| **AlphaEdit** | 2410.02355 (ICLR'25 Oral) | 把扰动投影到"保留知识键的零空间"，证明不动已保留输出              | OVERWRITE（受约束）          | 保护旧知识但仍继承 ROME 涟漪缺口；零空间抽样估计有渗漏       |

**编辑评测的"级联失效"锚点**（务必与上面合引）：

- **RippleEdits**（2307.12976, TACL'24）：5K 编辑，测**组合/2-hop/逆关系**传播；发现方法"recall 编辑但不更新蕴含事实"——**级联失效的标准基准**。
- **MQuAKE**（2305.14795, EMNLP'23）：答案**必须随蕴含后果改变**的多跳 QA；编辑器"多跳上灾难性失败"；提出 **MeLLo**（外存编辑 + 迭代提示 = keep/branch 的替代）。

### 知识冲突 / 时序 / 评测（框架与基准）

- **知识冲突综述**（2403.08319, 2024）：分类 = **context-memory / inter-context / intra-memory 冲突**。本专题 C 类的框架锚点。
- **ConflictQA**（出自 *Adaptive Chameleon or Stubborn Sloth* 2305.13300, ICLR'24）：诱出参数记忆→合成反记忆；发现 LLM 对连贯外证**易接受**、部分重叠时**确认偏误**。
- **温度/时序基准**：TempLAMA(2106.15110)、**TemporalWiki**(2204.14211, 快照 diff，retain-old vs acquire-new，**staleness 最佳经典拟合**)、Test-of-Time(2406.09170, 合成时序逻辑避污染)。
- **记忆基准——真测 update 吗**：**LoCoMo**(2402.17753) **不测** update（5 类=单跳/多跳/时序/开放域/对抗）；**LongMemEval**(2410.10813) **测**（"knowledge updates"是 5 能力之一，**最干净的 update 测试器**）；**MemoryAgentBench**(2507.05257) **部分**（test-time learning + selective forgetting，但无 named "conflict resolution"轴）。

---

## 二、2026-01 ~ 2026-06 命中论文（按机制 + 主题分类）

> 每条含 arXiv id / ↑ / 机构 / **机制标签** / 摘要。与 `memory/`、`evolve/` 重叠者标注交叉引用。

### A. 矛盾 / 失效检测与 belief revision（agent 侧，最核心）

- **STALE: Can LLM Agents Know When Their Memories Are No Longer Valid?** ⭐ **GATE** [HKUST NLP, 46↑｜2605.06527]
  - **本主题最正中靶心**。定义 **"Implicit Conflict"**——后续观测在**无显式否定**下使旧记忆失效（如"用户搬家了"使旧地址失效）。400 冲突场景 / 1200 query / 上下文至 150K，探测 State Resolution、Premise Resistance、Implicit Policy Adaptation。**best model 仅 55.2%**。提出 **CUPMem**：**写入时**做 structured state consolidation + **propagation-aware search**（一处变了，相关记忆一起重审）。已成精读 [[02-stale]]。
- **When Should Models Change Their Minds? Contextual Belief Management** 📄 **GATE** [浙大 zjunlp, 26↑｜2605.30219]
  - 直接做 **belief revision**：把"何时 **update / preserve / ignore** 信念"做成决策。**BeliefTrack** 基准，失败三类 = Failed Stay / Failed Update / Failed Isolation。**RL + belief-state 奖励把失败砍 70.9%**。是 ReTrace"加性 vs 非加性判定"的最佳借鉴（三分类）。精读 [[07-belief-revision]] 合并讨论。
- **Useful Memories Become Faulty When Continuously Updated by LLMs** ⭐ **GATE>OVERWRITE** [UIUC (Hao Peng), 19↑｜2605.12978]
  - **覆盖有害的最强实证**。LLM 持续 consolidation 覆盖记忆→效用**低于无记忆基线**；GPT-5.4 把已解决的 ARC-AGI **重做错 54%**；**保留原始 episode 比强制 consolidation 准确率翻倍**。结论：**门控 consolidation、别覆盖原始证据**。已成精读 [[03-useful-memories-faulty]]。
- **Struct-Searcher: Agentic Structural Thinking for Multimodal Deep Information Seeking** 📄 **VERSION/BRANCH** [5↑｜2606.07689]
  - 🔴 **与 ReTrace 极近**：显式建立在 **belief-revision 理论**上，维护**演进的结构图**，在搜索中**裁决跨模态矛盾证据**而非线性堆叠。是"search agent 做 belief revision"的同期工作（多模态侧）。精读 [[07-belief-revision]] 提及对比。
- **Bayesian-Agent: Posterior-Guided Skill Evolution for LLM Agent Harnesses** 📄 **GATE+patch** [14↑｜2606.08348]
  - 把记忆条目当**假设 + 后验**，按验证轨迹证据更新后验，映射到可检视动作 **patch / split / compress / retire / explore**。Bayesian belief revision over memory——与 ReTrace"按证据裁决分支"思路同源。
- **ClawArena: Benchmarking AI Agents in Evolving Information Environments** 📄 **benchmark** [UNC, 37↑｜2604.04202]
  - 基准：agent 须在信息环境演化中**维持正确信念**——证据散落于矛盾源、**新信息使旧结论失效**、用户偏好以**纠正**形式到来。三轴：多源冲突推理 / 动态 belief revision / 隐式个性化。（HF 缓存与 live arXiv 版本数字有别，引 live PDF。）
- **OAKS: Benchmarking Online Adaptation to Continual Knowledge Streams** 📄 **benchmark** [KAIST AI, 18↑｜2603.07392]
  - 基准：**单个事实在流式上下文里多次改变**，密集标注测模型是否**覆盖working belief**还是抱旧值。SOTA 模型与 agentic-memory 系统**都更新不稳**。
- **MMA: Multimodal Memory Agent (+MMA-Bench)** 📄 **GATE** [北大, 9↑｜2602.16493]
  - 检索面会surface**过时/低可信/冲突**条目；按 **来源可信度 × 时间衰减 × 冲突感知网络共识** 重加权、不支撑则**弃答（abstain）**。配 MMA-Bench 测 belief dynamics（文本-视觉结构化矛盾）。

### B. 结构化记忆的更新机制（树 / 图 / 版本 —— 与 ReTrace 最像）

- **MemForest: Agent Memory with Hierarchical Temporal Indexing** ⭐ **VERSION/树** [17↑｜2605.23986]
  - 🔴 **结构最像 ReTrace 的 prior art**。**MemTree（时间有序树）**替代 flat 全局 summary；**局部 per-node 更新替代全量重写**，只改"受影响树路径"，自然保留时序演化态。LongMemEval-S **79.8%**、构建吞吐 **~6×**。**但只测时序召回、无矛盾分支语义**——ReTrace 必引必 diff。已成精读 [[01-memforest]]。
- **EvoArena / EvoMem: Tracking Memory Evolution in Dynamic Environments** ⭐ **VERSION/patch** [Salesforce/NUS/MIT, 90↑｜2606.13681]
  - **你的引子**。把环境变化建模为**渐进 update 序列**（terminal/software/social），**EvoMem 用 patch 化 update history** 记"状态怎么变的"。当前 agent 仅 39.6%；**EvoMem 仅 +1.5%**（LoCoMo +4.8%、GAIA +6.1%）。**记演化好、但不解矛盾、不失效依赖项、无冲突检测**。已成精读 [[05-evomem]]（与 `evolve/18`、`memory/08` 交叉，本专题视角=patch 作为 update 机制）。
- **RoMem: Continuous Phase Rotation for Temporal KG & Agentic Memory** ⭐ **VERSION/keep-but-demote** [爱丁堡, 4↑｜2604.11544]
  - 🔴 **"不删旧值"的优雅实现**。**Semantic Speed Gate** 给每个关系打**易变度**（"president of"高、"born in"低）；复向量**连续相位旋转**做"几何遮蔽"——把过时事实**转出相位**让时序正确者排名更高，**不删除旧的**（keep-but-demote / 相位版本化）。显式对比"简单覆盖过时事实"的系统。静态记忆零退化、时序推理 2-3×。精读 [[01-memforest]] 内对比讨论。
- **Memory Beyond Recall (DCPM)** 📄 **VERSION+consolidate** [arXiv-only, ｜2606.09483]
  - System1"白天写手"把 belief revision 记成**双向 supersedes 链**；System2"夜间引擎"异步**裁决跨域碰撞**（version + consolidate）。——supersedes 链是 BRANCH 的近邻。
- **TOKI: Bitemporal Memory for Agents** 📄 **BRANCH/version** [arXiv-only, ｜2606.06240]
  - 🔴 **最接近 BRANCH**：**bitemporal 算子代数**把矛盾当**写入时并发控制**处理，**每次写版本化**，并**把落败事实留在 audit 行**（version + retain）。这正是 ReTrace"失活但保留"的另一实现——但仍非"并行活检索分支 + 收敛"。
- **Memanto: Typed Semantic Memory with Information-Theoretic Retrieval** 📄 **VERSION** [Moorcheh, 13↑｜2604.22085]
  - 核心是**自动冲突裁决 + 时序版本化**，覆盖 13 类型化记忆；新记忆与旧冲突时**裁决并打版本戳**而非盲目追加。LongMemEval 89.8% / LoCoMo 87.1%（单query 检索），消融隔离了冲突裁决阶段。
- **MemMA: Coordinating the Memory Cycle via Multi-Agent Reasoning & In-Situ Self-Evolution** 📄 **GATE/REPAIR** [10↑｜2603.18718]
  - backward-path 闭环：记忆定稿前**合成探针 QA → 验证当前记忆 → 把失败转成 REPAIR/覆盖动作**。是**验证失败触发的主动编辑**（gate-on-verification-failure），非纯追加。
- **FluxMem: Memory as Continuously Evolving Connectivity** 📄 **OVERWRITE-on-graph** [AGI Lab/ZJU, 34↑｜2605.28773]
  - 记忆=持续演化的连接图：修复缺失链接、**剪枝干扰**、长期巩固。
- **HAGE: Agentic Memory via RL-Driven Weighted Graph Evolution** 📄 **GATE/图** [16↑｜2605.09942]
  - 边的置信/强度随 RL 演化，抑制噪声/弱连接。
- **InKH: Interaction-Native Knowledge Harness for Financial Agents** 📄 **GATE+decay** [5↑｜2606.01886]
  - 时序图记忆 + **写入时失效** + 成熟度/衰减；后台抽取**覆盖/退休过时条目**，把 stale-knowledge 使用率砍 **~96.6%**。

### C. 知识冲突（context vs memory / retrieved vs parametric）

- **知识冲突综述（Knowledge Conflicts for LLMs: A Survey）** 📄 [2403.08319, 2024] — 三类冲突分类法，C 类框架锚点。精读 [[08-knowledge-conflict]] 展开。
- **TRACK: Tracking the Limits of Knowledge Propagation** 📄 **诊断** [EPFL (Bosselut), ｜2601.15495]
  - **冲突传播进推理**的最强证据：当 in-context/编辑事实**未能覆盖参数知识**，残留冲突**传播进多步推理**（WIKI/CODE/MATH）；给更新事实**反而比不给更差**，更新越多越糟。精读 [[08-knowledge-conflict]] 引用。
- **TCR: Transparent Conflict Resolution in RAG** 📄 **GATE（三标量）** [2601.06842]
  - 按 **语义匹配 + 事实一致性（双对比编码器） + self-answerability（对内部记忆的置信）** 三标量逐实例裁决"信检索还是信参数记忆"；误导上下文覆盖 **−29.3pp**。
- **Tool-Memory Conflicts (TMC)** 📄 **诊断** [UMass Lowell, ｜2601.09760]
  - 新冲突类型：**参数知识 vs 工具输出**；现有 prompt/RAG 裁决技术**都无法可靠让工具胜出**（负面结果，agent 相关）。
- **Intra-Memory Conflict（Where Knowledge Collides）** 📄 **steering** [2601.09445] — 预训练就烘进参数的冲突；机制可解释定位冲突组件 + 推理时**因果干预**控制哪方胜。
- **CC-VQA: Conflict- & Correlation-Aware KB-VQA** 📄 **GATE** [CQU, CVPR'26, ｜2602.23952] — 多模态 context-vs-parametric 冲突，相关度加权冲突评分在 decode 时门控谁胜。
- **Whose Facts Win? Source Preferences under Conflict** 📄 [Saarland, ｜2601.03746] — inter-context 冲突按**来源可信度**裁决，**但重复会翻转胜者**；提法降重复偏误 79.2%。

### D. 知识编辑 / 序列编辑稳定性（参数侧 OVERWRITE，主流但很卷）

> 经典 7 法见第一节。2026 新增聚焦"**序列/终身编辑怎么不崩**"：

- **MOSE: Multiplicative Orthogonal Sequential Editing** ⭐ **OVERWRITE（正交乘法）** [USTC, ｜2601.07873]
  - 放弃**加性 ΔW**（破坏数值稳定）；改用**正交矩阵乘法**携带新事实——保条件数/范数，反复覆盖仍稳。精读 [[06-rome-rippleedits]] 内对比。
- **REVIVE: Sequential Knowledge Editing Collapse** 📄 [2601.11042] — 序列编辑崩溃因**扰动主奇异方向**；在原权重谱基底写更新 + 过滤撞保护子空间的分量，测到 **2 万次编辑**。
- **NAS: Norm Anchors Make Model Edits Last** 📄 [2602.02543] — 发现**正反馈范数爆炸**致崩溃；把每个解出的 value 向量**重缩放回原模型范数**，编辑跨度 **>4×**。
- **CoRSA: Conflict-Resolving + Sharpness-Aware Editing** 📄 [UNC, ｜2602.03696] — **最大化新旧知识边际**解冲突 + 低曲率更新，遗忘 **−27.82%**。
- **ROME on Multi-hop（限制）** 📄 [2601.04600] — ROME 编辑**不传播到依赖的多跳事实**（"hopping-too-late"）；提 Redundant Editing 让覆盖**级联through推理链**（2-hop +15.5pp）。**级联失效的参数侧核心证据。**
- **EVK / EVK-Bench: Beyond Local Edits** 📄 [2602.01977] — 量化并约束一次编辑的**知识漂移（连带级联）**。
- **LOKI: Lifelong Editing via Null-Space Projection** 📄 [arXiv-only, ｜2606.19679] — 梯度更新**零空间投影**做终身编辑（不覆盖旧知识）+ HSIC 动态选层。

### E. 时序 / 过时知识 + 巩固覆盖

- **Language Models Need Sleep: Self-Modify and Consolidate Memories** ⭐ **OVERWRITE（巩固进权重）** [Google (Titans/Atlas 组), 29↑｜2606.03979]
  - **Sleep** 阶段把"短期脆弱记忆蒸馏进稳定长期权重"（**Knowledge Seeding** 小→大网络向上蒸馏）+ **Dreaming** RL 排演合成数据。**巩固即更新原语，但无冲突裁决**——覆盖默认"新>旧"，有把错的/矛盾的事实巩固进去的风险。强 provenance。已成精读 [[03-useful-memories-faulty]] 内对比（与"覆盖有害"正面对照）。
- **FadeMem: Bio-Inspired Forgetting** 📄 **GATE+融合** [2601.18642] — **自适应指数衰减** + **LLM 引导的冲突裁决与记忆融合**：相关项巩固、过时项淡出。
- **Masking Stale Observations Helps Search Agents — Until It Doesn't** 📄 **GATE（丢弃）** [UCSD, 62↑｜2606.00408] — mask 过时观测增益呈**非对称倒 U**；机制=token-for-turn。**注意：这里"stale"=轨迹里旧，不是被新事实矛盾**。已在 `memory/02` 精读，本专题交叉引用（对比"丢弃 stale" vs "裁决 contradiction"）。
- **温度基准**：TemporalWiki(2204.14211)、MQuAKE(2305.14795)、Test-of-Time(2406.09170)——见第一节。

### F. 记忆更新评测基准（区分"真更新"vs"仅检索"）

- **MEME: Multi-entity & Evolving Memory Evaluation** ⭐ **benchmark** [KAIST AI, 7↑｜2605.12477]
  - 演化记忆基准，含 **Deletion（删除后状态）+ Cascade / Absence（依赖推理）**——即更新应**传播/失效依赖项**。**所有系统在 Cascade ~3%、Absence ~1% 崩溃**。**级联失效的最佳度量基准**——ReTrace 若在此提分=杀手锏。已成精读 [[04-meme]]。
- **MINTEval: Memory under Multi-Target Interference** 📄 **benchmark** [UNC (Bansal), 5↑｜2605.18565] — facts 被后续 context **修订/干扰**（Wikipedia revisions、GitHub commits）；干扰数↑准确率↓（均 27.9%）。ReTrace"矛盾注入实验"的现成模板。
- **Your Agents Are Aging Too (AgingBench)** 📄 **benchmark** [32↑｜2605.26302] — 提出 **"revision aging"**：记忆随反复修订退化；诊断 revision/compression/interference/maintenance 四类老化。
- **SubtleMemory: Fine-Grained Relational Memory Discrimination** 📄 **benchmark** [19↑｜2606.05761] — relation-controlled 记忆变体（互补 / 微妙 / **直接矛盾**），测 agent 是否**检测并正确使用冲突记忆**。
- **MemTrace: Tracing & Attributing Errors in LLM Memory** 📄 **诊断** [AGI Lab/ZJU, 40↑｜2605.28732] — 记忆演化图，失败模式含信息**腐化（corruption）**。ReTrace 评测"错误证据是否被纠正"的工具。
- **EvolveMem: Self-Evolving Memory via AutoResearch** 📄 **GATE（回滚）** [24↑｜2605.13941] — 演化记忆，**回归就回滚（revert-on-regression）**。
- **LongMemEval**（2410.10813）/ **MemoryAgentBench**（2507.05257）—— 见第一节（真测/部分测 update）。

> **更多相关但偏外**（自演化/额外角度，简列）：MemSkill(2602.02474, 记忆操作 skill 化，已在 `evolve/`+`memory/11`)、Memento-Skills(2603.18743, 同)、Online Experiential Learning(2603.16856, 加性巩固)、LedgerAgent(2606.20529, ledger 取代 stale 状态)、SuperLocalMemory V3(2603.14588, sheaf 矛盾检测)、ATM-Bench(2603.01990, 含"处理冲突证据"子技能)、OCC-RAG(2606.00683, context-faithful，弱拟合)。

---

## 三、趋势与未解难题

### 趋势（2026-01 → 06）

1. **OVERWRITE → GATE 的范式转向**：从"直接覆盖/巩固"转向"先判该不该更新、倾向保留原始证据"。最强信号是 **2605.12978 实证覆盖有害**；STALE / CBM / EvolveMem / FadeMem 全独立走向门控。
2. **记忆侧 vs 参数侧的合流**：知识编辑（参数 OVERWRITE）与 agent 记忆（外部 VERSION/GATE）在**同一个失败点（级联失效）上汇合**——RippleEdits/ROME-multihop（参数）与 MEME（记忆）说的是同一件事。
3. **"不删"成为共识**：RoMem（相位降级）、TOKI（audit 行保留）、DCPM（supersedes 链）、SERAC/GRACE（base 冻结）——越来越多工作**保留旧值**而非物理删除。这正是早先给 ReTrace 的建议"删→失活"的领域级印证。
4. **评测从"检索"转向"真更新"**：MEME / MINTEval / AgingBench / SubtleMemory 集中在 2026 H1 出现，专测 contradiction / cascade / revision-aging——**"最终检索对"已不足以衡量记忆系统**。
5. **belief revision 经典理论被 LLM 复活**：CBM、Struct-Searcher、Bayesian-Agent、BeliefBank 显式调用 belief revision / 后验更新——TMS/AGM 正回到台前。

### 仍待解决（每条都是论文机会）

1. **级联失效（dependent invalidation）**：更新 X→失效依赖 X 的下游。参数侧 ~recall 不传播、记忆侧 MEME Cascade ~3%——**全领域最硬的未解点**。
2. **矛盾判定本身难**：STALE 的 Implicit Conflict best 仅 55.2%；软矛盾（时序/粒度/来源可信度）远比二值复杂。
3. **BRANCH 空地**：保留矛盾为**并行活分支 + 收敛裁决**几乎无人做（TOKI/DCPM/ATMS 是近邻但非此）。
4. **覆盖 vs 保留的代价权衡**：保留旧值利于可追溯/可恢复，但**记忆膨胀**——VERSION 系普遍回避了"版本/patch 自身怎么不爆炸"。
5. **评测仍碎片化**：update 基准刚出、口径不一（LoCoMo 根本不测、LongMemEval 只一个轴），缺统一的 contradiction/cascade 度量。

---

## 四、数据来源说明

- 论文来自 **HuggingFace daily-papers API + arXiv**（2026-01~06，按日聚合绕过月榜页截断）+ 必引经典 prior art。
- 机制摘要基于 arXiv 摘要 / 部分 README；**所有 2026 arXiv id 经 `arxiv.org/abs/<id>` 三方核对**；摘要未披露处标"需读 PDF"。
- 与 `memory/`、`evolve/`、`deep-research/` 去重：重叠论文（δ-mem、Masking、EvoMem、MemSkill 等）仅交叉引用或换视角，不重复深写。
- **ReTrace 关联**：本专题的分析骨架（4 机制 + 级联失效 + BRANCH 空地）即为评估 ReTrace idea 的文献依据，详见各精读"与 ReTrace 的关系"小节与 `README` 矩阵。
