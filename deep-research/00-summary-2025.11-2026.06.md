# Deep Research 专题 — HuggingFace 月榜 (2025-11 至 2026-06)

> **数据范围**：HuggingFace Papers **月榜** 2025-11、2025-12、2026-01、2026-02、2026-03、2026-04、2026-05、2026-06（共 8 个月）
> **筛选范围**：每月 **top 50** 中，标题/摘要/关键词明确涉及 **Deep Research / Search Agent / Web Agent / Agentic Search / 信息检索 Agent / 多跳研究** 的论文
> **去重**：与 `huggingface/` 目录已收录的论文（VideoDR、Youtu-Agent、MiroThinker-1.7&H1 等）不重复收录，仅做交叉引用
> **标记**：⭐ = 本专题精读论文（共 13 篇）｜📄 = 基于 arXiv 摘要的简短描述
> **目录定位**：`Q:/awesome-agent-paper/deep-research/`

---

## 一、按月列出（所有摘要均基于 arXiv 原文）

### 2025-11（11 月）

- **#4 MiroThinker: Pushing the Performance Boundaries of Open-Source Research Agents via Model, Context, and Interactive Scaling** ⭐ [MiroMind, 196↑] — arXiv 2511.11793
  - **核心主张**：在 model size、context length 之外，提出第三个 scaling 维度——**interaction scaling（交互规模）**：系统训练模型处理更深、更频繁的 agent-environment 交互。
  - **与 test-time scaling 的区别**：纯推理链拉长会退化（孤立运行、误差累积），而 interactive scaling 靠环境反馈 + 外部信息获取来纠错、refine trajectory。
  - **关键数字**：256K 上下文窗口下单任务可执行 **最多 600 次 tool call**；72B 变体在 GAIA 81.9% / HLE 37.7% / BrowseComp 47.1% / BrowseComp-ZH 55.6%，超过此前所有开源 agent，逼近 GPT-5-high。
  - **意义**：把"交互深度"确立为可预测的 scaling law，是开源 research agent 的奠基性工作。GitHub 8.2k★。已成精读 [[01-mirothinker-v1]]。
- **#5 General Agentic Memory Via Deep Research** ⭐ [BAAI, 171↑] — arXiv 2511.18423
  - **问题**：广泛采用的 static memory（提前构建好"即取即用"的记忆）必然带来严重信息损失。
  - **方法**——借用编译器的 **JIT（Just-In-Time）编译原则** 的 duo-design：① **Memorizer**——用轻量 memory 标注关键历史信息，同时把完整历史保存在 universal page-store；② **Researcher**——运行时根据在线请求，在预构建 memory 引导下从 page-store 检索并整合信息（即"用 deep research 的方式做记忆检索"）。
  - **优势**：充分利用 frontier LLM 的 agentic 能力与 test-time 可扩展性，且可端到端 RL 优化。GitHub 855★。已成精读 [[11-general-agentic-memory]]。
- **#36 IterResearch: Rethinking Long-Horizon Agents via Markovian State Reconstruction** ⭐ [80↑] — arXiv 2511.07327
  - **问题诊断**：现有 deep-research agent 用 **mono-contextual 范式**（所有信息堆进单一膨胀的上下文窗口），导致 **context suffocation（上下文窒息）** 与 **noise contamination（噪声污染）**。
  - **方法**：把长程研究重构为 **MDP + 策略性工作区重建**——维护一个 evolving report 作为记忆，周期性合成洞见，在任意探索深度下保持稳定的推理容量。配套 **EAPO（Efficiency-Aware Policy Optimization）**：几何奖励折扣激励高效探索 + 自适应下采样实现稳定分布式训练。
  - **关键数字**：6 个基准平均 **+14.5pp**；交互扩展到 **2048 次**时性能从 3.5% 飙到 42.5%；作为 prompting 策略可让 frontier 模型在长程任务上比 ReAct **+19.2pp**。已成精读 [[02-iterresearch]]。
