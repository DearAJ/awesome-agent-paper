# Deep Research 专题 — HuggingFace 月榜 (2025-11 至 2026-06)

> **数据范围**：HuggingFace Papers **月榜** 2025-11、2025-12、2026-01、2026-02、2026-03、2026-04、2026-05、2026-06（共 8 个月）
> **标记**：⭐ = 本专题精读论文（共 13 篇）｜📄 = 基于 arXiv 摘要的简短描述
> **目录定位**：`Q:/awesome-agent-paper/deep-research/`

---

## 一、按月列出（每篇 500–1000 字详述，摘要均基于 arXiv 原文）

> 每篇展开为 500–1000 字，覆盖**问题动机 / 核心方法 / 关键结果 / 意义与定位**。⭐ = 本专题精读，📄 = 简短描述（已扩写），🔗 = 已在 `huggingface/` 精读、此处交叉引用。

### 2025-11（11 月）

#### #4 MiroThinker: Pushing the Performance Boundaries of Open-Source Research Agents via Model, Context, and Interactive Scaling ⭐ [MiroMind, 196↑] — arXiv 2511.11793

**问题动机**：开源 research agent 一直在两个维度上做 scaling——把模型做大（model size）、把上下文做长（context length）。但这两条路都遇到瓶颈：纯粹拉长推理链会**退化**（孤立运行、误差累积、没有外部纠错），单纯堆模型规模成本又过高。社区缺一个**可预测的第三 scaling 轴**。

**核心方法**：MiroThinker 提出**第三个 scaling 维度——interaction scaling（交互规模）**：系统性地训练模型处理**更深、更频繁的 agent-environment 交互**。与单纯的 test-time scaling（推理链拉长）的本质区别在于——interactive scaling 靠**环境反馈 + 外部信息获取**来纠错、refine trajectory，而不是让模型在真空里自言自语。这把"交互深度"确立为一条**可预测的 scaling law**：交互次数越多，性能越高，且这种增长是稳定的、可外推的。在 256K 上下文窗口下，单个任务可执行**最多 600 次 tool call**。

**关键结果**：72B 变体在 **GAIA 81.9% / HLE 37.7% / BrowseComp 47.1% / BrowseComp-ZH 55.6%**——超过此前所有开源 agent，逼近 GPT-5-high。GitHub 8.2k★。

**意义**：MiroThinker 是开源 research agent 的**奠基性工作**——它把"交互"从一个工程细节提升为与 model size、context length 并列的第三 scaling 轴，为后续 IterResearch（交互推到 2048 次）、SearchSwarm（委派智能）等"交互维度"工作奠定了基础。已成精读 [[01-mirothinker-v1]]。本专题作为"interaction scaling"主线的源头。

#### #5 General Agentic Memory Via Deep Research ⭐ [BAAI, 171↑] — arXiv 2511.18423

**问题动机**：广泛采用的 **static memory（静态记忆）**——提前构建好"即取即用"的记忆——必然带来**严重信息损失**：写入时不知道未来问什么，任何提前压缩都会丢掉当时看似无关、日后却关键的细节，一旦丢失下游再强推理也找不回。

**核心方法**：General Agentic Memory（GAM）借编译器的 **JIT（Just-In-Time）编译原则**，提出 **duo-design**：① **Memorizer**——用轻量 memory 标注关键历史信息的索引/高亮，同时把**完整历史无损保存**在 universal page-store；② **Researcher**——运行时根据在线请求，在预构建 memory 引导下从 page-store **像做 deep research 一样按需检索整合**。换言之"记忆检索 = 一次以历史为语料的深度研究"。充分利用 frontier LLM 的 agentic 能力与 test-time 可扩展性，可端到端 RL 优化。

**关键结果**：GitHub 855★。在长程记忆基准上显著优于 static memory，尤其在"需组合多个分散历史片段"的查询上。

**意义**：GAM 把"记忆系统"从"提前建好的知识库"翻转为"运行时按需重建的研究过程"，是"don't compress early, reconstruct on demand"思想的奠基表达。已成精读 [[11-general-agentic-memory]]，同时是 `memory/` 专题"记忆 = 运行时重构"主线的源头。

#### #36 IterResearch: Rethinking Long-Horizon Agents via Markovian State Reconstruction ⭐ [阿里 Tongyi Lab, 80↑] — arXiv 2511.07327

**问题动机**：现有 deep-research agent 用 **mono-contextual 范式**（所有信息堆进单一膨胀的上下文窗口），导致 **context suffocation（上下文窒息）**——有效信息被无关 token 稀释、推理容量被吃满——与 **noise contamination（噪声污染）**——早期失败路径、过时观测一直挂在上下文里干扰决策。

**核心方法**：把长程研究重构为 **MDP + 策略性工作区重建**——维护一个 **evolving report（演进报告）作为记忆**，每隔若干步把已有信息**合成为洞见**写进报告、丢弃冗余原始内容，基于"重建后的紧凑工作区"继续决策，从而在任意探索深度下保持稳定推理容量。配套 **EAPO（Efficiency-Aware Policy Optimization）**：几何奖励折扣激励高效探索 + 自适应下采样实现稳定分布式训练。

**关键结果**：6 个基准平均 **+14.5pp**；交互扩展到 **2048 次**时性能从 3.5% 飙到 42.5%；作为纯 prompting 策略可让 frontier 模型在长程任务上比 ReAct **+19.2pp**。

**意义**：IterResearch 给"上下文该不该装下一切"明确的否定回答——**上下文是需要周期性 curate 的状态**。它与 Harness-1 的"状态外置"是同一病症（mono-context 膨胀）的两条解法。已成精读 [[02-iterresearch]]，亦是阿里通义 DeepResearch 系（见 `tongyi-deepresearch/`）的关键基建。

#### #24 GeoVista: Web-Augmented Agentic Visual Reasoning for Geolocalization 📄 [Tencent Hunyuan, 98↑] — arXiv 2511.15705

**问题动机**：现有 agentic 视觉推理多聚焦图像操作工具，缺乏更通用的 agentic 模型。地理定位（判断一张照片拍摄于地球何处）此前多被当作纯 retrieval 问题（query embedding 找最近邻），但人类做法是"看图找路标 → 想区域 → web 搜索确认/refine 假设"，是显式的工具增强推理过程。

**核心方法**：GeoVista 把地理定位**重新表述为 agentic 任务**——在推理 loop 内无缝集成 **image-zoom-in 工具**（放大兴趣区域看清细节）与 **web-search 工具**（搜索路标/地名确认假设）。完整训练 pipeline = **cold-start SFT**（学推理模式与工具先验）+ **RL**（hierarchical reward 利用多层级地理信息——国家/城市/街道逐级奖励）。配套发布 **GeoBench** 基准（覆盖全球照片 + 全景图 + 卫星图三种视角）。

**关键结果**：大幅超过开源 agentic 模型，多数指标逼近 Gemini-2.5-flash / GPT-5。

**意义**：GeoVista 把"thinking with tool"这一抽象原则具象到地理定位域，证明在专门任务上 agentic 设计（视觉 grounding + web 搜索）可超越纯 retrieval 范式，并逼近 frontier 闭源。它与 huggingface 库的 Thinking-with-Map（地图工具 agentic RL）同属"视觉 + web 工具 + RL"的地理定位路线，是多模态 deep research 在垂直任务上的代表。

#### #47 DR Tulu: Reinforcement Learning with Evolving Rubrics for Deep Research ⭐ [AI2 / UW, 63↑] — arXiv 2511.19399

**问题动机**：多数开源 DR 模型靠 **short-form QA 的 RLVR** 训练（答案唯一、好判分），但这**无法迁移**到真实的 long-form 任务——后者要产出长篇、有引用、多源的答案，没有唯一标准答案，无法用固定 verifier 判分。

**核心方法**：**RLER（RL with Evolving Rubrics）**——构建并维护与 policy 模型 **co-evolve（共同演化）** 的 rubric：rubric 能不断纳入模型新探索到的信息 + 对比模型回答，从而对长答案做更细、更有判别力的评分，提供 discriminative 的 on-policy 反馈。换言之，当答案不可验证时，**让评分标准本身在训练中长出来**。

**关键结果**：**DR Tulu-8B** 是首个直接为开放式 long-form deep research 训练的开源模型；在科学/医疗/通用 4 个 long-form 基准上大幅超现有开源 DR 模型，匹配或超过商用 DR 系统（与 OpenAI DR 持平、平均超 Tongyi DR +15.6%），且**每 query 便宜 1000×**。配套发布 MCP-based agent 基础设施。GitHub 659★。

**意义**：DR Tulu 回答了 DR RL 的核心难题——"长答案不可验证时奖励从哪来"——用演化 rubric 把可验证性从"固定 verifier"扩展到"会成长的评分标准"。已成精读 [[03-dr-tulu]]，与 `agentic-rl/12-dr-tulu` 同源（本专题侧重 DR 系统全貌，agentic-rl 专题侧重 RL 训练机制）。

