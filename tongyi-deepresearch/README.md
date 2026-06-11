# 通义 DeepResearch 系列阅读笔记

> 来源：[通义 DeepResearch：开源 AI 智能体的新纪元](https://tongyi-agent.github.io/zh/blog/introducing-tongyi-deep-research/)（Tongyi Lab, 2025-09-16）
> 配套模型：Tongyi-DeepResearch-30B-A3B（MoE，30B 总参 / 3B 激活）

## 目录
- [0. 博客总览](#0-博客总览先做一个地图)
- [1. WebWalker](#1-webwalker-benchmarking-llms-in-web-traversal)
- [2. WebDancer](#2-webdancer-towards-autonomous-information-seeking-agency)
- [3. WebSailor](#3-websailor-navigating-super-human-reasoning-for-web-agent)
- [4. WebShaper](#4-webshaper-agentically-data-synthesizing-via-information-seeking-formalization)
- [5. WebWatcher](#5-webwatcher-breaking-new-frontier-of-vision-language-deep-research-agent)
- [6. WebResearcher](#6-webresearcher-unleashing-reasoning-capability-in-long-horizon-agents)
- [7. ReSum](#7-resum-unlocking-long-horizon-search-intelligence-via-context-summarization)
- [8. WebWeaver](#8-webweaver-structuring-web-scale-evidence-with-dynamic-outlines-for-open-ended-deep-research)
- [9. WebSailor-V2](#9-websailor-v2-bridging-the-chasm-to-proprietary-agents)
- [10. Scaling Agents via Continual Pre-training (AgentFounder)](#10-scaling-agents-via-continual-pre-training-agentfounder)
- [11. Towards General Agentic Intelligence via Environment Scaling (AgentScaler)](#11-towards-general-agentic-intelligence-via-environment-scaling-agentscaler)
- [小结：通义 DR 系列演化脉络](#小结通义-dr-系列演化脉络)

---

## 0. 博客总览

通义 DeepResearch 是首个在性能上对标 OpenAI Deep Research 的**全开源 Web Agent**，配套模型 Tongyi-DR-30B-A3B（MoE，30B 总参 / 3B 激活）在 HLE / BrowseComp-EN / BrowseComp-ZH / xBench-DeepSearch 四大高难基准上全部 SOTA。博客还完整公开了一套"**数据合成 → Agentic CPT → Agentic SFT → Agentic RL**"端到端方法学，覆盖算法、数据、基础设施三个层面，并已落地于高德地图"小高老师"和通义法睿。

### 0.1 问题背景
- AI 范式正从 **Chatbot → Autonomous Agent** 迁移：用户希望模型不只回答问题，而是能自主完成"搜索 → 浏览 → 多跳推理 → 综合写作"的端到端深度研究。
- 已有的闭源标杆（OpenAI Deep Research、Gemini Deep Research）效果强但不开源；开源社区缺少一套**可复现、端到端、覆盖数据合成 + 预训练 + 后训练 + RL** 的方法学。
- 极难基准（HLE、BrowseComp-en/zh、xBench-DeepSearch）显示，主流开源 LLM + Tool 的 prompt-only 方案在"高不确定性、长程多跳"任务上几乎不可用，亟需"原生 agentic"训练范式。

### 0.2 方法设计（系统全栈）
通义把 Deep Research 拆为四层：

1. **合成数据**
   - 增量预训练数据：**AgentFounder** 用实体锚定的"开放世界知识记忆"造多风格 (Q, A) + 三类动作数据（规划/推理/多步决策）。
   - 后训练 High-quality QA：从 **WebWalker → WebSailor / V2（图谱随机游走 + 信息混淆）→ WebShaper（集合论形式化）** 逐步升级；并配自动化"博士级"问题升级引擎（迭代复杂度升级循环）。
2. **训练范式：Agentic CPT → Agentic SFT → Agentic RL** 闭环
   - SFT 阶段融合 **ReAct + IterResearch** 两类轨迹，通过拒绝采样选优。
   - RL 阶段基于 **GRPO 定制版**（严格 on-policy、token-level PG loss、leave-one-out 优势估计、过滤"长度溢出"负样本、加大 batch / group 替代 dynamic sampling）。
3. **推理 Rollout 模式**
   - **ReAct 模式**：128K 上下文，纯"思考-行动-观察"循环，最朴素也最能体现底模能力（呼应 The Bitter Lesson）。
   - **深度模式 (IterResearch)**：每轮重建"精简工作空间"，把上轮关键发现凝聚为"演进中的核心报告"，避免上下文窒息；之上叠 **Research-Synthesis** 并行多 agent 合议。
4. **基础设施**
   - 仿真训练环境（离线 Wikipedia + 自研工具套件 + SailorFog-QA-V2 数据）；
   - 统一**工具沙盒**（缓存、重试、饱和式响应）；
   - **自动数据管理**漏斗，按训练动态实时调整训练集；
   - 基于 **rLLM** 的 on-policy 异步 RL 框架。

### 0.3 实验结果（SOTA 概览）
通义 DeepResearch-30B-A3B 在四大极难基准上达到**全开源 SOTA，超越所有已知闭/开源 DR Agent**：

| 基准 | Tongyi-DR-30B-A3B |
|---|---|
| Humanity's Last Exam (HLE) | **32.9** |
| BrowseComp-EN | **43.4** |
| BrowseComp-ZH | **46.7** |
| xBench-DeepSearch | **75.0** |

落地应用：**高德地图"小高老师"**（地图 + 本地生活 Deep Research）、**通义法睿**（法律 Deep Research，在答案要点 / 案例引用 / 法条引用三维度领先行业）。

### 0.4 未来工作
1. 突破 128K 上下文限制；
2. 在 30B 之外验证更大规模的训练流水线；
3. 引入 **partial rollouts** 等技术提升 RL 效率，同时解决离线训练的分布偏移。

---

## [1] WebWalker: Benchmarking LLMs in Web Traversal
> arXiv: 2501.07572 · ACL 2025

**摘要**：首次把"**网页深度遍历**"（沿超链接一层层点进去找答案）单独立为一个被忽视的研究维度，构造了覆盖 4 个领域、按点击深度分级的 **WebWalkerQA**（≈680 道中英 QA），并提出 **Explore-Critic 双 Agent 范式** —— Explorer 负责真实点击/抽取，Critic 负责判断证据是否够用 —— 在所有难度上稳定优于单 ReAct，揭示连 GPT-4o 在该任务上也只有 ~40% 准确率、Hard 级别降至 15–20%，证明纯 RAG 远远不够，必须"横向搜索 + 纵向遍历"结合。

### 问题背景
- 传统 RAG 只看搜索引擎返回的"水平面 snippet"，无法进入承载关键信息的**深层子页面**。
- 真实人类查询多需要从主页出发**沿超链接逐层点击**（垂直遍历），现有 benchmark（WebQA、HotpotQA 等）几乎不覆盖这种"真网页交互"能力。
- 缺乏专门衡量 LLM **横向搜索 + 纵向遍历 + 长程记忆 + "信息何时足够" 判断力**的评测。

### 方法设计
- **WebWalkerQA Benchmark**：从教育 / 会议 / 组织 / 游戏四大领域真实网站构建约 **680 道**中英 QA，按"点击深度"分 Easy（2–3 次）/ Medium（4 次）/ Hard（≥5 次），并分 single-source 与 multi-source 两类。
- **WebWalker 框架 — Explore-Critic 多 Agent 范式**：
  - **Explorer**：ReAct 风格执行 `click_link / go_back / extract_info`，负责真正遍历网页并把关键证据写入记忆；
  - **Critic**：持续读取 Explorer 的观察轨迹，判断"证据是否足够回答原问题"，足够则终止并出答案，否则反馈方向继续探索。
  - 设计动机：把"探索"和"判断"解耦，缓解长上下文失焦、避免无限点击或过早结束。
- **与 RAG 的关系**：WebWalker 作为 RAG 的"垂直补强"——把搜索拿到的入口 URL 进行深度遍历，并把抽取的深层内容回填 RAG 上下文。

### 实验结果
- 在 WebWalkerQA 上即便是 GPT-4o 整体准确率也只在 **~40%**，远未饱和；Hard 级别**降到 15–20% 区间**，体现深度点击越多模型越难追踪。
- **Explore-Critic** 在所有难度上都优于单 ReAct，Hard 级别提升最明显，验证"探索-评判"分工价值。
- 与传统 RAG 联用后能同时拿到横向广度 + 纵向深度，整体优于任一单独方法。
- 关键贡献：**首个评估 LLM 垂直网页遍历能力的 benchmark**；提出**通用化的 explore-critic 双 Agent 范式**；系统揭示主流 LLM 在深度遍历上的能力上限。

---

## [2] WebDancer: Towards Autonomous Information Seeking Agency
> arXiv: 2505.22648

**摘要**：通义 DR 系列首套**端到端、可复现**的"原生 agentic search"训练配方。用 **CRAWLQA + E2HQA** 两路合成数据解决"训练数据稀缺"问题，用 **SFT 冷启动 + DAPO 强化学习** 两段式训练解决"纯 SFT 学不会规划、纯 RL 冷启动崩盘"两难，并保留长/短 CoT 两类教师轨迹兼顾深度与效率。Qwen2.5-32B 基座在 **GAIA 51.5% / BrowseComp-EN 18.0%** 上让开源模型首次逼近闭源 Deep Research，为后续 WebSailor / Tongyi-DR 奠基。

### 问题背景
- 闭源"信息求索智能体"（OpenAI / Gemini Deep Research）端到端自主搜索-浏览-推理，但完全黑盒；开源缺乏可复现配方。
- 两大瓶颈：(1) 数据稀缺——传统多跳 QA 难度不够，不要求真实网页交互；(2) 训练范式不统一——纯 SFT 学不到长程规划，纯 RL 冷启动太差会崩。
- 目标：给出**端到端、可复现**的原生 agentic search 训练范式，让开源 LLM 在 GAIA / BrowseComp 上逼近闭源系统。

### 方法设计（四阶段流水线）
1. **数据合成**
   - **CRAWLQA**：从 arxiv、Wikipedia、GitHub 等权威站点爬取页面，基于多页内容合成"必须真实点击/检索才能回答"的问题。
   - **E2HQA (Easy-to-Hard)**：把简单题里的关键实体用更模糊描述**反向重写**，单跳逐步升级成多跳，实现**难度可控的渐进生成**。
2. **轨迹采样**：基于 ReAct（`search / visit / answer`）用 GPT-4o 等"短 CoT"教师 + **QwQ-32B、DeepSeek-R1** "长 CoT"教师做 rollout，再用"答案正确性 + 格式合法性 + 推理质量"**三道过滤**保留高质量轨迹（保留长/短 CoT 两类）。
3. **SFT 冷启动**：在 Qwen-2.5-7B / 32B 上微调；损失只算在 `Thought / Action` token 上，Observation 不参与梯度，避免拟合工具噪声。
4. **RL 微调**：采用 **DAPO**（Decoupled Clip + Dynamic Sampling）替代裸 GRPO，更适合稀疏奖励 + 长轨迹；奖励以答案正确性为主，配格式辅助信号；对长轨迹做 advantage normalization 缓解长度偏置。

### 实验结果
- **GAIA Pass@1**：WebDancer-7B ≈ **40.7%**，32B ≈ **51.5%**，显著超同基座 prompt-only / 纯 SFT，逼近部分闭源 browsing agent。
- **BrowseComp-EN**：7B ≈ **3.8%**，32B ≈ **18.0%**（大多数开源接近 0），在开源阵营前列。
- **WebWalkerQA**：相对纯 prompt + 工具 baseline 大幅提升，证明端到端训练优于 prompt engineering。
- **BrowseComp-ZH**：保持中文跨语言竞争力。
- 消融：只 SFT 易陷早停/循环；只 RL 奖励太稀疏不收敛；**SFT + DAPO** 最稳；去掉 E2HQA 高难题下降、去掉 CRAWLQA 真实站点泛化变差；长 CoT 教师轨迹对难题增益明显。
- 意义：**开源 Deep Research Agent 的第一套系统化训练配方**，为后续 WebSailor / Tongyi-DR 奠基。

---

## [3] WebSailor: Navigating Super-human Reasoning for Web Agent
> arXiv: 2507.02592

**摘要**：针对"开源 Web Agent 在 BrowseComp 这类**Level 3 极高不确定性 + 信息高度遮蔽**任务上几乎全军覆没"的现实，提出三件套：(1) **SailorFog-QA** 在知识图谱上随机游走 + 信息混淆，生成"在迷雾中导航"的高难训练数据；(2) **RFT 冷启动**重构出简洁专家级轨迹，避免污染推理风格；(3) **DUPO (Duplicating Sampling Policy Optimization)** 通过批内/动态轨迹复制把昂贵 rollout 的利用率提升约 2-3 倍。结果让 72B 开源模型在 BrowseComp-EN 上首次达到 ~12%、BrowseComp-ZH 25-30%+，**首次把开源拉到 DeepResearch 的量级**。

### 问题背景
- 闭源 Web Agent（OpenAI DeepResearch 等）能解决需要"超人推理"的复杂搜索任务，开源几乎不可用。
- 作者将任务按"信息不确定性 × 推理复杂度"分级：
  - Level 1：低不确定性、单步检索
  - Level 2：多跳、路径相对清晰
  - **Level 3**：信息被遮蔽、路径高度发散、需持续探索 + 认知压缩（如 BrowseComp）
- 两大根因：(1) 训练数据没有 Level 3 难度；HotpotQA / 2WikiMultiHopQA 等信息过于直接；(2) 常规 PPO / GRPO 在长轨迹 + 稀疏奖励 + rollout 昂贵的场景下效率低。

### 方法设计
1. **SailorFog-QA 数据合成**
   - 在 **Wikidata** 等结构化知识图谱上做**随机游走**，采样高密度、强耦合的实体子图（喻为"sailor fog"——迷雾般交织的信息网）。
   - 在子图基础上由 LLM 合成 QA，并对实体/关系做**信息遮蔽 (obfuscation)**：用模糊描述代替专名、保留部分线索、隐藏中间实体，迫使智能体真正搜索 + 推理。
   - 结果同时具备**唯一可验证答案**与 **Level 3 级别的高发散探索路径**。
2. **RFT (Rejection Sampling Fine-Tuning) 冷启动**
   - 用强 agent 多次 rollout 收集成功轨迹，再做**轨迹重构**：剥离冗余思维链，重写为简洁、动作导向的"专家级"样例。
   - 让小模型学到的是"如何使用工具与推理"，而非教师模型的具体推理风格。
3. **DUPO (Duplicating Sampling Policy Optimization)**：定制 Agentic RL
   - **训练前批内复制**：对未达方差阈值的样本在 batch 内复制扩充，提升梯度有效性。
   - **训练中动态复制**：把高信息量的成功/失败 trajectory 在 batch 内复用，提升昂贵 rollout 的利用率。
   - 相比标准 GRPO/PPO，DUPO 报告约 **2–3 倍**的训练效率提升。
4. **模型规模**：在 Qwen2.5 / Qwen3 上训练，发布 3B / 7B / 32B / 72B 系列。

### 实验结果
- **BrowseComp-EN**：WebSailor-72B ≈ **12%**，是当时**首个**接近 OpenAI DeepResearch（~51%）量级的开源系统，绝大多数开源 baseline 在 <3%。
- **BrowseComp-ZH**：达到 **25–30%+** 区间，已与部分闭源系统持平或领先。
- **GAIA**：在开源模型中达到 SOTA 级别。
- 小尺寸（3B / 7B / 32B）也展现明显优于同尺寸 baseline 的 scaling 行为。
- 核心结论：**SailorFog-QA + RFT + DUPO** 三件套首次让开源模型在 BrowseComp 这类"超人推理"任务上接近闭源 DeepResearch。

---

## [4] WebShaper: Agentically Data Synthesizing via Information-Seeking Formalization
> arXiv: 2507.15061

**摘要**：把数据合成范式从"**信息驱动** (爬一批文档让 LLM 出题)"升级为"**形式化驱动** (先定义任务结构，再去网上找数据填充)"。基于集合论把信息搜索任务建模为 **R-Projection / R-Union / R-Intersection** 等可组合算子，再用 **Expander Agent** 在形式化骨架上层次化扩展任务，并实时调用工具去 Web 上验证可解性与唯一性。这种"先有数学骨架再有数据"的方式同时解决了**结构覆盖率、难度可控性、答案可验证性**三个老大难。WebShaper-72B 在 **GAIA ≈ 60.1%**，刷新当时开源 SOTA。

### 问题背景
- 现有数据合成主流是**信息驱动**——抓一批网页让 LLM 据此造 QA。两个固有缺陷：
  1. **推理结构与信息结构错位**：模型对"问题应该长什么样"无先验控制，多跳题型单一、推理链浅。
  2. **难以验证**：基于自由文本生成的答案常出现幻觉，质量参差。
- 缺乏对 Information-Seeking (IS) 任务本身的**形式化定义**，所以无法系统性地控制结构、难度、覆盖率与可验证性。
- 目标：从"数据→问题"反转为"形式化结构→数据"，让数据合成成为有数学保证、可控、可扩展的过程。

### 方法设计
1. **集合论形式化 (Set-Theoretic Formalization)**
   - 任务建模为对实体集合的运算：
     - 基本对象：实体集合 $E$
     - **R-Projection**：$R(S) = \{e' \mid \exists e \in S, (e, e') \in R\}$，即"沿关系 $R$ 从集合 $S$ 跳一步"
     - 配合 **R-Union / R-Intersection / R-Difference** 等运算可组合出任意复杂度的多跳 IS 任务
   - 每个任务对应一个**形式化表达式**（结构化"查询计划"），答案就是该表达式求值结果，**天然可验证**。
   - 显著扩大了 chain / tree / 并行约束 / 过滤 / 聚合等任务结构的覆盖率。
2. **Expander Agent (扩张智能体)**
   - 以小型种子任务（如单跳 R-Projection）为起点，逐步在形式化结构上做层次化扩展：替换、复合、增加约束、嵌套并/交。
   - 每一步扩展都通过**调用搜索/检索工具**真实地从 Web 验证新增结点和关系存在且唯一，确保生成任务在真实 Web 上**可解、答案唯一**。
   - 自身就是一个工具使用 Agent，从而"agentic 数据合成"。
3. **验证与轨迹合成**
   - (Question, Formal Expression, Answer) 三元组通过形式化求值 + 工具验证双重保证。
   - 在此数据上合成 Agent 解题轨迹用于 SFT / RL，得到 WebShaper 系列（Qwen 基座，提供 32B / 72B 等）。

### 实验结果
- **GAIA**：WebShaper-72B ≈ **60.1%**，刷新当时开源 SOTA；WebShaper-32B ≈ **53.3%**，同样优于同尺寸 baseline。
- **WebWalkerQA**：达到 **52%+** 准确率。
- **BrowseComp**：在多个尺寸上均优于同期 WebSailor。
- **消融**：去掉集合论形式化、只保留信息驱动合成时性能显著下降，证明 formalization-driven 是性能增益的核心来源。
- 核心结论：基于集合论形式化的 agentic 数据合成在**结构覆盖率、可验证性、难度可控性**上全面优于信息驱动方法，所训模型在 GAIA / WebWalkerQA / BrowseComp 上刷新开源 SOTA。

---

## [5] WebWatcher: Breaking New Frontier of Vision-Language Deep Research Agent
> arXiv: 2508.05748

**摘要**：把 Deep Research 从纯文本扩展到**视觉-语言**模态。基于 Qwen2.5-VL（7B / 32B）构建多模态 ReAct 智能体，集成 Web 搜索、图片搜索、网页浏览、OCR、代码解释器五件套；提出 **CrawlQA** 多模态轨迹合成流水线 + SFT 冷启动 + RL 两段式训练；同时开源 **BrowseComp-VL** 作为视觉-语言深度研究的新基准。WebWatcher 在 BrowseComp-VL / HLE / MMSearch / LiveVQA 上均显著超越 GPT-4o 与开源 VLM agent，是首个真正"会看图查资料"的开源 Deep Research Agent。

### 问题背景
- 现有 Deep Research 智能体（OpenAI DR、Tongyi WebSailor / WebDancer 系列等）都局限在纯文本，无法处理真实世界中含图像、图表、示意图的复杂查询。
- 多模态信息搜索长期被低估，缺少同时处理图像 + 文本、且能跨模态多跳推理的 agent。
- 现有 VLM Agent 的三大不足：缺真正深度研究能力、缺完整多模态工具链、缺高质量训练轨迹与评测基准。
- 目标：打造首个**视觉-语言 Deep Research Agent**，打破纯文本 DR 的边界。

### 方法设计
1. **多模态工具链**：Web 搜索、图片搜索、网页浏览、OCR、代码解释器五类工具协同。
2. **CrawlQA 多模态合成数据流水线**
   - 系统化爬取知识密集型网站（Wikipedia、学术资源、新闻），抽取文本与对应视觉元素（图像、图表、表格），保持模态间关联。
   - 用 GPT-4o 等强模型合成需要**多跳推理 + 视觉定位 + 跨模态信息整合**的 QA 对。
   - 生成完整的"思考-动作-观察"多模态轨迹，并配难度递进与质量筛选。
3. **两阶段训练**
   - **SFT 冷启动**：用 CrawlQA 合成轨迹微调 Qwen2.5-VL，建立基础的多模态工具使用 + 推理范式。
   - **RL**：基于 GRPO 类算法，按结果奖励优化策略，提升多步多模态推理的泛化能力，并通过拒绝采样过滤低质轨迹。
4. **BrowseComp-VL Benchmark**：OpenAI BrowseComp 的视觉-语言扩展版，要求跨多张图像 + 多个文本来源的多跳推理，填补多模态 DR 评测空白。

### 实验结果
> 以下数字综合自多源公开报道，**精确值以论文 Table 为准**。

- **BrowseComp-VL**：≈ **27%**，达 SOTA；其他 VLM baseline 显著低于此。
- **HLE (多模态子集)**：≈ **13–15%**，对比 GPT-4o ≈ 8%。
- **MMSearch**：≈ **55–60%**，大幅超过 GPT-4o ≈ 24%。
- **LiveVQA**：≈ **58.7%**。
- **SimpleVQA**：≈ **51%**。
- 核心结论：WebWatcher 在所有多模态深度研究基准上均超过闭源（GPT-4o / Gemini）和开源 VLM 基线，验证了"多模态合成轨迹 + SFT + RL"训练范式对 VL Deep Research 的有效性；BrowseComp-VL 为后续研究提供了标准化评测平台。

---

## [6] WebResearcher: Unleashing Reasoning Capability in Long-Horizon Agents
> arXiv: 2509.13309

**摘要**：诊断出长时程 Deep Research 的两大根因病症 —— **Context Suffocation（上下文窒息）** 与 **Noise Contamination（噪声污染）**，并提出 **IterResearch** 范式作为治疗：将研究重建为 MDP，每轮维护一份"演进中的核心报告"作为持久化记忆，**周期性巩固 + 丢弃原始观察**让上下文有界增长；配套 **WebFrontier** 工具增强复杂度递进数据合成引擎；并支持并行 Research-Synthesize 多 Agent 合议。30B 模型在 **GAIA 72.8% / HLE 36.7% / BrowseComp-EN 51.7% / BrowseComp-ZH 63.6% / xBench-DeepSearch 70.0%** 全面 SOTA。

### 问题背景
- 现有 Deep Research Agent 多采用"单一上下文累积"(mono-contextual) 的 ReAct 范式，在长时程任务暴露两个根本问题：
  - **Context Suffocation**：步数越多上下文越膨胀，最终超出模型工作记忆，推理质量崩塌。
  - **Noise Contamination**：搜索回来的低质量信息持续污染上下文，信噪比越来越差。
- 数据稀缺：缺乏可扩展、可控难度的长时程研究训练数据。
- 目标：解放长时程研究 Agent 的推理能力，让 Agent 进行真正"无限深度"的研究。

### 方法设计
1. **IterResearch 范式（核心创新）**
   - **MDP 重构**：状态 = 当前"工作空间"(精炼报告 + 最近观察)；动作 = 搜索 / 浏览 / 读取 / 综合 / 巩固 …；转移 = 工作空间更新；奖励 = 与目标契合度。
   - **报告中心工作流**：维护一份不断演化的"研究报告"作为持久化记忆。
   - **周期性巩固 (Periodic Consolidation)**：每轮迭代后把原始观察+思考"蒸馏"成结构化报告条目，然后**丢弃中间噪声上下文**，仅保留巩固报告进入下一轮。
   - 效果：上下文呈**有界**增长，突破上下文长度限制，保持聚焦工作区。
2. **WebFrontier：可扩展数据合成引擎**
   - 通过**工具增强的复杂度递进** (tool-augmented complexity escalation) 机制，从简单种子问题出发借助工具调用让 LLM 不断"加深"问题难度，得到需要多步搜索 + 综合的复杂研究任务及完整轨迹。
3. **并行 Research-Synthesize**：多个 IterResearch Agent 并行探索同一问题，再统一综合，提高推理效率与多样性。
4. **训练**：基于 Qwen 系列约 30B 规模基础模型，SFT + RL 流水线，用 WebFrontier 合成轨迹训练。

### 实验结果
> 数字综合自多源公开报道，**精确值以论文 Table 为准**。

- **GAIA**：≈ **72.8%** (Pass@1)
- **Humanity's Last Exam (HLE)**：≈ **36.7%**
- **BrowseComp-EN**：≈ **51.7%**
- **BrowseComp-ZH**：≈ **63.6%**
- **xBench-DeepSearch**：≈ **70.0%**
- 全面超过 OpenAI DR / GPT-4o / Claude / Gemini 等闭源系统及 WebSailor / WebDancer 等同系列开源工作。
- 核心结论：验证了 IterResearch 范式相比传统 ReAct / mono-contextual 在长时程任务上的显著优势；周期性报告巩固有效解决了上下文窒息和噪声污染；WebFrontier 是 RL 训练成功的关键支撑；并行 Research-Synthesize 进一步提升推理质量。

---

## [7] ReSum: Unlocking Long-Horizon Search Intelligence via Context Summarization
> arXiv: 2509.13313

**摘要**：用"**周期性上下文摘要替代无限累积**"的方式，把 ReAct 升级为理论上**无界 horizon** 的探索范式。三件套：(1) **ReSum 范式** —— 触发阈值后把交互轨迹压缩为"紧凑推理状态"再继续，与 ReAct 完全兼容；(2) **ReSumTool-30B** —— 专门蒸馏的 30B 摘要器，针对长网页搜索轨迹优化，能精准保留关键证据与可追溯链接；(3) **ReSum-GRPO** —— 把 GRPO 与"分段 rollout + 摘要桥接"结合，解决"训练-推理不一致"问题。在 WebSailor 上叠加 ReSum 后所有规模一致显著受益，30B 在 BrowseComp 等极难基准上可与远大于自身的闭源系统比肩。

### 问题背景
- ReAct 范式不断累积"思考-行动-观察"历史，上下文随交互轮数**线性增长**。
- 数十乃至上百轮工具调用的复杂研究任务（如 BrowseComp）很快撞破上下文上限，被迫提前终止。
- 即便不超限，长上下文也会导致 lost-in-the-middle、推理质量退化、token 成本飙升。
- 增大窗口 / 滑动窗口 / 外置记忆等现有方案要么治标不治本，要么破坏关键证据连贯性，无法真正实现"无界探索"。

### 方法设计
1. **ReSum (Reasoning with Summarization) 范式**：当交互历史达阈值，触发摘要器把整段 "思考-行动-观察" 轨迹压缩成 **compact reasoning state**（已收集的关键证据 + 未解决子问题 + 下一步推理方向），丢弃原始 trace，仅基于摘要继续。**Summary-Conditioned Reasoning** 通过与 ReAct 兼容的提示模板让 Agent 直接读最新摘要继续推理，无需改造已有工具调用框架。
2. **ReSumTool-30B 专用摘要工具模型**：通用 LLM 摘要长网页搜索 trace 时容易丢 URL、错过隐含证据；团队基于 30B 模型蒸馏出专用摘要器，能精准抽取关键证据、保留可追溯链接、识别未完成探索分支。
3. **ReSum-GRPO 训练算法（核心创新）**
   - 直接把 ReSum 与现成 ReAct agent 拼接会因"训练-推理不一致"导致性能下降：agent 没见过 summary-conditioned 输入。
   - ReSum-GRPO 将 GRPO 与 summary-conditioned rollout 结合：rollout 时把长轨迹切分成多段，每段在 summary 上继续，把 reward 沿 segment 反向传播。让小模型也能学会"基于摘要继续推理"的策略。

### 实验结果
- 评测基准：**BrowseComp-EN / BrowseComp-ZH / GAIA** 等长时程搜索任务。
- 对比基线：原版 ReAct + WebSailor (Tongyi 自家 Web Agent)、不同尺寸的 Qwen 系列 (7B / 30B / 72B)、OpenAI Deep Research 等商业代理。
- 关键发现：
  - ReSum 对所有规模的 WebSailor 都带来一致增益，**horizon 越长、ReAct 越容易 context overflow 的任务受益越明显**。
  - 经 **ReSum-GRPO** 训练的 WebSailor-30B 在 BrowseComp 等极难基准上取得与远大于自身的闭源系统相当甚至更优的成绩。
  - ReSumTool-30B 作为专用摘要器，相比让 agent 自身摘要或使用通用 LLM 摘要，在保留关键证据与下游答题准确率上均更优。
- 主要贡献：提出与 ReAct 兼容、训练-推理一致的"上下文摘要范式"；开源专用摘要模型 ReSumTool-30B；给出 GRPO 用于"分段 rollout + 摘要桥接"的工程化训练方案。

---

## [8] WebWeaver: Structuring Web-Scale Evidence with Dynamic Outlines for Open-Ended Deep Research
> arXiv: 2509.13312

**摘要**：面向"**开放式深度研究 (OEDR)**"——没有标准答案、需要广搜证据并产出长篇结构化报告的任务（行业调研、综述、政策分析）。提出 **Planner-Writer 双 Agent 系统**：Planner 边搜索边动态演化大纲（**dynamic outline**），把所有证据存入带唯一 ID 的 **memory bank**；Writer 按节做**hierarchical retrieval** —— 每节只拉取本节相关证据进上下文写作，从根本上抑制幻觉并保证引用可追溯。在 DeepResearch Bench / DeepConsult / DeepResearchGym 三大开放式基准上全面 SOTA，幻觉率、引用准确率、覆盖度、逻辑结构评分均明显优于"静态大纲 + 长上下文一次性生成"方案。

### 问题背景
- **静态规划局限**：传统方法一次性生成大纲再按章检索证据。但开放式问题在研究开始前根本无法预知完整结构，静态大纲常导致章节缺失、覆盖不全、内容失衡。
- **长上下文写作幻觉**：一次性把海量网页证据塞进 LLM 写长报告，出现 lost-in-the-middle、引用错配、夸大信息、捏造来源等问题。
- **证据-写作链路断裂**：缺乏可追溯的"引用-段落"映射，最终报告难以保证每个论断都有 Web 证据支持。
- **不符合人类研究流程**：人类研究员是"边搜边写边改大纲"的迭代过程，而非"一锤定音"。

### 方法设计
1. **Planner Agent（规划者）**
   - 在 Web 搜索 (evidence acquisition) 与大纲优化 (outline optimization) 之间**动态交替**。
   - 每次新增证据后立即评估对当前大纲的影响：新增章节 / 合并 / 拆分 / 调整层级或顺序。
   - 大纲是"**活的**" dynamic outline，最终形态是带"证据引用 ID"的层级结构，每个小节关联 memory bank 中的 evidence ID。
2. **Memory Bank（记忆库）**：集中存放所有 Web 证据片段，每条带唯一 ID，支持精确检索，是 Planner 与 Writer 间的桥梁。
3. **Writer Agent（写作者）**
   - 接收 Planner 产出的带引用大纲。
   - 采用 **hierarchical retrieval + section-by-section 写作**：逐节生成报告，每节只从 memory bank 中拉取本节相关证据子集进入上下文。
   - 从根本上抑制长上下文写作幻觉，并保证每段论断**有源可依**。
4. **核心创新**：动态大纲、证据-引用一体化大纲、分层按需检索、Planner-Writer 解耦分工。

### 实验结果
- 评测基准：**DeepResearch Bench / DeepConsult / DeepResearchGym** 三大开放式深度研究基准。
- 对比基线：OpenAI Deep Research、Gemini Deep Research、Perplexity Deep Research 等闭源；GPT-Researcher 及静态大纲 / 单 Agent 长上下文一次性生成方案等开源；同基座下的消融变体。
- 关键发现：
  - WebWeaver 在三大基准上均达到 SOTA。
  - 相比"静态大纲 + 长上下文一次性生成"，幻觉率显著下降、引用准确率明显提升、报告覆盖度与逻辑结构评分更好。
  - **消融**：去掉动态大纲或去掉分层检索写作，性能均明显退化，证明两个核心设计缺一不可。
  - 长篇报告的人类评估（factuality / coherence / coverage / citation quality）全面优于现有开源 baseline，与商业 DR 产品竞争。
- 主要贡献：提出首个面向 OEDR 的"**动态大纲 + 双 Agent + 分层检索写作**"系统化框架；给出引用可追溯、低幻觉的长篇研究报告生成范式；实证"模仿人类研究流程"的工程价值。

---

## [9] WebSailor-V2: Bridging the Chasm to Proprietary Agents
> arXiv: 2509.13305

**摘要**：WebSailor 的 V2 升级，目标是"**填平开源与闭源 Deep Research 的鸿沟**"。两条主线：(1) **SailorFog-QA-V2** —— 在 V1 图谱采样基础上生成更复杂、更高熵的多跳"模糊信息"QA；(2) **可扩展 Agentic RL** —— GRPO 基础上叠 DUPO 等定制策略 + 异步 actor-learner 分离架构，专门攻克长程 rollout 的稀疏奖励、采样效率与稳定性。基于 **30B-A3B (Qwen3 MoE)** 基座，配合 RFT 冷启动 + 大规模 RL，是 Tongyi-DR-30B-A3B 的核心实现之一；在 BrowseComp-EN 43.4 / BrowseComp-ZH 46.7 / HLE 32.9 / xBench-DeepSearch 75.0 等 SOTA 数据上贡献关键能力。

### 问题背景
- 开源 Web Agent 与 OpenAI / Gemini Deep Research 等专有系统在长程信息检索和复杂多跳推理上存在显著"chasm"。
- 前作 WebSailor 已展示出潜力，但仍受限于：(1) 训练数据复杂度不足，难以覆盖"高不确定性 + 模糊推理"场景；(2) RL 训练流程在长程 rollout 下成本高、扩展性差。
- V2 旨在通过**更高质量的合成数据**与**可扩展 RL 训练**进一步缩小与专有 Deep Research 系统的差距。

### 方法设计
1. **SailorFog-QA-V2 数据合成**：在 V1 图采样基础上升级，生成更复杂的多跳、高熵 QA，要求 agent 在"模糊"信息中执行长程信息检索。基于知识图谱实体纠缠采样，控制问题难度。
2. **两阶段训练**：RFT 冷启动 (Rejection Sampling FT) → 大规模 RL 后训练。
3. **可扩展 RL 框架**：在 GRPO 基础上采用 **DUPO** 等定制策略，解决长程 rollout 的稀疏奖励、采样效率与稳定性问题；引入**异步 rollout / actor-learner 分离架构**提升 GPU 利用率。
4. **基座模型**：**30B-A3B**（30B 总参 / 3B 激活的 MoE，基于 Qwen3 系列）。

### 实验结果
作为 Tongyi DeepResearch 30B-A3B 的核心实现之一，公开报道的对比对象包括 OpenAI DR、Gemini DR、闭源 GPT-4o / Claude，以及多个开源 agent（WebSailor v1、WebDancer 等）。代表性数字：

| 基准 | WebSailor-V2 / Tongyi-DR-30B-A3B | OpenAI Deep Research |
|---|---|---|
| BrowseComp-EN | **43.4** | ≈ 51.5 |
| BrowseComp-ZH | **46.7** | ≈ 42.9 |
| HLE | **32.9** | ≈ 26.6 |
| xBench-DeepSearch | **75.0**（SOTA 开源） | — |
| GAIA | **70+** | — |
| FRAMES | **90.6** | — |

> 注：上表数字综合自博客 + 公开报道；其中哪些数字专属 WebSailor-V2、哪些属于整套 Tongyi-DR-30B-A3B 系统，**需查阅论文原文 Table 区分**。

---

## [10] Scaling Agents via Continual Pre-training (AgentFounder)
> arXiv: 2509.13310

**摘要**：把"提升 Agent 能力"从传统的"对齐阶段问题"重新定位为"**预训练阶段问题**"。提出 **Agentic Continual Pre-Training (Agentic CPT)** 范式，在通用 LLM 之上用海量"thought-action-observation-reflection"轨迹做继续预训练，构建专用的 Agent 基础模型 **AgentFounder-30B**。数据由 **First-order（直接动作）+ Second-order（元层级反思）** 两类合成动作组成，与后训练 SFT + RL 无缝衔接，是 Tongyi-DR 系列里"基座专用化"那一层的关键作品。

### 问题背景
- 通用 LLM 直接做 SFT / RL 进入 agent 任务时，常出现训练不稳、收敛慢、agentic prior 不足等问题。
- 根因：通用预训练语料中"工具调用、规划、自反思、多跳推理"等 agent 行为信号极度稀缺。
- 目标：在基座 → agent 之间增加一层桥梁，在大规模合成 agentic 语料上做持续预训练，构建专用 agent foundation model。

### 方法设计
1. **Agentic CPT 范式**：在通用 LLM（基于 Qwen3 30B-A3B MoE）之上，使用海量 agent trajectory（thought-action-observation-reflection）做继续预训练，赋予模型 tool-use、planning、long-horizon reasoning 的内禀先验。
2. **First-order / Second-order 动作合成**
   - **First-order**：直接生成与环境交互的工具调用动作数据（API call、search、browse 等）。
   - **Second-order**：生成元层级的推理、规划和反思数据（对 first-order 行动的判断、修正）。
   - 二者混合以同时学习"做"和"想怎么做"。
3. **端到端衔接**：CPT 之后再做 SFT + RL（与 WebSailor-V2 等共享的后训练管线），得到 **AgentFounder-30B**。

### 实验结果
- 基线：Qwen3 系列原始模型、不做 Agentic CPT 直接 SFT/RL 的 ablation、其他开源 agent（WebSailor、WebDancer 等）、闭源 OpenAI Deep Research。
- 核心收益体现在 BrowseComp、BrowseComp-ZH、GAIA、HLE、xBench-DeepSearch 等 Deep-Research 基准上，数字范围与 Tongyi-DR-30B-A3B 整体相当（HLE ≈ 32.9 / BrowseComp-EN ≈ 43.4 / xBench ≈ 75）。
- 论文应包含 **CPT vs no-CPT 的 ablation 对比与训练效率提升数据**，具体数字需查阅论文原文。
- 意义：首次系统论证"**Agentic CPT 是从基座到 Agent 不可绕过的一层**"，可扩展、可数据飞轮，是通义 DR 体系的"基础设施层"。

---

## [11] Towards General Agentic Intelligence via Environment Scaling (AgentScaler)
> arXiv: 2509.13311

**摘要**：把 agent 训练的瓶颈定义为"**环境瓶颈**"——可交互训练环境数量少、领域单一、工具集封闭。提出 **Environment Scaling**：自动化合成成百上千个跨域可交互模拟环境（电商、出行、医疗、金融、办公等），每个环境包含 schema、API/工具、状态机和验证器。两阶段训练（广覆盖 + 领域精化）+ 环境内置 verifier 过滤高质量轨迹。验证了"**环境数量 × 多样性 → agent 能力**"的 scaling 关系，让较小模型在 τ-bench、BFCL 等多轮 function calling 任务上逼近或超越更大的专有模型。

### 问题背景
- 现有 agent 训练受限于"环境瓶颈"：可交互训练环境数量少、领域单一、工具集封闭，导致 agent 难以泛化到多样化真实场景。
- τ-bench、BFCL 等评测显示开源模型在多轮 function calling、跨域工具使用上仍远落后于专有系统。
- 目标：通过**大规模、多样化地合成可交互环境**驱动通用 agentic intelligence。

### 方法设计
1. **可扩展环境合成管线**：自动化生成跨多个领域（电商、出行、医疗、金融、办公等）的模拟环境，每个环境包含 schema、API / 工具集合、状态机与验证器，从手工构造的几个环境扩展到成百上千个。
2. **两阶段训练框架**
   - **Phase 1**：在大规模、多样化基础环境上做广覆盖训练，培养通用工具使用与多轮交互能力。
   - **Phase 2**：在更复杂、领域专用环境上做精化训练（包含专家数据 + RL），提升复杂任务完成率。
3. **轨迹生成与质量过滤**：agent 在合成环境中自我交互生成 trajectory，通过环境内置 verifier（状态比对）过滤高质量样本。
4. **AgentScaler 模型**：在 Tongyi 基座（30B-A3B 等规模）上验证**环境数量 ↔ agent 能力**的 scaling law。

### 实验结果
- 主要 benchmark：**τ-bench**（retail / airline 多轮客服）、**BFCL**（function calling），可能含 ToolBench、AgentBench。
- 对比对象：Qwen 基线、Llama 系列、GPT-4o、Claude、其他 tool-use 专用模型（xLAM、ToolACE、APIGen 等）。
- 关键结论：
  - 环境数量与多样性增加对 agent 能力呈正相关 scaling 趋势。
  - 经 AgentScaler 训练的较小模型可逼近或超过更大基线在多轮 function calling 上的表现。
- 具体数字（τ-bench pass^k、BFCL 子类别准确率等）需查阅论文原文确认。
- 意义：把"环境"提升到与"数据 / 算法"同等地位，给出可扩展的"环境造厂"方法，是通义 DR 体系里"通用工具能力"那一层的关键作品。

---

## 小结：通义 DR 系列演化脉络

把 11 篇放进一张"演化图"，会发现它们大致沿三条主轴推进：

### 主轴 A：数据合成范式（"题怎么出"）
$$\text{WebWalker (真实点击流逆向)} \to \text{WebSailor (图谱随机游走 + 信息混淆)} \to \text{WebShaper (集合论形式化)} \to \text{WebSailor-V2 (SailorFog-QA-V2)}$$

- 从"造一道像样的多跳题"到"用数学骨架批量造可控难度的题"，**可扩展性、可验证性、结构覆盖率**逐步打通。

### 主轴 B：训练范式（"模型怎么学"）
$\text{WebDancer (SFT + DAPO 配方)} \to \text{WebSailor (RFT + DUPO)} \to \text{ReSum (ReSum-GRPO)} \to \text{AgentFounder (Agentic CPT)} \to \text{Tongyi-DR (Agentic CPT - SFT - RL 闭环)}$    

- 从"后训练配方"演化成"覆盖预训练-后训练-RL 的全栈范式"，特别是 **Agentic CPT** 把能力起点从"对齐阶段"推到"预训练阶段"。

### 主轴 C：推理 / 部署范式（"上下文怎么管"）
$$\text{WebWalker (Explore-Critic 双 Agent)} \to \text{WebResearcher (IterResearch 周期性巩固)} \to \text{ReSum (Context Summarization)} \to \text{WebWeaver (Dynamic Outline + Hierarchical Retrieval)}$$

- 从单纯的"多 Agent 分工"到"主动管理上下文的有界增长"，本质上都是在和 **Context Suffocation / Noise Contamination** 斗争。

### 主轴 D：能力扩展
- **WebWatcher** 把 DR 从文本扩到 **视觉-语言** 模态。
- **AgentScaler** 把瓶颈定义为"环境数量与多样性"，给出 **Environment Scaling** 这条新的可扩展轴。

### 结论
通义 DR 给开源社区贡献的，不仅是 Tongyi-DR-30B-A3B 这个模型，而是一整套"**数据 + 训练 + 推理 + 环境**"四位一体、互为飞轮的方法论。其中最具方法论级影响力的设计是：
1. **WebShaper 的集合论形式化** —— 让数据合成成为"可控制的数学过程"；
2. **IterResearch / ReSum 的上下文管理** —— 把"上下文窒息"从工程问题变成可被范式解决的问题；
3. **Agentic CPT** —— 把 Agent 能力从对齐推到预训练，是真正"为 Agent 时代准备的基座"。

---

## 参考链接
- 博客：<https://tongyi-agent.github.io/zh/blog/introducing-tongyi-deep-research/>
- 代码：<https://github.com/Alibaba-NLP/DeepResearch>
- 模型：<https://huggingface.co/Alibaba-NLP/Tongyi-DeepResearch-30B-A3B>
- 11 篇 arXiv：2501.07572 / 2505.22648 / 2507.02592 / 2507.15061 / 2508.05748 / 2509.13309 / 2509.13313 / 2509.13312 / 2509.13305 / 2509.13310 / 2509.13311