- **#24 GeoVista: Web-Augmented Agentic Visual Reasoning for Geolocalization** 📄 [Tencent Hunyuan, 98↑] — arXiv 2511.15705
  - **任务**：把地理定位重新表述为需要"视觉 grounding + web 搜索确认/refine 假设"的 agentic 任务，而非纯 retrieval。
  - **方法**：GeoVista 在推理 loop 内无缝集成 **image-zoom-in 工具**（放大兴趣区域）与 **web-search 工具**；完整训练 pipeline = cold-start SFT（学推理模式与工具先验）+ RL（hierarchical reward 利用多层级地理信息）。配套 **GeoBench** 基准（全球照片 + 全景 + 卫星图）。
  - **结果**：大幅超过开源 agentic 模型，多数指标逼近 Gemini-2.5-flash / GPT-5。
- **#47 DR Tulu: Reinforcement Learning with Evolving Rubrics for Deep Research** ⭐ [RL ReSearch, 63↑] — arXiv 2511.19399
  - **问题**：多数开源 DR 模型靠 short-form QA 的 RLVR 训练，无法迁移到真实的 long-form 任务。
  - **方法**——**RLER（RL with Evolving Rubrics）**：构建并维护与 policy 模型 **co-evolve（共同演化）** 的 rubric——rubric 能纳入模型新探索到的信息，提供 discriminative 的 on-policy 反馈。
  - **成果**：**DR Tulu-8B** 是首个直接为开放式 long-form deep research 训练的开源模型；在科学/医疗/通用 4 个 long-form 基准上大幅超过现有开源 DR 模型，匹配或超过商用 DR 系统，且更小更便宜。配套发布 **MCP-based agent 基础设施**。GitHub 659★。已成精读 [[03-dr-tulu]]。

### 2025-12（12 月）

- **#16 Probing Scientific General Intelligence of LLMs with Scientist-Aligned Workflows (SGI-Bench)** 📄 [Intern Science, 120↑] — arXiv 2512.16969
  - **定位**：提出 **SGI（Scientific General Intelligence）** 的可操作定义，基于 **PIM（Practical Inquiry Model：Deliberation → Conception → Action → Perception）**，落地为 4 个科学家对齐任务：deep research、idea generation、dry/wet 实验、experimental reasoning。
  - **基准**：**SGI-Bench**——1000+ 专家策划的跨学科样本，灵感来自《Science》125 个大问题。
  - **关键发现**：deep research 即使 step-level 对齐，exact match 也仅 10-20%；ideas 缺可行性与细节；dry 实验代码可执行率高但结果准确率低；wet 实验序列保真度低。配套 **TTRL（Test-Time RL）**：推理时优化 retrieval-augmented novelty reward，无需参考答案即可提升假设新颖度。
- **#32 Step-DeepResearch Technical Report** ⭐ [StepFun, 88↑] — arXiv 2512.20491
  - **问题**：BrowseComp 等学术基准无法满足开放式研究的真实需求（intent recognition、长程决策、跨源验证）。
  - **方法**：① **基于 Atomic Capabilities 的数据合成策略**——强化 planning 与 report writing；② 渐进式训练路径 = agentic mid-training → SFT → RL；③ **Checklist-style Judger** 显著提升鲁棒性。配套 **ADR-Bench** 填补中文 DR 评测空白。
  - **关键数字**：32B 模型在 Scale AI Research Rubrics 拿 **61.4%**；ADR-Bench 上大幅超过同级模型，匹敌 OpenAI / Gemini DeepResearch。GitHub 560★。已成精读 [[04-step-deepresearch]]。
- **#42 Deep Research: A Systematic Survey** ⭐(综述) [73↑] — arXiv 2512.02038
  - **定位**：DR 领域**首个系统性综述**——给出清晰 roadmap、基础组件、实现技术、挑战与未来方向。
  - **四大核心贡献**：① 形式化 **三阶段 roadmap** 并区分 DR 与相关范式；② 提出四大组件——**query planning（查询规划）、information acquisition（信息获取）、memory management（记忆管理）、answer generation（答案生成）**，各配 fine-grained 子分类；③ 总结优化技术（prompting / SFT / agentic RL）；④ 整合评测标准与开放挑战。GitHub 318★（持续更新）。作为本专题阅读地图，详见 README 与各精读交叉引用。