### 2025-12（12 月）

#### #16 Probing Scientific General Intelligence of LLMs with Scientist-Aligned Workflows (SGI-Bench) 📄 [Intern Science, 120↑] — arXiv 2512.16969

**问题动机**：尽管科学 AI 进展迅速，但缺乏一个连贯框架来定义 **SGI（Scientific General Intelligence，科学通用智能）**——即"自主构思、调查、跨科学领域推理"的能力。

**核心方法**：本文提出基于 **PIM（Practical Inquiry Model：Deliberation 审议 → Conception 构思 → Action 行动 → Perception 感知）** 的可操作 SGI 定义，并落地为**四个科学家对齐任务**：deep research（深度研究）、idea generation（想法生成）、dry/wet experiments（干/湿实验）、experimental reasoning（实验推理）。配套 **SGI-Bench**——1000+ 专家策划的跨学科样本，灵感来自《Science》125 个大问题。还提出 **TTRL（Test-Time RL）**——推理时优化 retrieval-augmented novelty reward，无需参考答案即可提升假设新颖度。

**关键结果**：揭示多个 gap——deep research 即使 step-level 对齐，exact match 也仅 **10–20%**；ideas 缺可行性与细节；dry 实验代码可执行率高但结果准确率低；wet 实验序列保真度低；多模态比较推理持续困难。113 位作者的大型协作。

**意义**：SGI-Bench 把"科学通用智能"从模糊概念形式化为 PIM 框架 + 工作流中心的基准，揭示了 deep research 在科学场景下"step 对齐但 exact match 极低"的严峻现状。它与同机构 Intern Science 的 InternAgent-1.5 形成"评测 + 系统"的一对，是 deep research 向科学发现深化的代表。

#### #32 Step-DeepResearch Technical Report ⭐ [StepFun, 88↑] — arXiv 2512.20491

**问题动机**：随着 LLM 转向自治 agent，Deep Research 成为关键指标。但 BrowseComp 等学术基准**无法满足开放式研究的真实需求**——后者需要 intent recognition（意图识别）、long-horizon decision-making（长程决策）、cross-source verification（跨源验证）三大能力。

**核心方法**：Step-DeepResearch 是一个高性价比、端到端 agent。三大创新：① **基于 Atomic Capabilities 的数据合成策略**——把研究能力分解为原子能力单元（planning、report writing 等），针对性强化；② **渐进式训练路径**——agentic mid-training → SFT → RL 三阶段；③ **Checklist-style Judger（清单式评判器）**——显著提升鲁棒性。还建立 **ADR-Bench** 填补中文 DR 评测空白。

**关键结果**：32B 模型在 **Scale AI Research Rubrics 拿 61.4%**；ADR-Bench 上大幅超过同级模型，**匹敌 OpenAI / Gemini DeepResearch** 等 SOTA 闭源。证明精细训练能让中等规模模型达到专家级能力 + 工业领先的性价比。GitHub 560★。65 位作者。

**意义**：Step-DeepResearch 证明"中等规模 + 精细训练"可匹敌闭源 frontier DR——atomic capability 数据合成 + 三阶段渐进训练成为开源 DR 的实用配方。已成精读 [[04-step-deepresearch]]，与 OpenSeeker/OpenResearcher 同属"开源 DR 系统 + 训练范式"主线。

#### #42 Deep Research: A Systematic Survey ⭐(综述) [73↑] — arXiv 2512.02038

**问题动机**：DR 领域快速膨胀但缺乏系统梳理——各种工作的术语、组件、技术各说各话，缺一份清晰的 roadmap 与分类框架。

**核心方法**：这是 **DR 领域首个系统性综述**，四大核心贡献：① 形式化 **三阶段 roadmap** 并区分 DR 与相关范式（single-shot prompting、标准 RAG——DR 强调 critical thinking、multi-source、verifiable output）；② 提出四大组件——**query planning（查询规划）、information acquisition（信息获取）、memory management（记忆管理）、answer generation（答案生成）**——各配 fine-grained 子分类；③ 总结优化技术（prompting / SFT / agentic RL）；④ 整合评测标准与开放挑战。

**关键结果**：GitHub 318★（持续更新）。提供 DR 领域的术语体系、分类框架、技术总结与开放问题清单。

**意义**：本综述是**本专题的阅读地图**——它的"query planning → information acquisition → memory management → answer generation"四组件框架，是本专题"研究现状与趋势"一节的分析主线。所有精读工作都能在这个框架里定位（如 IterResearch/Harness-1 在 memory management，OpenSeeker/OpenResearcher 在 information acquisition 的数据侧）。

### 2026-01（1 月）

#### #3 Watching, Reasoning, and Searching: A Video Deep Research Benchmark 🔗(已收录 `huggingface/04`) [QuantaAlpha, 214↑] — arXiv 2601.06943

**问题动机**：填补文本 DR（BrowseComp 等）与多模态理解（VideoMME 等）之间的空白——缺一个**视频 deep research** 基准，要求"视频提供视觉锚点，但答案必须从开放 web 检索"。

**核心方法**：**首个 video deep research benchmark**——以视频为视觉锚点（如"这段视频是哪起新闻事件"），但最终答案必须从开放 web 检索得到，必须做"视频→实体/事件→web 搜索→多跳推理→合成答案"的跨模态深度研究。100 个样本全部通过两个 ablation 保证真正需要跨模态：Web-only 失败（无视频找不到关键词）+ Video-only 失败（无 web 拼不出答案）。系统比较 Workflow（固定流程）与 Agentic（自由探索）× 多个 MLLM。

**关键结果**：**反直觉发现——Agentic 不必优于 Workflow**：强模型（Gemini-3）在 agentic 模式有显著增益（+10pp+），但弱模型（MiniCPM-V）在 agentic 模式反而**退化 9 分**——多步工具调用放大了底层视觉感知错误。

**意义**：VideoDR 把"video + DR + agentic 三者交叉点"建成可重复评测，并指出 agentic 范式有 **floor effect**（弱模型用不好自由度）。详见 `huggingface/04-videodr.md`，本专题作为多模态 DR 的**起点**交叉引用——它揭示的"视觉感知错误被多步放大"是后续 Vision-DR/OpenSearch-VL 用工具增强缓解的核心痛点。

#### #18 DeepResearchEval: An Automated Framework for Deep Research Task Construction and Agentic Evaluation ⭐ [Infinity Lab, 127↑] — arXiv 2601.09688

**问题动机**：DR benchmark 面临双重痛点——**构造成本高**（需专家设计真实复杂任务）+ **静态评测过时快**（固定 benchmark 被新模型迅速刷爆）。

**核心方法**：**双框架**应对。① **任务构造**——persona-driven pipeline（为律师/医生/记者等多领域专家 persona 生成真实复杂的研究需求）+ 两阶段过滤：**Task Qualification**（任务是否真实可解）+ **Search Necessity**（是否真的需要多源证据，过滤掉单页就能答的）；最终保留只有多源证据 + 多步推理才能完成的任务。② **评测设计**——**Adaptive Point-wise Quality Evaluation**（根据任务复杂度动态推导评测维度/权重）+ **Active Fact-Checking**（即使 agent 没给引用，框架也主动 web 搜索验证关键断言）。

**关键结果**：在 GPT-5 / Claude / Gemini 等 frontier 模型上的得分分布更具区分度，比固定 benchmark 更能反映真实差异；自动构造的任务在多月后仍保持挑战性。GitHub 137★。

**意义**：DeepResearchEval 把 DR 评测从"人工构造静态 benchmark"推进到"自动构造 + 主动事实核查"，解决了"benchmark 被快速饱和"的根本问题。已成精读 [[13-dr-eval-and-error]]（与 Span-Level Error、K-BrowseComp 合并为"DR 评测三件套"）。

#### #22 Youtu-Agent 🔗(已收录 `huggingface/05`) [Tencent, 119↑] — arXiv 2512.24615

**问题动机**：当下 agent 系统两大痛点——**手工搭建 agent 太贵**（tool 集成 + prompt 工程需数百 person-days）+ **RL 训练成本极高**（上万美元算力）。如何把 agent 的"自动生成"与"持续优化"端到端整合，大幅降低这两项成本？这对 DR 系统尤其相关——deep research agent 的搭建（多工具集成、检索/合成 prompt 设计）和优化（在真实 web 上 RL）都是这两项痛点的重灾区。

**核心方法**：Youtu-Agent（腾讯优图）是把 agent 自动生成与持续优化端到端整合的开源框架，四件套：① **三层 YAML 框架**（Agent/Tools/Env 解耦，声明式描述，秒级换 LLM 后端）；② **自动配置**两路——Workflow（专家模板）+ Meta-Agent（基于用户自然语言需求由 LLM 自动生成 YAML 与 prompt）；③ **Training-free GRPO**（核心创新——用 LLM evaluator 把 group rollout 的成败信号蒸馏成"语义 advantage"，注入 textual LoRA（外挂 system prompt + few-shot），实现 **$18 击败 $10K 传统 RL** 的同等效果，完全不更新权重）；④ **端到端 Agent RL**（基于 Agent-Lightning + VeRL，稳定扩展到 128 GPU 并行 rollout）。

**关键结果**：WebShop / GAIA / SWE-bench Lite 等多基准达开源 SOTA；Training-free GRPO 在多个 agent 任务相对 baseline 提升 **8–15pp**。

**意义**：Youtu-Agent 把"data synthesis + automated configuration + training-free optimization"做成完整工程闭环，是 2026 H1 工业级 agent 框架的代表作。其 **agentic search** 能力直接支撑 DR 任务（GAIA 等基准含大量需多步检索的研究型任务），且 **Training-free GRPO** 这一"frozen 模型 + 文本侧优化"范式对 DR 系统尤其有吸引力——在闭源 frontier 模型时代，无法 fine-tune 权重时，靠 textual LoRA 优化 DR agent 的 prompt/few-shot 是一条低成本路径（$18 vs $10K）。本专题作为旗舰框架对 DR 的支撑列出（非精读），详见 `huggingface/05-youtu-agent.md`。它与 MATTRL（测试时经验注入）、SkillOpt（text-space skill 优化）同属"frozen model + 文本侧优化"潮流。

#### #9 LongCat-Flash-Thinking-2601 Technical Report 📄 [Meituan LongCat, 181↑] — arXiv 2601.16725

**问题动机**：如何训出一个在 agentic 基准上达 SOTA 的大规模开源推理模型，同时兼顾计算效率？随着 LLM 从被动应答转向自主 agent，"agentic reasoning + 高效推理"成为旗舰模型的新竞争维度——既要会多轮工具调用、长程决策，又要在 MoE 架构下控制推理成本。

**核心方法**：LongCat-Flash-Thinking-2601（美团 LongCat）是 **560B 参数 MoE 推理旗舰**，通过 **domain-parallel expert training + fusion（领域并行专家训练 + 融合）** 的统一训练框架，在 agentic 基准上达 SOTA——不同领域的专家并行训练后融合，兼顾专精与泛化。论文把 **agentic search（agentic 搜索）** 明确列为关键能力维度之一——模型具备**多轮工具调用、tool-integrated reasoning** 能力，能完成 deep research 类任务（多步检索、证据收集、跨源验证、长报告合成）。采用 **RL 训练 multi-turn tool use**——通过强化学习让模型在长程工具交互中学会规划与纠错，而非靠 prompt 临时拼装。

**关键结果**：在多个 agentic 基准上达到开源 SOTA 级表现，agentic search 为其核心能力维度之一（具体基准分数见技术报告）。

**意义**：LongCat-Flash-Thinking 代表了"**大规模 MoE 推理旗舰把 agentic search 作为一等能力**"的趋势——DR/搜索能力不再是外挂的工程脚手架，而是旗舰模型在训练时（agentic mid-training + RL）就内化的核心维度。这与 Vision-DeepResearch"专门训练 > 强模型 + workflow"的结论一脉相承：把 DR 能力训进权重，比在通用模型上搭检索 pipeline 更稳。本专题作为旗舰模型对 DR 能力的支撑列出（非精读），与 GLM-5（BrowseComp 75.9）、Kimi K2.5（Agent Swarm）、Step 3.5 Flash（11B active）等同属"旗舰模型的 agentic 化"潮流——2026 H1 国产大厂集体把 agentic/DR 能力作为旗舰核心卖点。181↑。

### 2026-02（2 月）

#### #20 Vision-DeepResearch: Incentivizing DeepResearch Capability in Multimodal Large Language Models ⭐ [155↑] — arXiv 2601.22060

**问题动机**：现有多模态搜索假设"单张 full/entity-level 图 query + 少量文本 query"就够——这在**真实视觉噪声**下不现实，且推理深度/搜索广度受限。

**核心方法**：提出**多模态 deep-research 新范式**——执行 **multi-turn（多轮）× multi-entity（多实体）× multi-scale（多尺度）** 的视觉 + 文本搜索，在重噪声下稳健命中**真实搜索引擎**（Google/Bing 而非 sandbox API）；支持**数十步推理 + 数百次引擎交互**；通过 **cold-start 监督 + RL** 把 DR 能力**内化进 MLLM 权重**，无需复杂 prompt 模板。

**关键结果**：大幅超过现有多模态 DR MLLM，且**超过 GPT-5 / Gemini-2.5-pro / Claude-4-Sonnet 上搭的 workflow**——证明专门训练 > 强模型 + 工程脚手架。GitHub 643★。

**意义**：Vision-DeepResearch 把 deep research 从"文本黑客"推到"多模态系统"，证明在真实视觉噪声 + 真实搜索引擎下，专门训练的 MLLM 能超越 frontier 闭源 + workflow。已成精读 [[07-vision-deepresearch]]（与配套 VDR-Bench 合并），是 VideoDR 之后多模态 DR 的代表性 OSS 工作。

#### #26 Vision-DeepResearch Benchmark (VDR-Bench): Rethinking Visual and Textual Search for MLLMs ⭐(配套精读) [118↑] — arXiv 2602.02185

**问题动机**：诊断现有多模态搜索基准的**两大缺陷**——① **非视觉搜索中心**：答案常被文本里的 cross-textual cue 泄露，或可被 MLLM 先验知识直接推出，根本不需要真正的视觉搜索；② **评测场景过度理想化**：图搜索侧近似"全图精确匹配"即可，文搜索侧太直接，与真实条件脱节。

**核心方法**：**VDR-Bench**——2000 个 VQA 实例，经**多阶段策划 + 专家评审**，专门构造"不能被文本 cue 或先验知识泄露、必须真正做视觉搜索"的样本，模拟真实条件。配套 **multi-round cropped-search workflow**（多轮裁剪搜索工作流）提升视觉检索——把图像裁剪成关注区域逐轮搜索。

**关键结果**：在 VDR-Bench 上现有 MLLM 暴露出对视觉搜索的真实短板（远低于在被泄露的旧基准上的虚高分数）。

**意义**：VDR-Bench 与 Vision-DeepResearch 同源（Osilly 团队），合并精读 [[07-vision-deepresearch]]。它的核心贡献是揭示"旧多模态搜索基准答案泄露"——很多看似考视觉搜索的题其实被文本 cue 或先验知识泄露了，必须专门构造抗泄露样本才能真正衡量多模态 DR 能力。这与 VideoDR 的"双 ablation 保证跨模态必要性"是同一思路。

#### #30 WideSeek-R1: Exploring Width Scaling for Broad Information Seeking via Multi-Agent RL ⭐ [清华 / RLinf, 100↑] — arXiv 2602.04634

**问题动机**：LLM 能力扩展过去主要走 **depth scaling**（单 agent 多轮推理 + 工具）。但当任务变**广**（需同时覆盖很多并行子问题），瓶颈从"个体能力"转为"组织能力"。现有多 agent 系统多靠手工 workflow + 轮流交互，**无法有效并行**。

**核心方法**：提出 **width scaling（宽度扩展）**——用多 agent 系统处理 broad information seeking。**lead-agent-subagent 框架 + MARL**：共享 LLM 但 isolated context + 专用工具，在 **20k broad IR 任务**上**联合优化** lead agent 与并行 subagent，让模型学会真正的并行分工（而非 prompt 工程拼出来的多 agent）。

**关键结果**：WideSeek-R1-4B 在 WideSearch 上 item F1 **40.0%**，**匹敌单 agent DeepSeek-R1-671B**；并行 subagent 越多性能越好。GitHub 3.7k★。

**意义**：WideSeek-R1 提出与"深度"正交的"宽度"scaling 维度，证明 MARL 能训出真正的并行组织能力。已成精读 [[12-searchswarm-and-wideseek]]（与 SearchSwarm 合并：宽度 vs 委派），与 `agentic-rl/10` 同源。它和 MiroThinker（交互深度）、SearchSwarm（委派）共同构成 DR scaling 的三个新轴。

#### #41 InternAgent-1.5: A Unified Agentic Framework for Long-Horizon Autonomous Scientific Discovery ⭐ [Intern Science, 77↑] — arXiv 2602.08990

**问题动机**：科学发现需要跨**计算域与实证域（wet lab）**的端到端能力——从设计 ML 算法到执行湿实验——但缺乏一个能持续运转、跨域协调的统一系统。