### 2026-01（1 月）

- **#3 Watching, Reasoning, and Searching: A Video Deep Research Benchmark** ⭐(已收录 huggingface/04) [QuantaAlpha, 214↑] — arXiv 2601.06943
  - **首个 video deep research benchmark**——视频提供视觉锚点，开放 web 提供可验证答案，必须做"视频→实体→web 搜索→多跳推理"。
  - **反直觉发现**：Agentic 不必优于 Workflow——强模型（Gemini-3）有增益，弱模型（MiniCPM-V）反退化 9 分。
  - 详见 `huggingface/04-videodr.md`，本专题作为多模态 DR 的起点交叉引用。
- **#18 DeepResearchEval: An Automated Framework for Deep Research Task Construction and Agentic Evaluation** ⭐ [Infinity Lab, 127↑] — arXiv 2601.09688
  - **双框架**：解决"DR benchmark 构造成本高 + 静态评测过时快"。
  - **任务构造**：persona-driven pipeline（多领域专家 persona 生成真实复杂研究需求）+ 两阶段过滤（**Task Qualification** 任务可解性 + **Search Necessity** 是否真需多源证据）。
  - **评测设计**：**Adaptive Point-wise Quality Evaluation**（按任务动态推导评测维度/权重）+ **Active Fact-Checking**（即使无引用也主动 web 搜索验证断言）。GitHub 137★。已成精读 [[13-dr-eval-and-error]]（与 Span-Level Error 合并精读）。
- **#22 Youtu-Agent** ⭐(已收录 huggingface/05) [Tencent, 119↑] — arXiv 2512.24615
  - 把 agent 的自动生成与持续优化端到端整合的开源框架；核心创新 **Training-free GRPO**（$18 击败 $10K RL）。详见 `huggingface/05-youtu-agent.md`。
- **#9 LongCat-Flash-Thinking-2601 Technical Report** 📄 [Meituan LongCat, 181↑] — arXiv 2601.16725
  - 560B MoE 推理模型，通过 domain-parallel expert training + fusion 的统一训练框架在 agentic 基准上达 SOTA；论文将 **agentic search** 列为关键能力维度之一（本专题作为旗舰模型对 DR 能力的支撑列出，非精读）。

### 2026-02（2 月）

- **#20 Vision-DeepResearch: Incentivizing DeepResearch Capability in Multimodal Large Language Models** ⭐ [155↑] — arXiv 2601.22060
  - **问题**：现有多模态搜索假设"单张 full/entity-level 图 query + 少量文本 query"就够，在真实视觉噪声下不现实，且推理深度/搜索广度受限。
  - **方法**——**多模态 deep-research 新范式**：执行 **multi-turn × multi-entity × multi-scale** 的视觉+文本搜索，在重噪声下稳健命中真实搜索引擎；支持数十步推理 + 数百次引擎交互；通过 cold-start 监督 + RL 把 DR 能力内化进 MLLM。
  - **结果**：大幅超过现有多模态 DR MLLM，且超过 GPT-5 / Gemini-2.5-pro / Claude-4-Sonnet 上搭的 workflow。GitHub 643★。已成精读 [[07-vision-deepresearch]]。
- **#26 Vision-DeepResearch Benchmark (VDR-Bench): Rethinking Visual and Textual Search for MLLMs** ⭐(配套精读) [118↑] — arXiv 2602.02185
  - **诊断现有基准两大缺陷**：① 非视觉搜索中心——答案常被文本里的 cross-textual cue 泄露，或可被 MLLM 先验知识推出；② 评测场景过度理想化——图搜索侧近似全图精确匹配即可，文搜索侧太直接。
  - **VDR-Bench**：2000 个 VQA 实例，多阶段策划 + 专家评审，模拟真实条件。配套 **multi-round cropped-search workflow** 提升视觉检索。与 [[07-vision-deepresearch]] 同源（Osilly 团队），合并精读。