**核心方法**：InternAgent-1.5 是端到端科学发现统一系统。架构是**三个协调子系统**——**generation（生成）/ verification（验证）/ evolution（演化）**——底层由**三项基础能力**支撑：**deep research（深度研究）、solution optimization（方案优化）、long-horizon memory（长程记忆）**。这一架构让系统能在延长的发现周期内**持续运转、保持连贯且不断改进的行为**，并在单一系统内协调计算建模与实验室实验。

**关键结果**：在 **GAIA / HLE / GPQA / FrontierScience** 上领先；能自主设计 ML 算法（algorithm discovery）、执行完整计算或 wet-lab 实验（empirical discovery，覆盖地球/生命/生物/物理域）。GitHub 1.3k★。57 位作者。

**意义**：InternAgent-1.5 把 deep research 作为科学发现统一系统的三大基础能力之一，展示了 DR 能力如何融入更大的"自治科学发现"框架。它与同机构 SGI-Bench（评测）形成"系统 + 评测"一对，代表 deep research 向科学发现深化、并与 wet-lab 实验闭环的前沿方向（列出，未单独精读）。

#### #15 Step 3.5 Flash: Open Frontier-Level Intelligence with 11B Active Parameters 📄 [StepFun, 200↑] — arXiv 2602.10604

**问题动机**：如何在**极少 active 参数**（11B）下达到 frontier-level 的 agentic 智能，兼顾性能与效率？大模型 agentic 能力强但推理成本高、延迟大，难以在成本/延迟敏感的真实 agent 部署中大规模使用——能否用稀疏 MoE 把"frontier 级 agentic 智能"压到极小的 active 参数？

**核心方法**：Step 3.5 Flash（StepFun）是**稀疏 MoE 模型**，仅 **11B active 参数**即达 frontier-level agentic 智能。设计哲学聚焦"构建 agent 时最重要的东西"——**sharp reasoning（敏锐推理）+ fast, reliable execution（快速可靠执行）**，即不追求知识广度，而追求"推理准、执行快、可靠"这三个 agent 落地最关键的属性。摘要将 **BrowseComp** 列为关键评测之一——模型具备 **deep search / web browsing** 能力，能完成多步检索类 DR 任务。采用 **off-policy RL** 训练——用离线收集的高质量轨迹做强化学习，提升训练效率与稳定性。

**关键结果**：11B active 参数达到 frontier-level agentic 表现，BrowseComp（DR 能力）为其核心评测之一（具体分数见技术报告，200↑ 高票）。

**意义**：Step 3.5 Flash 代表"**小 active 参数 MoE 也能做 frontier agentic 智能**"的效率路线——这对真实 agent 部署（需要低延迟、低成本地跑大量 agentic 工作流）极具价值，呼应趋势 方向8️⃣"让 agent 用得起"。BrowseComp（DR 能力）是其关键评测维度，说明即便是主打效率的小 active 参数旗舰，也把 deep search 作为必备能力。本专题作为旗舰模型对 DR 的支撑列出（非精读），与 LongCat（560B MoE）、GLM-5（744B MoE）同属"旗舰模型把 agentic/DR 作为核心能力"潮流，但 Step 3.5 Flash 更强调**参数效率**——用 11B active 达到大模型级 agentic 表现，是"小而强"路线的代表（与 OCC-RAG 的小 SLM 忠实推理形成"效率优先"的呼应）。

### 2026-03（3 月）

#### #8 MiroThinker-1.7 & H1: Towards Heavy-Duty Research Agents via Verification 🔗(已收录 `huggingface/12`) [MiroMind, 187↑] — arXiv 2603.15726

**问题动机**：单 agent 长程任务**不可靠**——长 context 下容易失稳、goal drift、前期假设与后期证据矛盾。如何让 research agent 在 50+ 步的"重型"任务上保持可靠？

**核心方法**：MiroThinker-1.7 用 **agentic mid-training** 阶段专门提升单步交互可靠性。H1 进一步**把 verification 嵌入推理过程**——① **Local verification**：每个中间决策点自我评估"这步合理吗"；② **Global verification**：整条证据链做一致性审计（避免前期假设与后期证据矛盾）。verification 不是事后过滤、不是 ensemble 投票，而是**嵌入推理过程本身**。

**关键结果**：在 open-web research / scientific reasoning / financial analysis 三大类长任务上达 SOTA，尤其在需 50+ 步推理的"重型"任务上稳定性显著优于竞品。开源 MiroThinker-1.7（30B）+ 1.7-mini（7B）权重 + 训练代码。

**意义**：MiroThinker H1 把"验证嵌入推理"确立为 long-horizon agent 的新范式，正面回应 VideoDR 揭示的 goal drift 问题。详见 `huggingface/12-mirothinker.md`，是 MiroThinker v1.0（[[01-mirothinker-v1]]）的可靠性升级版。

#### #20 OpenSeeker: Democratizing Frontier Search Agents by Fully Open-Sourcing Training Data ⭐ [上交 等, 150↑] — arXiv 2603.15594

**问题动机**：frontier-level search agent 被工业巨头垄断，根因是**高质量训练数据不透明**。如何**完全开源（模型 + 数据）**地复现 frontier 搜索能力？

**核心方法**：OpenSeeker 是**首个完全开源（模型 + 数据）** 的 frontier-level search agent，两大创新：① **Fact-grounded scalable controllable QA synthesis**——**逆向工程 web graph**（拓扑扩展 + 实体混淆），生成可控覆盖/复杂度的多跳任务（难度可调而非碰运气）；② **Denoised trajectory synthesis**——用 retrospective summarization（事后总结）机制**去噪轨迹**，促使 teacher LLM 产出更高质量的动作供 student 学习。

**关键结果**：仅 **11.7k 合成样本 + 单次 SFT** 即达多基准 SOTA；BrowseComp 上 **29.5%** vs 次优开源 DeepDive 15.3%；BrowseComp-ZH 上 **48.4% 超过 Tongyi DeepResearch 46.7%**（后者用了大量 continual pre-training + SFT + RL，OpenSeeker 仅 SFT）。GitHub 746★。

**意义**：OpenSeeker 用极少数据 + 纯 SFT 就在中文基准超过用了全套 CPT+SFT+RL 的 Tongyi DR——证明**数据质量 > 数据量、> 训练复杂度**。已成精读 [[05-openseeker]]，是"data efficiency 取代 model scale"开源叙事的标杆，与 OpenResearcher、Step-DeepResearch 同属开源 DR 训练范式主线。

#### #36 OpenResearcher: A Fully Open Pipeline for Long-Horizon Deep Research Trajectory Synthesis ⭐ [TIGER-Lab, 100↑] — arXiv 2603.20278

**问题动机**：现有数据采集 pipeline **依赖专有 web API**，大规模轨迹合成昂贵、不稳定、难复现——训练 DR agent 需要真实 web 调用，成本和不稳定性都极高。

**核心方法**：**解耦"一次性 corpus bootstrapping"与"多轮轨迹合成"**——在 **1500 万文档**的离线语料上，用三个显式 browser 原语（**search / open / find**，完全本地实现，模拟真实 web 交互但 100% 可复现）执行 search-and-browse loop。用 **GPT-OSS-120B 作 teacher** 合成 **97K+ 轨迹**（含 100+ tool call 的长程长尾任务）。

**关键结果**：30B-A3B backbone SFT 后 BrowseComp-Plus **54.8%**（比 base **+34.0pp**）——超过许多调用真实 web 的强基线。离线全可控环境支持受控分析（数据过滤策略、agent 配置、检索成功率与最终准确率的关系）。GitHub 777★。完整开源语料 + 轨迹 + 训练代码 + 权重。

**意义**：OpenResearcher 把"长程 DR 训练"的复现门槛从"需要 web API quota"降到"本地能跑"，是 OSS deep research 生态的关键基建。已成精读 [[06-openresearcher]]，与 OpenSeeker 互补（一个逆向 web graph 合成 QA、一个离线语料合成轨迹）。

#### #30 T2S-Bench & Structure-of-Thought 📄 [121↑] — arXiv 2603.03790

**问题动机**：text-to-structure（文本转结构化表示）推理是很多下游任务的底层能力——把自然语言描述转成结构化的表（table）、图（graph）、schema、计划等。但这一能力**缺乏系统的评测基准与有效的 prompting 技术**，模型在"先建结构再推理"上的表现没有被严格衡量过。对 deep research 而言，这恰是第一步的核心能力：DR agent 拿到一个模糊的开放式研究需求，必须先把它**分解为结构化的子查询、检索计划、证据收集 schema**，才能有序地多源检索——这一步做不好，后续再强的检索与合成也会失序。