- **#30 WideSeek-R1: Exploring Width Scaling for Broad Information Seeking via Multi-Agent RL** ⭐ [RLinf, 100↑] — arXiv 2602.04634
  - **核心维度**：相对 depth scaling（单 agent 多轮），提出 **width scaling（宽度扩展）**——用多 agent 系统处理"broad information seeking"。
  - **方法**——**lead-agent-subagent 框架 + MARL**：共享 LLM 但 isolated context + 专用工具，在 20k broad IR 任务上联合优化 lead agent 与并行 subagent。
  - **关键数字**：WideSeek-R1-4B 在 WideSearch 上 item F1 **40.0%**，匹敌单 agent DeepSeek-R1-671B；并行 subagent 越多性能越好。GitHub 3.7k★。已成精读 [[12-searchswarm-and-wideseek]]（与 SearchSwarm 合并：宽度 vs 委派）。
- **#41 InternAgent-1.5: A Unified Agentic Framework for Long-Horizon Autonomous Scientific Discovery** ⭐ [Intern Science, 77↑] — arXiv 2602.08990
  - **定位**：端到端科学发现统一系统，跨计算与实证（wet lab）领域。
  - **架构**：三个协调子系统 generation / verification / evolution，底层由 **deep research、solution optimization、long-horizon memory** 三项基础能力支撑。
  - **结果**：在 GAIA / HLE / GPQA / FrontierScience 上领先；能自主设计 ML 算法（algorithm discovery）、执行完整计算或 wet-lab 实验（empirical discovery，覆盖地球/生命/生物/物理）。GitHub 1.3k★。（科学发现 agent，与 SGI-Bench 同机构 Intern Science；列出，未单独精读）
- **#15 Step 3.5 Flash: Open Frontier-Level Intelligence with 11B Active Parameters** 📄 [StepFun, 200↑] — arXiv 2602.10604
  - 稀疏 MoE 模型，11B active 参数达 frontier-level agentic 智能；摘要将 **BrowseComp** 列为关键评测之一（作为旗舰模型对 DR 的支撑列出，非精读）。

### 2026-03（3 月）

- **#8 MiroThinker-1.7 & H1: Towards Heavy-Duty Research Agents via Verification** ⭐(已收录 huggingface/12) [MiroMind, 187↑] — arXiv 2603.15726
  - 把 verification 嵌入推理过程——local（中间决策评估）+ global（证据链一致性审计）。详见 `huggingface/12-mirothinker.md`。
- **#20 OpenSeeker: Democratizing Frontier Search Agents by Fully Open-Sourcing Training Data** ⭐ [OpenSeeker, 150↑] — arXiv 2603.15594
  - **定位**：**首个完全开源（模型 + 数据）** 的 frontier-level search agent，破解"高质量训练数据被工业巨头垄断"。
  - **两大创新**：① **Fact-grounded scalable controllable QA synthesis**——逆向工程 web graph（拓扑扩展 + 实体混淆），生成可控覆盖/复杂度的多跳任务；② **Denoised trajectory synthesis**——retrospective summarization 去噪轨迹，促 teacher LLM 产出高质量动作。
  - **关键数字**：仅 **11.7k 合成样本 + 单次 SFT** 即达多基准 SOTA；BrowseComp 上 29.5% vs 次优开源 DeepDive 15.3%；BrowseComp-ZH 上 48.4% 超过 Tongyi DeepResearch 46.7%（后者用了大量 continual pre-training+SFT+RL）。GitHub 746★。已成精读 [[05-openseeker]]。