**核心方法**：本文提出 **T2S-Bench**——一个 text-to-structure 推理基准，覆盖含科学领域在内的多种结构化任务，系统评估模型把自然语言转成结构化表示的能力（结构的正确性、完整性、可执行性）。配套提出 **Structure-of-Thought** prompting 技术——区别于平铺的 Chain-of-Thought，它**引导模型先显式构建一个结构骨架（schema/skeleton）再在其上逐步推理**，把"结构化"这一隐式步骤显式化为推理的第一阶段。这与 DR 的 **query planning（查询规划）** 子能力直接对应——DR agent 需要把研究需求转成结构化的查询/计划树，Structure-of-Thought 正是这一转换的通用方法。

**关键结果**：在 text-to-structure 任务上 Structure-of-Thought 一致优于平铺 prompting（具体数字见 PDF），尤其在需要先建复杂结构再填充内容的科学领域任务上增益明显。121↑。

**意义**：T2S-Bench 与 DR 的关联在于 **query planning**——deep research 的第一步是把开放式研究需求分解为结构化的子查询/计划，而 text-to-structure 推理正是这一步的底层能力。它揭示了一个常被忽视的环节：DR 系统往往把注意力放在"检索"和"合成"上，但"如何把模糊需求结构化"这一前置步骤同样决定成败。Structure-of-Thought 的"先建结构再推理"思路可直接用于强化 DR 的 planning 模块，与 `Deep Research: A Systematic Survey` 四组件框架中的 query planning 一环呼应。本专题作为 DR query planning 子能力的相关工作列出（非精读）。

#### #33 HopChain: Multi-Hop Data Synthesis for Generalizable Vision-Language Reasoning 📄 [Qwen, 110↑] — arXiv 2603.17024

**问题动机**：VLM 的**长链多跳推理**能力（需要跨多个视觉线索 + 文本线索逐跳推导才能得出答案）受限于训练数据——现有视觉-语言数据集多是单跳或浅层推理，**缺乏大规模、真正多跳的视觉-语言推理数据**。这与多模态 deep research 的瓶颈完全同源：多模态 DR 任务本质就是多跳的（视频/图像→识别实体→web 检索→交叉验证→合成答案），但训练这种能力的数据极其稀缺。

**核心方法**：HopChain（Qwen 出品）是一套**多跳数据合成**方法，自动生成多跳视觉-语言推理数据来提升 VLM 的长链推理能力。它把多个视觉-语言推理步骤**串成链（hop chain）**——每一跳依赖前一跳的结论，合成出需要跨多个视觉区域/文本线索**逐跳推理**才能解的训练样本（而非一眼可答的浅层题）。通过控制 hop 的数量与类型，可生成不同难度、不同推理结构的多跳样本，覆盖"先看图局部→再关联文本→再跨区域比对"这类组合推理模式。

**关键结果**：用 HopChain 合成数据训练后，VLM 在多跳视觉-语言推理基准上一致提升（具体数字见 PDF），长链推理（需 3+ 跳）的提升尤其显著。110↑。

**意义**：HopChain 与多模态 DR 的关联在于**数据合成**——Vision-DeepResearch、OpenSearch-VL、VideoDR 等多模态 DR 工作的核心瓶颈都是"多跳视觉-文本推理数据稀缺"，而 HopChain 的"把推理步骤串成 hop chain 合成训练数据"思路可直接迁移到多模态 DR 的训练数据生产。它与 OpenSearch-VL 的 Wikipedia path sampling（沿知识图谱路径生成多跳问题）异曲同工——都在解决"如何大规模合成需要多跳推理的训练样本"。本专题作为多模态 DR 数据合成的相关工作列出（非精读）。

### 2026-04（4 月）

> 本月 top50 中**无新增的纯 deep-research 论文**——候选项（Agentic World Modeling [已收录 `huggingface/15`]、GameWorld、OpenGame、Terminal Agents、AgentSPEX）经审查均为 world model / 游戏 agent / 企业自动化 / agent DSL，不属于 DR 范畴，故本月不收录。这与 2026 H1 "文本 DR 已卷得差不多、4 月主题转向 world model 与 coding agent"的趋势一致（详见 `agentic-rl/00-summary` 2026-04 与 `huggingface/00-summary` W17-18 的相同观察）。

### 2026-05（5 月）

#### #33 Beyond Semantic Similarity: Rethinking Retrieval for Agentic Search via Direct Corpus Interaction ⭐ [TIGER-Lab, 123↑] — arXiv 2605.05242

**问题动机**：传统 retriever（lexical/semantic）把语料压缩成"固定相似度接口 + 单步 top-k"，对 agentic search 成了**瓶颈**——精确词约束、稀疏线索合取、局部上下文检查、多步假设修正都难以用现成 retriever 实现，且**早期被滤掉的证据无法被下游更强推理找回**。

**核心方法**：**DCI（Direct Corpus Interaction，直接语料交互）**——agent 直接用通用终端工具（**grep、文件读取、shell、轻量脚本**）搜索原始语料，**无需 embedding 模型 / 向量索引 / 检索 API**；无需离线索引，天然适配演化中的本地语料。本质是把"检索"从"固定相似度接口"还原为"agent 用 shell 自由探索原始语料"。

**关键结果**：在 BRIGHT / BEIR 多个数据集大幅超过 sparse/dense/reranking 强基线；BrowseComp-Plus 与多跳 QA 不依赖任何语义 retriever 即达高准确率。提出"**检索质量取决于交互接口的分辨率**"这一新视角。GitHub 340★。19 位作者（含 Yejin Choi、James Zou、Jiawei Han、Jimmy Lin 等 IR/NLP 重磅人物）。

**意义**：可能是 2026 H1 最具颠覆性的检索方向论文——质疑了过去 5 年 embedding/vector DB 整个范式。已成精读 [[10-dci-and-grepseek]]（与 GrepSeek 合并：DCI 范式），与 huggingface/20 "Code as Agent Harness"同一思想脉络（代码/shell 作为 agent 的 operational substrate）。

#### #44 OpenSearch-VL: An Open Recipe for Frontier Multimodal Search Agents ⭐ [腾讯混元, 102↑] — arXiv 2605.05185

**问题动机**：top-tier 多模态搜索 agent 仍难以复现——缺乏完全开源的训练 recipe（数据 + 算法 + 权重）。

**核心方法**：完全开源的 frontier 多模态 deep search agent 训练 recipe（agentic RL）。① **数据 pipeline**——Wikipedia path sampling（按知识图谱路径生成多跳问题）+ fuzzy entity rewriting（模糊实体改写降低表面匹配作弊）+ source-anchor visual grounding（源锚视觉定位减少 shortcut），联合降低 shortcut 与 one-step retrieval collapse；产出 SearchVL-SFT-36k + SearchVL-RL-8k。② **工具环境**——统一 text search / image search / OCR / cropping / sharpening / super-resolution / perspective correction，让 agent 结合主动感知与外部知识。③ **算法——multi-turn fatal-aware GRPO**：mask 失败后 token（避免错误污染后续）+ one-sided advantage clamping（保留失败前有用推理）。

**关键结果**：7 个基准平均 **+10pt+**，多任务匹敌商用模型。GitHub 215★。

**意义**：OpenSearch-VL 把"开源多模态 DR"整套 pipeline 公开——数据、算法、模型权重全部开源。已成精读 [[08-opensearch-vl]]，是 vision-DR 方向 OSS 路线的代表，与 Vision-DeepResearch（专门训练范式）互补。其 fatal-aware GRPO 亦见于 `agentic-rl/00-summary`。

### 2026-06（6 月）

#### #12 GrepSeek: Training Search Agents for Direct Corpus Interaction ⭐ [UMass Amherst, 106↑] — arXiv 2605.29307

**问题动机**：把 DCI（Direct Corpus Interaction）概念从"论点"变成"可训练、可部署的系统"——如何用 RL 训练 agent 直接通过 shell 命令与语料交互？

**核心方法**：与 retriever 互补——search agent 把语料本身当搜索环境，用可执行 shell 命令（grep/find/sed/awk）找证据。**两阶段训练**：① **cold-start 数据集**——用 **answer-aware Tutor（知答案的导师，解释 shell 用法）+ answer-blind Planner（不知答案的规划器，拆解检索意图）** 生成 verified、因果 grounded 的搜索轨迹；② **GRPO** refine policy（以"找到正确答案 + 用了多少 shell 命令"为 reward）。**工程贡献**：**semantics-preserving sharded-parallel execution engine（语义保持的分片并行执行引擎）**——把 shell 检索加速**最多 7.6×**，且 byte-exact 等价于顺序执行。

**关键结果**：7 个开放域 QA 基准上 token-level F1 与 EM 最强；也指出纯 lexical 交互在 surface-form 变化大的 query 上的局限。GitHub 42★。