- **#36 OpenResearcher: A Fully Open Pipeline for Long-Horizon Deep Research Trajectory Synthesis** ⭐ [TIGER-Lab, 100↑] — arXiv 2603.20278
  - **问题**：现有数据采集 pipeline 依赖专有 web API，大规模轨迹合成昂贵、不稳定、难复现。
  - **方法**：解耦"一次性 corpus bootstrapping"与"多轮轨迹合成"，在 **1500 万文档**离线语料上用三个显式 browser 原语（**search / open / find**）执行 search-and-browse loop。用 GPT-OSS-120B 作 teacher 合成 **97K+ 轨迹**（含 100+ tool call 的长程长尾）。
  - **关键数字**：30B-A3B backbone SFT 后 BrowseComp-Plus **54.8%**（比 base **+34.0pp**）。离线全可控环境支持受控分析（数据过滤策略、agent 配置、检索成功率与最终准确率的关系）。GitHub 777★。已成精读 [[06-openresearcher]]。
- **#30 T2S-Bench & Structure-of-Thought** 📄 [121↑] — arXiv 2603.03790 — text-to-structure 推理基准与 prompting 技术，含科学领域覆盖；与 DR 的 query planning 子能力相关（列出，非精读）。
- **#33 HopChain: Multi-Hop Data Synthesis for Generalizable Vision-Language Reasoning** 📄 [Qwen, 110↑] — arXiv 2603.17024 — 生成多跳视觉-语言推理数据提升 VLM 长链推理；与多模态 DR 的数据合成相关（列出，非精读）。

### 2026-04（4 月）

> 本月 top50 中**无新增的纯 deep-research 论文**——候选项（Agentic World Modeling [[已收录 huggingface/15]]、GameWorld、OpenGame、Terminal Agents、AgentSPEX）经审查均为 world model / 游戏 agent / 企业自动化 / agent DSL，不属于 DR 范畴，故本月不收录。这与 2026 H1 "文本 DR 已卷得差不多、4 月主题转向 world model 与 coding agent"的趋势一致。

### 2026-05（5 月）

- **#33 Beyond Semantic Similarity: Rethinking Retrieval for Agentic Search via Direct Corpus Interaction** ⭐ [TIGER-Lab, 123↑] — arXiv 2605.05242
  - **核心论点**：传统 retriever（lexical/semantic）把语料压缩成"固定相似度接口 + 单步 top-k"，对 agentic search 成了瓶颈——精确词约束、稀疏线索合取、局部上下文检查、多步假设修正都难以用现成 retriever 实现，且早期被滤掉的证据无法被下游更强推理找回。
  - **方法**——**DCI（Direct Corpus Interaction）**：agent 直接用通用终端工具（**grep、文件读取、shell、轻量脚本**）搜索原始语料，**无需 embedding 模型/向量索引/检索 API**；无需离线索引，天然适配演化中的本地语料。
  - **结果**：在 BRIGHT / BEIR 多个数据集大幅超过 sparse/dense/reranking 强基线；BrowseComp-Plus 与多跳 QA 不依赖任何语义 retriever 即达高准确率。提出"检索质量取决于交互接口的分辨率"这一新视角。GitHub 340★。已成精读 [[10-dci-and-grepseek]]（与 GrepSeek 合并：DCI 范式）。
- **#44 OpenSearch-VL: An Open Recipe for Frontier Multimodal Search Agents** ⭐ [Tencent Hunyuan, 102↑] — arXiv 2605.05185
  - **定位**：完全开源的 frontier 多模态 deep search agent 训练 recipe（agentic RL）。
  - **数据 pipeline**：Wikipedia path sampling + fuzzy entity rewriting + source-anchor visual grounding，联合降低 shortcut 与 one-step retrieval collapse；产出 SearchVL-SFT-36k + SearchVL-RL-8k。
  - **工具环境**：统一 text search / image search / OCR / cropping / sharpening / super-resolution / perspective correction，让 agent 结合主动感知与外部知识。
  - **算法**——**multi-turn fatal-aware GRPO**：mask 失败后 token，同时用 one-sided advantage clamping 保留失败前有用推理。
  - **结果**：7 个基准平均 **+10pt+**，多任务匹敌商用模型。GitHub 215★。已成精读 [[08-opensearch-vl]]。

### 2026-06（6 月）