**意义**：GrepSeek 把 DCI 从论点变成可训练、可部署的系统，预示 retrieval 范式从"embed + ANN"全面转向"agent + shell"的可能。已成精读 [[10-dci-and-grepseek]]（与 Beyond Semantic Similarity 合并），与 `agentic-rl/00-summary` 的 GrepSeek 条目交叉。

#### #16 OCC-RAG: Optimal Cognitive Core for Faithful Question Answering 📄 [89↑] — arXiv 2606.00683

**问题动机**：很多实际应用更受益于 **robust reasoning（鲁棒推理）** 而非海量参数知识——一个能严格依据给定上下文做忠实推理的小模型，往往比一个塞满世界知识但容易"自由发挥/幻觉"的大模型更可靠。因此 task-specialized **SLM（小语言模型）** 是合理的设计选择，而非一味堆大模型。这对 DR/RAG 系统尤其重要——DR 的答案必须 grounded 在检索到的证据上、可溯源、不能凭参数记忆编造，而这恰恰是"忠实推理"而非"知识广度"的问题。

**核心方法**：提出 **OCC（Optimal Cognitive Core）** 模型家族，其中 **OCC-RAG** 专为 context-grounded faithful QA 优化——要求对供给的段落做**多跳推理**、同时**忽略记忆知识**（只信 context、不靠参数里记的，避免"记忆污染"）。配套合成 **300 万+ 多上下文多跳 QA 语料**，三个训练目标并重：**多跳推理**（跨多个段落逐跳推导）+ **严格 context 忠实度**（答案每一步都有 context 支撑）+ **校准 abstention（拒答）**（证据不足时主动说"无法回答"而非瞎猜）。发布 **OCC-RAG-0.6B / 1.7B** 两档小模型，产出带 **source citation（逐字引用）** 的结构化推理 trace——每个结论都标注来自上下文的哪一句。

**关键结果**：在 HotpotQA / MuSiQue / TAT-QA（多跳推理）、ConFiQA（忠实度）、MuSiQue-Un（拒答）上**匹敌或超过 2–6× 大的通用模型**——即 0.6B/1.7B 的专用 SLM 在 context 忠实任务上能打过数倍大的通用 LLM。

**意义**：OCC-RAG 证明"小而专"的 SLM 在 context-grounded faithful QA 上可超过数倍大的通用模型——为 DR/RAG 系统提供了"用小模型做忠实推理核心"的设计选择，对成本敏感的生产部署尤其有价值（方向8️⃣低成本）。它的"严格 context 忠实 + 逐字引用 + 校准拒答"三件套，正是 DR 系统**可信答案生成**所需的核心能力——DR 的最大风险之一是编造数字/虚假引用（参见 DeepResearchEval 的 Active Fact-Checking、DRIFT 的 claim 审计），OCC-RAG 从模型训练层面给出了"只信证据、可溯源、敢拒答"的解法。本专题作为 RAG/多跳的 DR 邻接工作列出（非精读）。

#### #22 FORT-Searcher: Synthesizing Shortcut-Resistant Search Tasks for Training Deep Search Agents ⭐ [RUC AIBox, 68↑] — arXiv 2606.12087

**问题动机**：训练 deep search agent 需要"证据充分前答案不可得"的 verifiable 问题；现有方法靠丰富 graph 结构提难度，但**结构复杂 ≠ 真实搜索难度**——可能被更便宜的 identifying route 抄近道（structure complex 但被 shortcut 绕过）。

**核心方法**：**shortcut-aware difficulty framework（捷径感知难度框架）**——识别四类 shortcut 风险：**evidence co-coverage（证据共覆盖）、single-clue selectivity（单线索选择性）、exposed constants（暴露常数）、prior-knowledge binding（先验知识绑定）**，用 **trajectory signature**（solving cost 求解成本、answer hit time 答案命中时间、prior-shortcut rate 先验捷径率）诊断真实难度。**FORT** 在实体选择 / 证据图构造 / 问题表述 / 对抗 refinement 四环节控制 shortcut 风险。

**关键结果**：FORT 数据比现有开源 deep search 数据集诱导**更长的 pre-answer 搜索、更少 shortcut**；仅 SFT 训出的 FORT-Searcher 在挑战性基准上同级最佳。

**意义**：FORT 把"难度"从"图结构复杂度"重新定义为"**realized search difficulty（实测搜索难度）**"——必须用 trajectory signature 实测、并主动消除四类捷径。已成精读 [[09-harness-1]]（与 Harness-1、Masking 合并：DR 训练范式与状态管理），是"训练数据反 shortcut"趋势的代表。

#### #28 Masking Stale Observations Helps Search Agents — Until It Doesn't: A Regime Map and Its Mechanism ⭐ [McAuley-Lab, 62↑] — arXiv 2606.00408

**问题动机**：长程 search agent 跨多次 tool call 积累大量检索内容，context 预算效率重要。最小干预是 **mask 掉过期观察（stale observations）**，但**何时有用、为何有用不清楚**。

**核心方法**：**系统实验**——扫 4B–284B backbone × 3 个 retriever × 离线/实时 web 基准。**核心发现**：masking 的准确率增益（相对"无 context 管理时的准确率"）呈**非对称的倒 U 形 regime**——弱 retriever 下平台（没用）、强 retriever 配中等模型时峰值（增益最大）、模型饱和时骤降（强模型自己会忽略噪声，硬删反而误伤）。**机制**：masking 是 **token-for-turn 权衡**——移除模型已基本停止 attend 的观测、换取更多 turn；增益取决于 **retriever recall 与模型 implicit filtering capacity 的交互**。

**关键结果**：把 context 管理重新定义为 **regime-dependent** 干预——没有银弹。GitHub 16★。

**意义**：这篇是对所有"激进 context 压缩"工作的重要反思——证明 context 管理策略必须匹配 (retriever recall × model filtering capacity) 的具体 regime。已成精读 [[09-harness-1]]（与 FORT、Harness-1 合并：DR 训练/上下文的"踩坑与机制"），亦是 `memory/02-masking` 的核心。

#### #38 K-BrowseComp: A Web Browsing Agent Benchmark Grounded in Korean Contexts ⭐ [CMU, 56↑] — arXiv 2606.02404

**问题动机**：web-browsing agent 评测几乎全是英文/中文，**韩语语境的 agentic 评测空白**——韩语 web 的检索难度、韩语 LLM 的真实能力都缺乏严格衡量。

**核心方法**：**韩语语境的 web-browsing agent 基准（400 题）**。构造：**300 题 K-BrowseComp-Verified** 由母语者手工构造验证；**100 题合成 split** 用 hard few-shot exemplar + failure-mode-targeted generation，利用"解题 vs 出题"的不对称性做对抗过滤（出题比解题容易，故能造出模型答不出但可验证的题）。

**关键结果**：GPT-5.5 / DeepSeek-V4-Pro / GLM-5.1 仅 **30.0–45.67%**（远低于 BrowseComp）；韩国 PAF 项目的韩语 LLM 仅 **0.0–10.33%**；对抗合成 split 上最强模型仅 **26.0%**。

**意义**：K-BrowseComp 暴露了韩语语境下 web-browsing agent 的巨大差距（尤其韩语本土 LLM 仅 0–10%），填补多语言 agentic 评测空白。已成精读 [[13-dr-eval-and-error]]（与 DeepResearchEval、Span-Level Error 合并：DR 评测），是"评测向多语言/真实环境扩张"趋势的代表。

#### #40 Where Do Deep-Research Agents Go Wrong? Span-Level Error Localization in Agent Trajectories ⭐ [NJU-LINK, 54↑] — arXiv 2606.02060

**问题动机**：基于最终答案的评测只能看"成没成功"，看不出"**轨迹哪部分让答案不可靠**"——DR agent 的长轨迹里到底是哪一步开始错的？

**核心方法**：收集 2 个 agent 框架 × 3 个 backbone × 3 个基准的 **2,790 条真实轨迹**，转成 semantic span，LLM-assisted 专家标注 harmful error span，建 **TELBench（1000 实例）**。提出 **DRIFT**——**claim-centric 审计框架**：追踪 agent 的 claim、核查其在轨迹证据中的支撑、标记影响答案路径的 unsupported/conflicting span（无支撑/冲突的片段）。

**关键结果**：DRIFT 把 span-level error localization 与 first-error accuracy 提升**最多 30pp**，提供 DR agent 可靠性的 **process-level（过程级）** 视角。GitHub 21★。

**意义**：DRIFT 把 DR 评测从"最终答案对错"推进到"span 级错误定位 + 主动事实核查"，回答"哪一步开始错"。已成精读 [[13-dr-eval-and-error]]，是"评测从答案正确率走向过程可信度"趋势的代表，与 `memory/` 的 EvoMem（记录演化）、HaluMem（操作级幻觉定位）精神相通。

#### #45 Harness-1: Reinforcement Learning for Search Agents with State-Externalizing Harnesses ⭐ [Chroma / UIUC, 52↑] — arXiv 2606.02373