- **#12 GrepSeek: Training Search Agents for Direct Corpus Interaction** ⭐ [UMass Amherst, 106↑] — arXiv 2605.29307
  - **范式**：与 retriever 互补——search agent 把语料本身当搜索环境，用可执行 shell 命令找证据。
  - **两阶段训练**：① cold-start 数据集——用 **answer-aware Tutor + answer-blind Planner** 生成 verified、因果 grounded 的搜索轨迹；② **GRPO** refine policy。
  - **工程**：**semantics-preserving sharded-parallel execution engine** 把 shell 检索加速 **最多 7.6×**，且 byte-exact 等价于顺序执行。
  - **结果**：7 个开放域 QA 基准上 token-level F1 与 EM 最强；也指出纯 lexical 交互在 surface-form 变化大的 query 上的局限。GitHub 42★。已成精读 [[10-dci-and-grepseek]]（与 Beyond Semantic Similarity 合并）。
- **#16 OCC-RAG: Optimal Cognitive Core for Faithful Question Answering** 📄 [OCC, 89↑] — arXiv 2606.00683
  - **理念**：很多实际应用更受益于 robust reasoning 而非海量参数知识——task-specialized **SLM** 是合理设计选择。
  - **OCC-RAG**：为 context-grounded faithful QA 优化，要求对供给段落做多跳推理、忽略记忆知识。配套合成 **300 万+** multi-context multi-hop QA 语料（多跳推理 + 严格 context 忠实度 + 校准 abstention）。
  - **结果**：OCC-RAG-0.6B/1.7B 产出带 source citation（逐字引用）的结构化推理 trace；在 HotpotQA/MuSiQue/TAT-QA（多跳）、ConFiQA（忠实度）、MuSiQue-Un（拒答）上匹敌或超过 2-6× 大的通用模型。（RAG/多跳，DR 邻接，列出）
- **#22 FORT-Searcher: Synthesizing Shortcut-Resistant Search Tasks for Training Deep Search Agents** ⭐ [RUC AIBox, 68↑] — arXiv 2606.12087
  - **问题**：训练 deep search agent 需要"证据充分前答案不可得"的 verifiable 问题；现有方法靠丰富 graph 结构提难度，但**结构复杂 ≠ 真实搜索难度**——可能被更便宜的 identifying route 抄近道。
  - **方法**——**shortcut-aware difficulty framework**：识别四类 shortcut 风险（evidence co-coverage、single-clue selectivity、exposed constants、prior-knowledge binding），用 trajectory signature（solving cost、answer hit time、prior-shortcut rate）诊断。**FORT** 在实体选择/证据图构造/问题表述/对抗 refinement 四环节控制 shortcut 风险。
  - **结果**：FORT 数据比现有开源 deep search 数据集诱导更长的 pre-answer 搜索、更少 shortcut；仅 SFT 训出的 FORT-Searcher 在挑战性基准上同级最佳。已成精读 [[09-harness-1]]（与 Harness-1、Masking 合并：DR 训练范式与状态管理）。
- **#28 Masking Stale Observations Helps Search Agents — Until It Doesn't: A Regime Map and Its Mechanism** ⭐ [McAuley-Lab, 62↑] — arXiv 2606.00408
  - **问题**：长程 search agent 跨多次 tool call 积累大量检索内容，context 预算效率重要。最小干预是 **mask 掉过期观察**，但何时有用、为何有用不清楚。
  - **系统实验**：扫 4B-284B backbone × 3 个 retriever × 离线/实时 web 基准。**核心发现**：masking 的准确率增益相对"无 context 管理时的准确率"呈 **非对称倒 U 形**——弱 retriever 下平台、强 retriever 配中等模型时峰值、模型饱和时骤降。
  - **机制**：masking 是 **token-for-turn 权衡**——移除模型已基本停止关注的观察、换取更多 turn；增益取决于 retriever recall 与模型 implicit filtering capacity 的交互。把 context 管理重构为 **regime-dependent** 干预。GitHub 16★。已成精读 [[09-harness-1]]（与 FORT、Harness-1 合并：DR 训练/上下文的"踩坑与机制"）。
- **#38 K-BrowseComp: A Web Browsing Agent Benchmark Grounded in Korean Contexts** ⭐ [CMU, 56↑] — arXiv 2606.02404
  - **定位**：韩语语境的 web-browsing agent 基准（400 题），填补韩语 agentic 评测空白。
  - **构造**：300 题 K-BrowseComp-Verified 由母语者手工构造验证；100 题合成 split 用 hard few-shot exemplar + failure-mode-targeted generation，利用"解题 vs 出题"的不对称性做对抗过滤。
  - **关键数字**：GPT-5.5 / DeepSeek-V4-Pro / GLM-5.1 仅 30.0-45.67%（远低于 BrowseComp）；韩国 PAF 项目的韩语 LLM 仅 0.0-10.33%；对抗合成 split 上最强模型仅 26.0%。已成精读 [[13-dr-eval-and-error]]（与 DeepResearchEval、Span-Level Error 合并：DR 评测）。
- **#40 Where Do Deep-Research Agents Go Wrong? Span-Level Error Localization in Agent Trajectories** ⭐ [NJU-LINK, 54↑] — arXiv 2606.02060
  - **问题**：基于最终答案的评测只能看"成没成功"，看不出"轨迹哪部分让答案不可靠"。
  - **方法**：收集 2 个 agent 框架 × 3 个 backbone × 3 个基准的 **2,790 条真实轨迹**，转成 semantic span，LLM-assisted 专家标注 harmful error span，建 **TELBench**（1000 实例）。提出 **DRIFT**——claim-centric 审计框架：追踪 agent claim、核查其在轨迹证据中的支撑、标记影响答案路径的 unsupported/conflicting span。
  - **结果**：DRIFT 把 span-level error localization 与 first-error accuracy 提升 **最多 30pp**，提供 DR agent 可靠性的 process-level 视角。GitHub 21★。已成精读 [[13-dr-eval-and-error]]。
- **#45 Harness-1: Reinforcement Learning for Search Agents with State-Externalizing Harnesses** ⭐ [Chroma, 52↑] — arXiv 2606.02373
  - **核心论点**：把 search agent 训成"在不断增长的 transcript 上的 policy"，让 RL 同时优化语义搜索决策与可恢复的 bookkeeping——后者环境能更可靠地维护。
  - **方法**——**Harness-1**：20B 检索 subagent，在 **stateful search harness** 内用 RL 训练。harness 维护环境侧 working memory（candidate pool、importance-tagged curated set、compact evidence links、verification records、压缩去重的 observation、budget-aware context rendering）；policy 只保留语义决策（搜什么、留/弃哪些文档、验证什么、何时停）。
  - **关键数字**：8 个检索基准（web/finance/patents/multi-hop QA）平均 curated recall **0.730**，比次优开源 subagent **+11.4pt**，在 held-out transfer 上增益尤强。GitHub 548★。已成精读 [[09-harness-1]]。
- **#48 SearchSwarm: Towards Delegation Intelligence in Agentic LLMs for Long-Horizon Deep Research** ⭐ [SearchSwarm, 49↑] — arXiv 2606.09730
  - **核心能力**：**delegation intelligence（委派智能）**——main agent 分解任务、决定何时委派什么给 subagent、整合 subagent 返回的摘要结果，以节省主上下文预算。
  - **方法**：设计 harness 引导模型做高质量任务分解与委派，并约束 subagent 正确返回结果；harness 引导的轨迹天然编码正确委派决策，用作 SFT 数据把 delegation intelligence 内化进权重。
  - **关键数字**：SearchSwarm-30B-A3B 在 BrowseComp **68.1** / BrowseComp-ZH **73.3**，同级最佳。GitHub 63★。已成精读 [[12-searchswarm-and-wideseek]]（与 WideSeek-R1 合并）。