**问题动机**：把 search agent 训成"在不断增长的 transcript 上的 policy"，让 RL **同时优化语义搜索决策与可恢复的 bookkeeping**——后者环境能更可靠地维护，不该由 policy 学。

**核心方法**：**Harness-1**——20B 检索 subagent，在 **stateful search harness** 内用 RL 训练。harness 维护**环境侧 working memory**：candidate pool、importance-tagged curated set、compact evidence links、verification records、压缩去重的 observation、budget-aware context rendering；policy 只保留**语义决策**（搜什么、留/弃哪些文档、验证什么、何时停）。

**关键结果**：8 个检索基准（web/finance/patents/multi-hop QA）平均 curated recall **0.730**，比次优开源 subagent **+11.4pt**，在 **held-out transfer 上增益尤强**（语义决策比记账更可迁移）。GitHub 548★。

**意义**：Harness-1 给出可操作的设计原则——**把"可恢复的状态"外置给环境，只让 RL 优化"不可恢复的语义决策"**。已成精读 [[09-harness-1]]，亦是 `memory/01-harness-1`（working memory 表示侧标杆）与 `agentic-rl/13-harness-1`（harness×RL 范式）的核心，是"state externalization"主线的代表。

#### #48 SearchSwarm: Towards Delegation Intelligence in Agentic LLMs for Long-Horizon Deep Research ⭐ [SearchSwarm, 49↑] — arXiv 2606.09730

**问题动机**：长程 deep research 的上下文需求可无界增长，但模型上下文窗口有限。如何让 agent 通过"委派"分摊上下文压力？

**核心方法**：核心能力是 **delegation intelligence（委派智能）**——main agent **分解任务、决定何时委派什么给 subagent、整合 subagent 返回的摘要结果**，以节省主上下文预算。设计 **harness 引导模型做高质量任务分解与委派**，并约束 subagent 正确返回结果；harness 引导的轨迹**天然编码正确委派决策**，用作 SFT 数据把 delegation intelligence **内化进权重**。

**关键结果**：SearchSwarm-30B-A3B 在 **BrowseComp 68.1 / BrowseComp-ZH 73.3**，同级最佳。GitHub 63★。

**意义**：SearchSwarm 把"何时委派什么给 subagent"作为可训练能力（delegation intelligence），是 DR scaling 第三轴"委派"的代表。已成精读 [[12-searchswarm-and-wideseek]]（与 WideSeek-R1 合并：委派 vs 宽度），与 MiroThinker（交互深度）、WideSeek-R1（宽度）共同构成 DR scaling 的三个新轴。

#### #43 WeaveBench: A Long-Horizon, Real-World Benchmark for Computer-Use Agents with Hybrid Interfaces 📄 [Microsoft, 52↑] — arXiv 2606.09426

**问题动机**：computer-use agent（CUA）越来越多地在**混合 runtime**（视觉桌面控制 + 命令行 + 代码编辑 + 浏览器 + 外部工具）中运作，但现有基准多把这些接口当**可分离的能力**单独评测，**长程跨接口编排**严重缺测。

**核心方法**：**WeaveBench**——长程 hybrid-interface 基准，**114 任务 × 8 真实工作域**，基于真实用户请求 + 可公开验证的产物。每个任务要求 agent 在**单条轨迹内**结合 GUI 观察/动作与 CLI/code 操作。在**真实 Ubuntu 桌面 + 部署的 CLI-agent runtime**（加最小桌面控制插件）上评测。配套 **trajectory-aware judge（轨迹感知评判器）**——检查 deliverables/files/screenshots/logs/action traces，并检测 fabricated visual evidence（编造视觉证据）/ hard-coded metrics（硬编码指标）等 shortcut。

**关键结果**：最佳 PassRate 仅 **41.2%**（远未饱和）；trajectory-aware judge 揭示 **outcome-only grading 大幅高估 agent 性能**。

**意义**：WeaveBench 暴露了 CUA 评测的关键空白——长程跨 GUI/CLI/code 接口编排，并证明"只看最终结果"会高估性能（agent 会用编造证据/硬编码指标抄近道）。本专题作为 computer-use 的 DR 邻接工作列出（非精读），其"轨迹感知评判"与 DRIFT 的"过程级审计"精神一致。

## 二、统计速览

| 月份           | 月榜论文总数 | top50 内 DR 命中 | 本专题收录                | 精读                |
| -------------- | ------------ | ---------------- | ------------------------- | ------------------- |
| 2025-11        | 105          | 5                | 5                         | 4 ⭐                |
| 2025-12        | 105          | 3                | 3                         | 2 ⭐ + 1 综述       |
| 2026-01        | 105          | 4(含2已收录)     | 2 新                      | 1 ⭐                |
| 2026-02        | 105          | 4                | 4                         | 4 ⭐                |
| 2026-03        | 105          | 2 新(+1已收录)   | 2 新                      | 2 ⭐                |
| 2026-04        | 105          | 0 真 DR          | 0                         | —                  |
| 2026-05        | 105          | 2                | 2                         | 2 ⭐                |
| 2026-06        | 105          | 9                | 9                         | 7 ⭐                |
| **合计** | —           | —               | **27** 篇（新收录） | **13** 篇精读 |

> 注：2026-06 是 DR 论文密度最高的月份（9 篇），与"搜索 agent 训练方法学（DCI、shortcut-resistant、state-externalizing、delegation）+ 过程级评测"在 2026 H1 末集中爆发吻合。

---

## 三、精读论文一览（13 篇，按主题）

| 编号         | 论文                                                         | 月/票       | 机构              | 主题                                     |
| ------------ | ------------------------------------------------------------ | ----------- | ----------------- | ---------------------------------------- |
| **01** | [MiroThinker v1.0](01-mirothinker-v1.md)                        | 11/196↑    | MiroMind          | Interaction scaling 第三维度             |
| **02** | [IterResearch](02-iterresearch.md)                              | 11/80↑     | —                | Markovian state，反 context 窒息         |
| **03** | [DR Tulu](03-dr-tulu.md)                                        | 11/63↑     | RL ReSearch       | RLER 演化 rubric，long-form              |
| **04** | [Step-DeepResearch](04-step-deepresearch.md)                    | 12/88↑     | StepFun           | Atomic capability 数据合成               |
| **05** | [OpenSeeker](05-openseeker.md)                                  | 03/150↑    | OpenSeeker        | 全开源数据，逆向 web graph               |
| **06** | [OpenResearcher](06-openresearcher.md)                          | 03/100↑    | TIGER-Lab         | 离线 corpus + 97K 轨迹合成               |
| **07** | [Vision-DeepResearch (+VDR-Bench)](07-vision-deepresearch.md)   | 02/155↑    | —                | 多模态 DR + 配套基准                     |
| **08** | [OpenSearch-VL](08-opensearch-vl.md)                            | 05/102↑    | 腾讯混元          | 多模态搜索 fatal-aware GRPO              |
| **09** | [Harness-1 (+FORT+Masking)](09-harness-1.md)                    | 06/52↑     | Chroma            | State externalization & 训练踩坑         |
| **10** | [DCI &amp; GrepSeek](10-dci-and-grepseek.md)                    | 05-06/123↑ | TIGER-Lab/UMass   | 直接语料交互范式                         |
| **11** | [General Agentic Memory](11-general-agentic-memory.md)          | 11/171↑    | BAAI              | JIT 记忆 = deep research                 |
| **12** | [SearchSwarm &amp; WideSeek-R1](12-searchswarm-and-wideseek.md) | 06/02       | SearchSwarm/RLinf | 委派智能 vs 宽度扩展                     |
| **13** | [DR 评测三件套](13-dr-eval-and-error.md)                        | 01-06       | Infinity/NJU/CMU  | DeepResearchEval+Span-Error+K-BrowseComp |

---

## 四、与 `huggingface/` 库的关系

本专题是 `huggingface/` 库（每周 top10，覆盖全 agent 主题）的**深度补充**：

- **不重复**：VideoDR、Youtu-Agent、MiroThinker-1.7&H1、ARIS（已在 huggingface/04/05/12/07）不再精读，仅交叉引用
- **更专注**：仅聚焦 deep research / search agent 一条主线，覆盖每月 **top50**（比 huggingface 的 top10 更深），因此挖出大量 huggingface 库未触及的 DR 专门工作（OpenSeeker、OpenResearcher、DR Tulu、IterResearch、Harness-1、DCI、SearchSwarm 等）
- **互补阅读**：先读 `huggingface/README.md` 建立全景 → 读本专题 README 与精读建立 DR 一条线的纵深理解

---

## 五、研究发展现状与方向趋势（2025-11 → 2026-06）