- **#43 WeaveBench: A Long-Horizon, Real-World Benchmark for Computer-Use Agents with Hybrid Interfaces** 📄 [Microsoft, 52↑] — arXiv 2606.09426
  - **定位**：长程 hybrid-interface 基准（114 任务 × 8 真实工作域），要求 agent 在单条轨迹内结合 GUI 观察/动作与 CLI/code 操作；在真实 Ubuntu 桌面 + 部署的 CLI-agent runtime 上评测。
  - **关键发现**：最佳 PassRate 仅 **41.2%**（远未饱和）；配套 trajectory-aware judge 揭示 outcome-only grading 大幅高估 agent 性能，并能检测 fabricated visual evidence / hard-coded metric 等 shortcut。（computer-use，DR 邻接，列出）

---

## 二、统计速览

| 月份 | 月榜论文总数 | top50 内 DR 命中 | 本专题收录 | 精读 |
| --- | --- | --- | --- | --- |
| 2025-11 | 105 | 5 | 5 | 4 ⭐ |
| 2025-12 | 105 | 3 | 3 | 2 ⭐ + 1 综述 |
| 2026-01 | 105 | 4(含2已收录) | 2 新 | 1 ⭐ |
| 2026-02 | 105 | 4 | 4 | 4 ⭐ |
| 2026-03 | 105 | 2 新(+1已收录) | 2 新 | 2 ⭐ |
| 2026-04 | 105 | 0 真 DR | 0 | — |
| 2026-05 | 105 | 2 | 2 | 2 ⭐ |
| 2026-06 | 105 | 9 | 9 | 7 ⭐ |
| **合计** | — | — | **27** 篇（新收录） | **13** 篇精读 |

> 注：2026-06 是 DR 论文密度最高的月份（9 篇），与"搜索 agent 训练方法学（DCI、shortcut-resistant、state-externalizing、delegation）+ 过程级评测"在 2026 H1 末集中爆发吻合。

---

## 三、精读论文一览（13 篇，按主题）

| 编号 | 论文 | 月/票 | 机构 | 主题 |
| --- | --- | --- | --- | --- |
| **01** | [MiroThinker v1.0](01-mirothinker-v1.md) | 11/196↑ | MiroMind | Interaction scaling 第三维度 |
| **02** | [IterResearch](02-iterresearch.md) | 11/80↑ | — | Markovian state，反 context 窒息 |
| **03** | [DR Tulu](03-dr-tulu.md) | 11/63↑ | RL ReSearch | RLER 演化 rubric，long-form |
| **04** | [Step-DeepResearch](04-step-deepresearch.md) | 12/88↑ | StepFun | Atomic capability 数据合成 |
| **05** | [OpenSeeker](05-openseeker.md) | 03/150↑ | OpenSeeker | 全开源数据，逆向 web graph |
| **06** | [OpenResearcher](06-openresearcher.md) | 03/100↑ | TIGER-Lab | 离线 corpus + 97K 轨迹合成 |
| **07** | [Vision-DeepResearch (+VDR-Bench)](07-vision-deepresearch.md) | 02/155↑ | — | 多模态 DR + 配套基准 |
| **08** | [OpenSearch-VL](08-opensearch-vl.md) | 05/102↑ | 腾讯混元 | 多模态搜索 fatal-aware GRPO |
| **09** | [Harness-1 (+FORT+Masking)](09-harness-1.md) | 06/52↑ | Chroma | State externalization & 训练踩坑 |
| **10** | [DCI & GrepSeek](10-dci-and-grepseek.md) | 05-06/123↑ | TIGER-Lab/UMass | 直接语料交互范式 |
| **11** | [General Agentic Memory](11-general-agentic-memory.md) | 11/171↑ | BAAI | JIT 记忆 = deep research |
| **12** | [SearchSwarm & WideSeek-R1](12-searchswarm-and-wideseek.md) | 06/02 | SearchSwarm/RLinf | 委派智能 vs 宽度扩展 |
| **13** | [DR 评测三件套](13-dr-eval-and-error.md) | 01-06 | Infinity/NJU/CMU | DeepResearchEval+Span-Error+K-BrowseComp |

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