> 本节基于上述 27 篇论文 + `Deep Research: A Systematic Survey`（arXiv 2512.02038）的框架综合而成。该综述把 DR 系统形式化为 **query planning → information acquisition → memory management → answer generation** 四大组件，并区分 DR 与 single-shot prompting / 标准 RAG（DR 强调 critical thinking、multi-source、verifiable output）。下面的现状与趋势即沿这条主线展开。

### 现状：DR 已从"能不能做"进入"如何低成本做对、做稳、做可信"

这 8 个月的论文密度本身就说明问题——2025-11 已有 5 篇进入月榜 top50，2026-06 更达 9 篇。整个领域的关注点已经不在"LLM + 搜索引擎能否完成研究任务"（这一点 2025 上半年已被 OpenAI/Gemini DeepResearch、Tongyi DeepResearch 证明），而是转向四个更精细的问题：

1. **如何用更小的模型、更少的数据追平闭源？**——OpenSeeker 用 11.7k 样本 + 单次 SFT 就在 BrowseComp-ZH 超过 Tongyi DeepResearch；Step-DeepResearch 32B 拿 61.4% Research Rubrics；DR Tulu-8B 匹敌商用系统。"data efficiency" 取代 "model scale" 成为开源叙事核心。
2. **如何让长程轨迹不崩？**——IterResearch 诊断 "context suffocation"，MiroThinker 提出 "interaction scaling"，Harness-1 把 state 外置到环境，都在解决"轨迹一长就乱"的同一个病。
3. **如何防止训练数据被"抄近道"？**——FORT-Searcher 形式化四类 shortcut 风险，OpenSearch-VL/OpenSeeker 在数据合成阶段就主动 anti-shortcut。"难度"被重新定义为"realized search difficulty"而非"图结构复杂度"。
4. **如何知道 agent 到底错在哪？**——Where-Do-DR-Go-Wrong（DRIFT/TELBench）、DeepResearchEval、K-BrowseComp 把评测从"最终答案对错"推进到"span 级错误定位 + 主动事实核查"。

### 七大趋势

#### 1️⃣ Interaction / Width / Delegation —— scaling 的三个新轴

继 model size、context length 之后，社区在 2025 H2-2026 H1 集中提出了**三条互补的 scaling 轴**：

- **Interaction scaling（交互深度）**：MiroThinker 把"agent-environment 交互次数"训成可预测的 scaling law（600 tool calls / task）；IterResearch 把交互推到 2048 次（3.5%→42.5%）。
- **Width scaling（宽度/并行）**：WideSeek-R1 用 lead-subagent + MARL 做"broad information seeking"，4B 匹敌 671B。
- **Delegation intelligence（委派）**：SearchSwarm 把"何时委派什么给 subagent"作为可训练能力，30B 在 BrowseComp 拿 68.1。

> 三者共同回答："单 agent 单上下文已经不够，下一步靠交互深度、并行宽度、还是委派结构？"——目前看是**三者并存、按任务选型**。

#### 2️⃣ 训练数据合成成为真正的胜负手（且开始"反 shortcut"）

几乎每一篇开源 DR 工作的核心贡献都落在**数据合成 pipeline**而非模型架构：

- OpenSeeker：逆向 web graph（拓扑扩展 + 实体混淆）做可控多跳 QA。
- OpenResearcher：离线 1500 万文档语料 + GPT-OSS-120B teacher 合成 97K 轨迹（含 100+ tool call 长尾）。
- FORT-Searcher：**shortcut-aware** 难度框架——证明"结构复杂 ≠ 真难"，必须主动消除 evidence co-coverage / single-clue / exposed constants / prior-knowledge binding 四类捷径。
- Step-DeepResearch：基于 atomic capability 反推合成数据。

> 2026 H1 的新认知：**合成数据的"难度"必须用 trajectory signature（solving cost、answer hit time）实测，而非凭 graph 结构臆断**。

#### 3️⃣ State Externalization —— 把 bookkeeping 从 policy 里搬出去

长程 agent 的一个范式转变：**不要让 RL 同时学"搜什么"和"记什么"**。

- Harness-1：环境侧维护 candidate pool / curated set / evidence links / verification records，policy 只做语义决策——held-out transfer 增益尤强（说明这种分离能泛化）。
- IterResearch：evolving report 作为外置记忆，周期性合成。
- General Agentic Memory：JIT 式 page-store + memorizer/researcher 分离。

> 共同信号：**context window 不是用来"装下一切"的，而是要主动 curate**；环境比 policy 更适合做可恢复的状态管理。

#### 4️⃣ Context 管理被发现是"regime-dependent"，没有银弹

Masking Stale Observations 这篇做了反直觉的系统性结论：**mask 过期观察的增益相对模型能力呈非对称倒 U 形**——弱 retriever 下没用、强 retriever 配中等模型时峰值、模型饱和时反而有害。这给前面所有"激进 context 压缩"的工作打了一个重要补丁：**context 管理策略必须匹配 (retriever recall × model filtering capacity) 的具体 regime**，不能无脑套用。

#### 5️⃣ 检索接口本身成为研究对象：DCI（Direct Corpus Interaction）

Beyond Semantic Similarity 与 GrepSeek 提出一个尖锐观点：**固定的 top-k 相似度接口对 agentic search 是瓶颈**——精确词约束、稀疏线索合取、被早期滤掉的证据都无法被下游推理找回。解法是让 agent 直接用 **grep / shell / 文件读取**与原始语料交互（无 embedding、无向量索引），在 BRIGHT/BEIR/BrowseComp-Plus 上超过 sparse/dense/reranking 基线。

> 新视角：**"检索质量取决于交互接口的分辨率"**——这与 huggingface/20 的 "Code as Agent Harness" 是同一思想脉络（代码/shell 作为 agent 的 operational substrate）。

#### 6️⃣ 多模态 DR 成为新蓝海（文本 DR 已趋成熟）

文本 DR 的 SOTA 竞争已相当拥挤，2026 H1 明显向多模态扩张：

- VideoDR（视频，huggingface/04）→ Vision-DeepResearch + VDR-Bench（图像 multi-turn/multi-entity/multi-scale 搜索）→ OpenSearch-VL（统一 OCR/crop/super-res 工具环境）→ GeoVista（地理定位）。
- 共同痛点：**真实视觉噪声下的检索**（near-exact 匹配的理想化假设被打破）+ **答案泄露问题**（VDR-Bench 专门构造"不能被文本 cue 或先验知识泄露"的样本）。

#### 7️⃣ 评测从"答案正确率"走向"过程可信度 + 抗饱和 + 多语言"

- **过程级**：DRIFT/TELBench 做 span-level 错误定位（claim-centric 审计），把"哪一步开始错"量化（+30pp first-error accuracy）。
- **抗饱和 + 自动化**：DeepResearchEval 用 persona-driven 自动构造 + active fact-checking，避免 benchmark 被新模型快速刷爆。
- **多语言/真实环境**：K-BrowseComp 暴露韩语语境下的巨大差距（韩语 LLM 仅 0-10%）；WeaveBench 把 GUI+CLI+code 混合接口的长程编排纳入评测（最佳 PassRate 仅 41.2%）。

> 评测趋势与 huggingface/19（Agents' Last Exam，经济价值未饱和）一脉相承：**"最终答案对"已不足以衡量 DR agent，过程、抗饱和、真实部署成为新前线**。

### 仍待解决的开放问题

1. **长程一致性的根因**仍未根治——interaction scaling / state externalization 都是"缓解"而非"消除"goal drift。
2. **多模态检索的视觉感知瓶颈**——VideoDR 已指出 categorical/numerical error 是持续短板，OpenSearch-VL/Vision-DR 用工具增强缓解但未根治。
3. **评测的"标准答案"主观性**——long-form DR 的 rubric 即使 co-evolve（DR Tulu）或 adaptive（DeepResearchEval），仍依赖 LLM judge，存在循环依赖。
4. **DCI 的 lexical 局限**——GrepSeek 自己指出纯词法交互在 surface-form 变化大的 query 上会失效，如何与语义检索融合是开放问题。
5. **真实经济价值落地**——WeaveBench / Agents' Last Exam 显示，离"agent 能独立完成有经济价值的研究工作"仍有显著距离。

### 一句话总结

> **2025-11 → 2026-06 的 deep research 研究，主线是"把开源 DR 从昂贵的工业特权变成可复现、可审计、低成本的公共能力"**——数据合成（OpenSeeker/OpenResearcher/FORT）、训练范式（interaction/width/delegation scaling）、状态管理（Harness-1/IterResearch）、检索接口（DCI）、多模态扩张（Vision-DR/OpenSearch-VL）、过程级评测（DRIFT/DeepResearchEval）六条战线齐头并进，而下一阶段的核心矛盾，是**长程一致性的根因消除**与**真实经济价值的可靠落地**。
