# 2025.11–2026.06 HuggingFace 月榜 — Agentic RL 专题汇总

> **数据范围**：HuggingFace Papers **月榜 top 50**，2025-11 至 2026-06（共 8 个月）
> **标记**：⭐ = 本专题精读｜📄 = 基于摘要的简述｜🔗 = 已在 `huggingface/` 周榜库精读，此处交叉引用

---

## 一、分类总览

> Agentic RL = 「**把 RL 的信用分配/奖励/探索机制，适配到 agent 的多轮、长程、工具交互轨迹上**」。2025 H2–2026 H1 这条线已从「能不能用 RL 训 agent」走到「**怎么把 rollout 成本、稀疏延迟奖励、多轮信用分配、技能内化做精细**」。

| 主题                                         | 精读                                                         | 简述                                                                                                              | 一句话定位                                                                       |
| -------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **A. RL 算法 / 训练范式**              | 01 DreamGym · 02 ERL · 03 OpenClaw-RL · 04 LLM-in-Sandbox | MEDS · DVAO · GrandCode🔗 · SDAR🔗                                                                             | 把"经验合成 / 反思 / next-state 信号 / 极简环境"做进 RL loop                     |
| **B. Agent 自演化 & 经验**             | 05 Agent0 · 06 EvoCUA · 07 CUDA Agent                      | MetaClaw · Agent-World · GroundCUA · GeoVista · Role-Agent · OpenGame · Thinking-with-Map · SWE-rebench-V2 | 自动合成环境/任务 + 大规模 rollout + 失败回收的自维持循环                        |
| **C. Multi-Agent RL**                  | 08 HACRL · 09 MATTRL · 10 WideSeek-R1                      | SearchSwarm                                                                                                       | 异构互学 / 测试时协作 / 宽度扩展——MARL 的三种新解法                            |
| **D. Deep Research / Search Agent RL** | 11 IterResearch · 12 DR Tulu · 13 Harness-1                | OpenSearch-VL🔗 · GrepSeek🔗 · Observation-Masking · DRIFT · MiroThinker🔗                                    | 长程上下文管理 + 可验证奖励 + 状态外置，是 DR 训练的三条主线                     |
| **E. Skill RL**                        | 14 SkillRL · 15 SKILL0                                      | Skill1🔗                                                                                                          | 把 skill 的发现/检索/内化统一进 RL，取代 inference-time 检索                     |
| **F. 旗舰模型（Agent RL 训练）**       | —                                                           | GLM-5🔗 · Kimi K2.5🔗 · Youtu-Agent🔗 · LongCat-Flash-Thinking                                                 | 工业旗舰里 agentic RL 的工程化落地（异步 RL / Agent Swarm / Training-free GRPO） |
| **G. Agent 安全 + RL**                 | —                                                           | AgentDoG 1.5                                                                                                      | 安全对齐里用到 RL training                                                       |

---

## 二、按月列出（每篇 800–1000 字详述，基于 arXiv 原文）

> 每篇展开为 800–1000 字，基于论文正文（method/实验节）撰写，覆盖**问题动机 / 核心方法（含算法细节）/ 关键结果（含具体数字与 ablation）/ 意义与定位**。⭐ = 本专题精读，📄 = 简短描述（已扩写），🔗 = 已在 `huggingface/` 精读、此处交叉引用。

### 2025-11（14 命中 / 收录 9）

#### ⭐ #17 Agent0: Self-Evolving Agents from Zero Data via Tool-Integrated Reasoning [UNC, 110↑] — arXiv 2511.16043

**问题动机**：LLM agent 用 RL 训练时严重依赖人类策划的数据——既限制规模，也把 AI 能力上限拴死在人类知识上。已有自演化框架有两大短板：受限于模型自身能力（自己出的题不会超过自己太多）、且多为单轮交互，难构造涉及工具使用/动态推理的复杂课程。

**核心方法**：Agent0 从同一个 base LLM（Qwen3-Base）初始化两个 agent——**Curriculum Agent π_θ（出题者）** 与 **Executor Agent π_φ（解题者）**——做 T=3 轮共进化。每轮两阶段：① **Curriculum Evolution**——用 GRPO 训 π_θ 生成挑战*冻结*的 executor 的任务；② **Executor Evolution**——冻结 π_θ、生成任务池、过滤出数据集 𝒟，再用 **ADPO** 训 π_φ。**任务过滤**靠 self-consistency p̂(x)（k 个回答里多数票占比）：𝒟={x : |p̂(x)−0.5|≤δ}，δ=0.25（保留 p̂∈[0.3,0.8] 的"不太易也不太难"任务）。**Curriculum reward** 三信号：Uncertainty（R_unc=1−2|p̂−0.5|，p̂=0.5 时最大）+ Tool Use（R_tool=γ·min(N_tool,C)，λ=0.6、cap=4，鼓励工具调用）+ Repetition Penalty（BLEU 聚类的任务相似度惩罚）。**工具集成推理**用"stop-and-go"多轮 rollout：模型生成推理 → 输出 ``python`` → 暂停 → 沙箱执行 → ``output`` 回传 → 继续直到 \boxed{} 答案（基于 VeRL-Tool）。关键算法 **ADPO（Ambiguity-Dynamic Policy Optimization）** 在 GRPO 上加两点：**Ambiguity-Aware Advantage Scaling**（降权低一致性=噪声伪标签样本）+ **Ambiguity-Modulated Trust Regions**（对模糊任务放松 upper clip ε_high）。

**关键结果**：Qwen3-8B-Base 上**数学推理 +18%、通用推理 +24%**。数学 AVG：Base 49.2 → Agent0 58.2（超 R-Zero 6.4%、Absolute Zero 10.6%、Socratic-Zero 3.7%）；AIME24 13.9→28.0、AIME25 16.7→24.8。通用推理 8B：34.5→42.1、4B：27.1→37.6。**共进化进程证据**：固定 executor 的 pass rate 逐轮 64.0→58.5→51.0 下降、平均 tool call 1.65→2.10→2.60 上升——证实"出题越来越难、解题越来越依赖工具"的良性循环（8B 数学逐轮 55.1→56.5→58.2）。基准：数学用 AMC/Minerva/MATH/GSM8K/Olympiad/AIME，通用用 SuperGPQA/MMLU-Pro/BBEH。

**意义**：Agent0 证明 agent 可"自己造课程、自己出题、自己进化"，摆脱人类数据天花板。出题者-解题者对抗是优雅的 curriculum 自动化（self-play 的课程版），ADPO 对"伪标签噪声"的处理是零数据自演化能稳定的关键。与同组 SkillRL [[14-skillrl]]、MetaClaw 形成 UNC aiming-lab 的自演化 agent 系列，与 DreamGym [[01-dreamgym]]、EvoCUA [[06-evocua]] 共享"自动造任务/环境"内核。精读见 [[05-agent0]]。GitHub github.com/aiming-lab/Agent0。

#### ⭐ #33 DreamGym: Scaling Agent Learning via Experience Synthesis [Meta, 83↑] — arXiv 2511.03773

**问题动机**：RL 能通过交互增强 LLM agent，但实际落地受阻于**昂贵的 rollout、有限的任务多样性、不可靠的奖励信号、复杂的基建**——这些都阻碍可扩展经验数据的收集。真实环境 rollout 慢、贵、奖励噪声大，是 agent RL 工业落地的最大障碍。

**核心方法**：DreamGym 是**首个为可扩展性设计、用合成经验做 online agent RL** 的统一框架。核心是**把环境动态蒸馏成一个 reasoning-based experience model（基于推理的经验模型）**——给定 `(state, action)`，模型通过 step-by-step 推理产生**一致的状态转移与反馈信号**，替代昂贵的真实环境 rollout。三大组件：① **Reasoning-based experience model**——不直接预测下一帧像素/HTML，而是用语言推理刻画环境状态转移逻辑（"点了提交→表单进入 pending→若字段缺失则报错"），一次 LLM 前向代替一次真实容器执行；② **Experience replay buffer**——用离线真实数据初始化、并持续用新交互补充，提升 transition stability 与质量；③ **Adaptive 课程**——经验模型自适应生成挑战当前 policy 的新任务，实现 online curriculum learning。对照 RL 算法用 GRPO 与 PPO，核心卖点是"用什么数据喂 RL"而非新 RL 算法。

**关键结果**：在 **WebArena 非 RL-ready 设定**超所有基线 **30%+**；在 RL-ready 但 rollout 昂贵的设定下，**仅用合成交互即媲美 GRPO/PPO**；**sim-to-real 迁移**用远少于真实交互量获得显著额外增益。作者含 Jason Weston、Dawn Song、Bo Li（Meta，18 作者）。

**意义**：DreamGym 把 agent RL 的成本从"维护真实环境集群"降到"训练 + 调用一个经验模型"，大幅降低复现门槛，确立 **world-model-for-RL** 在 LLM agent 语境下的可行性——经验模型用"推理"而非"像素生成"，避开传统 video world model 的保真度/成本难题。它与 EvoCUA [[06-evocua]] 形成 CUA/agent 降本两路（合成经验代替 rollout vs 规模化真实 rollout + 失败回收），是 2026 H1"绕过真实 rollout"潮流的源头。局限：经验模型的保真度上界决定一切，故仍需少量真实交互校准 sim-to-real gap。精读见 [[01-dreamgym]]。

#### ⭐ #36 IterResearch: Long-Horizon Agents via Markovian State Reconstruction [阿里 Tongyi Lab, 80↑] — arXiv 2511.07327

**问题动机**：现有 deep-research agent 用 **mono-contextual 范式**（所有信息堆进单一膨胀上下文窗口），随任务推进（可达数百步）引发 **context suffocation（上下文窒息）**——有效信息被海量无关 token 稀释、推理容量被吃满——与 **noise contamination（噪声污染）**——早期失败路径、过时观测持续干扰后续决策。

**核心方法**：把长程研究**重新形式化为 MDP + 策略性工作区重建**。不做线性累积，而是维护一份 **evolving report（演进报告）作为记忆**——每隔若干步把已有信息**合成为洞见**写进报告、丢弃冗余原始内容，基于"重建后的紧凑工作区"继续决策，保证**在任意探索深度下推理容量一致**（不随步数退化）。训练侧配套 **EAPO（Efficiency-Aware Policy Optimization）**：用**几何奖励折扣**激励高效探索（早找到答案的轨迹奖励更高）+ **自适应下采样**稳定分布式训练。把"扩展"从"更大模型/更长上下文"转向"更多交互步数"——周期性重建让交互可扩展到 2048 步而不崩。

**关键结果**：6 个基准平均 **+14.5pp**；交互扩展到 **2048 次**时性能从 **3.5% 飙升到 42.5%**；即使只作为**纯 prompting 策略**（不训练），也能让 frontier 模型在长程任务上比 ReAct **+19.2pp**。16 位作者（含 Wayne Xin Zhao、Pengjun Xie、Jingren Zhou），ICLR 2026 收录。

**意义**：IterResearch 给"上下文该不该装下一切"明确的否定回答——**上下文是需要周期性 curate 的状态**。它与 Harness-1 [[13-harness-1]] 的"状态外置"是同一病症（mono-context 膨胀）的两条解法（policy 周期性重建 vs 环境侧维护）。EAPO 把"高效探索"做进奖励设计，与 GrandCode 的难度自适应长度惩罚同属"用奖励塑形控制 agent 行为长度"。作为"深度扩展"代表，与 WideSeek-R1 [[10-wideseek-r1]]（宽度扩展）形成 DR scaling 双维坐标。精读见 [[11-iterresearch]]，亦是阿里通义 DeepResearch 系（`tongyi-deepresearch/`）与 `deep-research/02` 的核心。GitHub github.com/Chen-GX/IterResearch。

#### 📄 #4 MiroThinker: Interaction Scaling for Research Agents [MiroMind, 196↑] — arXiv 2511.11793 🔗

**问题动机**：开源 research agent 一直在 model size、context length 两维 scaling，但纯拉长推理链会退化（孤立运行、误差累积），社区缺一条可预测的第三 scaling 轴。

**核心方法**：MiroThinker 提出**第三个 scaling 维度——interaction scaling（交互规模）**：系统训练模型处理更深、更频繁的 agent-environment 交互。与 test-time scaling（推理链拉长）的本质区别——interactive scaling 靠**环境反馈 + 外部信息获取**纠错、refine trajectory，把"交互深度"确立为可预测的 scaling law。256K 上下文窗口下单任务可执行**最多 600 次 tool call**。tool-augmented research agent 用 RL 后训练提升单步工具使用可靠性。

**关键结果**：72B 变体在 **GAIA 81.9% / HLE 37.7% / BrowseComp 47.1% / BrowseComp-ZH 55.6%**——超过此前所有开源 agent，逼近 GPT-5-high。GitHub 8.2k★。

**意义**：MiroThinker 是开源 research agent 的奠基性工作，把"交互"提升为与 model size、context length 并列的第三 scaling 轴，为 IterResearch（交互推到 2048 次）、SearchSwarm（委派智能）等奠基。其 H1 版（验证嵌入推理）已在 `huggingface/12-mirothinker.md` 精读，本专题作为"interaction scaling"主线源头交叉引用（亦见 `deep-research/01`）。

#### 📄 #20 GroundCUA: Grounding Computer Use Agents on Human Demonstrations [ServiceNow, 107↑] — arXiv 2511.07332

**问题动机**：构建可靠的 computer-use agent 需要 **grounding**（把自然语言指令精确连到正确的屏幕元素）。web 和 mobile 交互有大量数据集，但**桌面环境的高质量 grounding 资源稀缺**。

**核心方法**：GroundCUA 基于**人类演示**构建桌面 grounding 数据集——记录真实用户在桌面环境的操作演示，提取"指令→屏幕元素"的精确对应关系。配套 GroundNext 模型，RL 用于优化 grounding 准确率——让 agent 准确把"点击保存按钮"这样的指令连到屏幕上正确的像素位置。

**关键结果**：在桌面 grounding 基准上显著提升 grounding 准确率（具体数字见论文）。GitHub github.com/ServiceNow/GroundCUA。

**意义**：GroundCUA 填补了桌面 CUA grounding 数据的空白——grounding 是 CUA 可靠执行的前提（连不对元素，再强的规划也白搭）。它偏数据基建，与 EvoCUA [[06-evocua]]（自动合成 CUA 经验）、`huggingface/` 的 CUA-Suite/Video2GUI 形成 CUA 数据侧的多条路线（人类演示 grounding vs 自动合成 vs 挖视频），本专题作为 agent_train 数据侧工作列出（非精读）。

#### 📄 #24 GeoVista: Web-Augmented Agentic Visual Reasoning for Geolocalization [Tencent Hunyuan, 98↑] — arXiv 2511.15705

**问题动机**：现有 agentic 视觉推理多聚焦图像操作工具，缺乏更通用的 agentic 模型。地理定位此前多被当纯 retrieval 问题，但人类做法是"看图找路标→想区域→web 搜索确认/refine 假设"，是显式的工具增强推理过程。

**核心方法**：GeoVista 把地理定位**重表述为 agentic 任务**——在推理 loop 内无缝集成 **image-zoom-in 工具**（放大兴趣区域看清细节）与 **web-search 工具**（搜索路标/地名确认假设）。完整训练 pipeline = **cold-start SFT**（学推理模式与工具先验）+ **RL**（hierarchical reward 利用多层级地理信息——国家/城市/街道逐级奖励）。配套发布 **GeoBench** 基准（覆盖全球照片 + 全景图 + 卫星图三种视角）。

**关键结果**：大幅超过开源 agentic 模型，多数指标逼近 Gemini-2.5-flash / GPT-5。GitHub github.com/ekonwang/GeoVista。

**意义**：GeoVista 把"thinking with tool"原则具象到地理定位域，证明在专门任务上 agentic 设计（视觉 grounding + web 搜索 + RL）可超越纯 retrieval 范式并逼近 frontier 闭源。它与 `huggingface/` 的 Thinking-with-Map（地图工具 agentic RL）同属"视觉 + web 工具 + RL"的地理定位路线，是多模态 deep research 在垂直任务上的代表（本专题作为 agent_train 简述，亦见 `deep-research` 月榜）。

### 2025-12（17 命中 / 收录 2）

#### 📄 #1 From Code Foundation Models to Agents and Applications: A Practical Guide to Code Intelligence [Beihang, 305↑] — arXiv 2511.18538

**问题动机**：LLM 已根本性地改变自动化软件开发（GitHub Copilot、Cursor 等驱动商业采用），但 code intelligence 领域缺乏一份系统、实用的指南——从 code foundation model 到 coding agent 到应用的完整图景。

**核心方法**：这是一篇 **code intelligence 大综述/实用指南**，系统梳理从 code foundation model（代码基础模型）到 agents（agent 化）再到 applications（应用）的演进。含 coding agent 的 **RL 训练**章节——梳理 SWE agent 如何用 RL（SWE-RL、SkyRL 等）从真实 issue/repo 学习，以及数据构建、训练范式、关键挑战（长上下文、多文件 patching 一致性、回归测试）。

**关键结果**：提供 code intelligence 的术语体系、方法分类、技术总结与实践指南。

**意义**：本综述是 code intelligence/coding agent 方向的 reference，其 RL 训练章节对理解"coding agent 如何用 agentic RL 训练"有参考价值。本专题作为 coding-agent RL 的综述类工作列出（非精读），与 GrandCode（竞赛编程 agentic RL）、CUDA Agent [[07-cuda-agent]]（系统软件 agentic RL）等具体工作互补。

#### 📄 #13 ToolOrchestra: Elevating Intelligence via Efficient Model and Tool Orchestration [NVIDIA, 128↑] — arXiv 2511.21689

**问题动机**：LLM 是强大的通才，但解决 HLE（Humanity's Last Exam）这类深而复杂的问题，既概念上困难又计算上昂贵——直接用最强大模型攻每道题成本过高。

**核心方法**：ToolOrchestra 证明**小 orchestrator（编排器）调度其他模型和工具**可以高效攻克复杂问题。用 RL 训练一个轻量编排策略——它学会针对不同子问题动态选择最合适的模型/工具组合（便宜模型先试、失败再升级到强模型），以低成本逼近"全程用最强模型"的效果。

**关键结果**：在 HLE 等复杂基准上以显著更低成本达到有竞争力的表现（具体数字见论文）。GitHub github.com/NVlabs/ToolOrchestra。

**意义**：ToolOrchestra 把 agent 编排从"训练大模型当 router"推到"用 RL 训轻量 orchestrator"，更接近实际工业 latency/成本约束。它与 `huggingface/` 的 SkillOrchestra（skill 感知路由）同属"agent/工具编排"方向，本专题作为 orchestration + RL 的工作列出（非精读），核心 RL 贡献是"用 RL 学性能-成本权衡的编排策略"。

### 2026-01（16 命中 / 收录 7）

#### ⭐ #35 EvoCUA: Evolving Computer Use Agents via Scalable Synthetic Experience [美团, 92↑] — arXiv 2601.15876

**问题动机**：原生 CUA 的潜力被**静态数据 scaling 的天花板**卡住——主流做法是被动模仿静态数据集，覆盖不了真实操作的长尾。

**核心方法**：EvoCUA 把数据生成与 policy 优化合并为**自维持进化循环**。① **Verifiable Synthesis Engine**——"Generation-as-Validation"范式消除 reward hacking：层级域 taxonomy 把 app 分解为原子能力，ReAct VLM "Task Architect"**双流共生成**指令流 g + validator 流 V_g（ground truth + 可执行评估代码），闭环反馈在真实沙箱执行生成代码直到成功，配三重去污（语义/配置/评估器），扩到数万实例。② **可扩展基建**——基于 tools（不可变环境定义）+ clusters（动态扩展单元）抽象，异步 gateway 用 reactor 模式（每分钟数十万请求），burst scaling 一分钟拉起数万沙箱，持续**>100,000 并发沙箱**（QEMU-KVM VM 封进 Docker）。③ **Iterative Evolving Learning** 三阶段：**Cold-Start**（统一动作空间 + 结构化思维空间 + Hindsight Reasoning Generation）→ **RFT**（Dynamic Compute Budgeting：分层预算 K*=k_{i*}，i*=min{i:SR(k_i)≥τ_i}，把算力集中到**boundary queries** 高方差任务 + Step-Level Denoising）→ **失败回收 RL（Step-Level DPO）**：Causal Deviation Discovery 定位 Critical Deviation Step t*，Paradigm I（在 t* 做 Action Correction）+ Paradigm II（在 t*+1 做 Reflection and Recovery，盲目续走=rejected、合成反思轨迹=chosen）。

**关键结果**：**OSWorld EvoCUA-32B 56.7%**（开源 SOTA，50 步），EvoCUA-8B 46.1%——超 OpenCUA-72B 45.0%（+11.7%）、UI-TARS-2 53.1%（+3.6%）、与 Claude-4.5-Sonnet（50 步 58.1%）差距缩到 1.4%；EvoCUA-8B 超同 backbone 的 Step-GUI-8B +5.9%。**Ablation**（32B，逐阶段 Δ）：+统一动作空间 +4.84%、+Cold Start +2.62%、+RFT +3.13%、+Offline DPO +3.21%、+Iterative +1.90%。**经验 scaling**（OpenCUA-72B 纯 RFT）：20k +2.61pp → 226k +6.79pp → 1M +8.12pp。逾千次实验、超 100 万加速器小时。

**意义**：EvoCUA 把"环境自动合成 + 大规模并行 RL + 失败数据回收"整合为可持续运转的工业级 CUA 生产线，是 CUA 训练从"静态数据"到"自维持循环"的范式转变。失败回收（Step-Level DPO 定位关键分叉点）印证 2026 H1 共识——**失败轨迹 = 高质量"诊断+修复"监督**。与 DreamGym [[01-dreamgym]] 形成 CUA 降本两路。精读见 [[06-evocua]]。GitHub github.com/meituan/EvoCUA。

#### ⭐ #36 MATTRL: Collaborative Multi-Agent Test-Time RL for Reasoning [NUS / MIT, 92↑] — arXiv 2601.09667

**问题动机**：multi-agent RL（MARL）训练**资源重且不稳定**——队友 co-adapting 导致非平稳性，奖励稀疏且高方差。能否把 RL 的"经验积累"挪到推理时、不更新权重？

**核心方法**：MATTRL **把结构化文本经验注入多 agent 推理时审议**，不更新任何权重。三阶段：① **Team formation**——coordinator 从预定义 specialist catalog（24 个医学科室）选团队（数学用"自由招募"）；② **经验增强对话达成共识**——同步多轮（max R=3），每个未收敛 specialist 检索经验（Qwen3-Embedding-4B + FAISS，top-K=8 余弦相似度，"EXPERIENCE HINTS"模板）、更新意见、通过"MEETING"聚合；③ **Report synthesis**——coordinator 综合证据出报告与最终决策（把证据聚合与决策分离）。**turn-level credit assignment**：每条发言由 LLM-judge 打分 s_{i,t}，团队结果 G 按衰减权重 w_t=γ^{R−t} 回传，r_{i,t}=λs_{i,t}+(1−λ)G·w_t·c_{i,t}；高价值发言（top 25%）被蒸馏成可检索文本。研究**三种 credit-assignment**：Naive（比例归一）/ Difference Rewards（反事实中和 agent i）/ Shapley-style（Monte Carlo 排列的边际贡献）。

**关键结果**：医学（RareBench Task4）Hit@1 0.39/Hit@10 0.75/MRR 0.51（超 RareAgent-Refined 0.35/0.70/0.47）；数学（HLE）单 agent 0.27→多 agent 0.33→MATTRL 0.36；教育（SuperGPQA）post 0.60→0.73→**0.77**。整体**比多 agent +3.67%、比单 agent +8.67%**。**Credit ablation**：Difference Rewards 在严格精度上最优（Shapley"把功劳摊到联盟、有限排列下方差大、稀释决定性发言"），推荐 Difference 为默认。团队规模 Hit@1 在三 agent 时峰值（三 agent 比单 agent Hit@10 高约 14%）。

**意义**：MATTRL 提供 MARL 的零训练替代——在"训练式 MARL 太贵/不稳"场景，test-time 经验注入是实用解法。它把"经验积累 + 信用分配"剥离出权重训练，证明可纯靠文本机制在推理时实现（与 Youtu-Agent 的 Training-free GRPO 精神一致）。与 HACRL [[08-hacrl]]（训练式异构）、WideSeek-R1 [[10-wideseek-r1]]（训练式宽度）构成 MARL 三种新解法——MATTRL 占据"测试时/零训练"一极。精读见 [[09-mattrl]]。GitHub github.com/zhiyuanhubj/MATTRL。

#### ⭐ #38 LLM-in-Sandbox: Computer Environments Elicit General Agentic Intelligence [Microsoft Research, 87↑] — arXiv 2601.16206

**问题动机**：主流假设是"agent 必须配复杂 harness（LangChain/AutoGen 一大堆脚手架）"。但能否把计算机虚拟化为**仅含基础功能的 code sandbox**，就激发 LLM 的通用任务求解元能力？

**核心方法**：sandbox 是基于 Docker 的 Ubuntu 系统（单一 ~1.1GB 共享镜像），**仅三个工具**：`bash`（执行任意终端命令，作为安装包/引导功能的元工具）+ `file_editor` + `finish`，基础环境只有 Python + NumPy/SciPy，域工具交给模型自己装。**三大元能力**从交互中自然涌现：外部资源访问（如装 Java+OPSIN 转化学名）、文件管理（如对 100K+ token 报告先 grep/sed 再写 Python）、代码执行（如组合搜索解指令约束）。**LLM-in-Sandbox-RL** 关键——**只用非 agentic 数据**（普通问答/推理/选择题，来自 Instruction Pre-Training，与任何评测基准零重叠、不针对 coding）训练弱模型。**核心设计——context as files**：背景材料存成环境里的文件（多文档拆成 introduction.txt/methods.txt、单文件加 distractor）而非塞进 prompt，强制探索。RL 用 outcome-based reward（跟 DeepSeek-R1，整条轨迹按最终输出正确性给 rule-based 奖励），基于 rLLM/R2E-Gym/DeepSWE。

**关键结果**：7 模型评测，**强模型免训练**最大 +15.5%（Qwen3-Coder 数学 26.0→41.5），DeepSeek-V3.2 指令跟随 +14.4%、Claude-Sonnet-4.5 +12.7%、GPT-5 数学 +10.1%。**token 8× 缩减**（长上下文 Qwen3-Coder 102.9K→12.9K），总 token 仅 LLM 模式的 0.5–0.8×。**强弱分化**：强模型 External 6.2%/File 21.1%/Computation 12.5%、平均 12.6 turn；弱 Qwen3-4B 仅 0.8%/2.9%/2.9%、平均 23.7 turn（"乱逛"，数学反退 13.5%）。**RL 后**：Qwen3-4B sandbox 模式数学 32.5→53.3（+20.8）且迁移回纯文本模式，验证行为从 20.22→36.91 次/回答。**基建**：1.1GB 镜像 vs SWE-Gym 6TB，执行 <4% 总时间。

**意义**：LLM-in-Sandbox 质疑"agent 必须复杂 harness"的主流假设，给出"sandbox + RL"极简替代。与 Code as Agent Harness（`huggingface/20`）形成张力（后者论证代码是 substrate，本文论证最小化代码环境已足够），与 Harness-1 [[13-harness-1]] 构成"环境职责切分"两极（做薄 vs 做厚）。"非 agentic 数据即可训 agent"与 DreamGym 的"合成经验即可训 agent"共同降低 agent RL 数据门槛。精读见 [[04-llm-in-sandbox]]。

#### 📄 #9 LongCat-Flash-Thinking-2601 Technical Report [Meituan LongCat, 181↑] — arXiv 2601.16725

**问题动机**：如何训出在 agentic 基准上达 SOTA 的大规模开源推理模型，同时兼顾计算效率？

**核心方法**：LongCat-Flash-Thinking-2601 是 **560B 参数 MoE 推理旗舰**，通过 **domain-parallel expert training + fusion（领域并行专家训练 + 融合）** 的统一训练框架，在 agentic 基准上达 SOTA。把 **agentic search** 列为关键能力维度——模型具备多轮工具调用、tool-integrated reasoning 能力，用 RL 训练 multi-turn tool use 完成 deep research 类任务。

**关键结果**：在多个 agentic 基准上达开源 SOTA 级，agentic search 为其核心能力维度之一。GitHub github.com/meituan-longcat/LongCat-Flash-Thinking-2601。

**意义**：LongCat 代表"大规模 MoE 推理旗舰把 agentic search/tool use 作为一等能力、用 RL 训练"的趋势——DR/工具能力不再是外挂工程，而是旗舰训练时就内化的核心维度。本专题作为旗舰模型（agentic RL 训练）列出（非精读），与 GLM-5、Kimi K2.5、Step 3.5 Flash 同属"旗舰模型的 agentic 化"潮流。

#### 📄 #11 Thinking with Map: Reinforced Parallel Map-Augmented Agent [AMAP, 170↑] — arXiv 2601.05432

**问题动机**：现有 VLM 地理定位方法把它当 retrieval 问题，但人类做法是"看图找路标→想区域→看地图比对→缩小范围"，是显式的工具增强推理过程。

**核心方法**：让 VLM 在图像地理定位中**像人类一样使用地图**。① **Agent-in-the-map loop**——agent 显式调用地图工具（缩放/平移/对照路标），把每步地图截图作为新观察；② **两阶段训练**——agentic RL（GRPO 训 multi-step 地图工具使用）+ Parallel TTS（test-time 多个 agent 并行推断后投票）；③ 配套发布 **MAPBench** 基准（城市/农村/特殊地标场景）。

**关键结果**：**Acc@500m（500 米误差内）从 8.0%（Gemini-3-Pro + Google Map baseline）提到 22.1%**（提升近 3×）。

**意义**：Thinking with Map 把"thinking with tool"原则具象到地图域，证明在专门任务上 agentic 设计（地图工具 + GRPO + parallel TTS）可超越 frontier 闭源 + 工具的简单组合。它与 GeoVista（web 工具地理定位）同属"视觉 + 工具 + agentic RL"路线。本专题作为 agent_train 简述（亦见 `huggingface/00` W03）。

#### 📄 #22 Youtu-Agent: Automated Generation + Hybrid Policy Optimization [Tencent, 119↑] — arXiv 2512.24615 🔗

**问题动机**：当下 agent 系统两大痛点——手工搭建 agent 太贵（数百 person-days）+ RL 训练成本极高（上万美元）。

**核心方法**：把 agent 的自动生成与持续优化端到端整合的开源框架。四件套：三层 YAML 框架（Agent/Tools/Env 解耦）+ 自动配置（Workflow 专家模板 + Meta-Agent 自动生成 YAML）+ **Training-free GRPO**（核心——用 LLM evaluator 把 group rollout 成败蒸馏成"语义 advantage"注入 textual LoRA，**$18 击败 $10K 传统 RL**）+ 端到端 Agent RL（基于 Agent-Lightning + VeRL，扩到 128 GPU）。

**关键结果**：WebShop/GAIA/SWE-bench Lite 等多基准达开源 SOTA；Training-free GRPO 相对 baseline 提升 8–15pp。

**意义**：Youtu-Agent 把"data synthesis + automated configuration + training-free optimization"做成完整工程闭环，是工业级 agent 框架代表。Training-free GRPO 与 MATTRL [[09-mattrl]]（测试时经验）同属"frozen model + 文本侧优化"。详见 `huggingface/05-youtu-agent.md`，本专题作为旗舰框架交叉引用。

#### 📄 #27 Let It Flow / ROME: Open Agentic Learning Ecosystem [AGI Lab, 109↑] — arXiv 2512.24873

**问题动机**：开源社区缺一个有原则的、端到端的生态来支撑 agentic crafting（agent 在真实环境多轮操作、迭代精炼产物）——训练框架、环境管理、上层 agent CLI 三者割裂。

**核心方法**：端到端开源 agent 生产基础设施 **ALE（Agentic Learning Ecosystem）**，三大组件：**ROLL**（后训练框架，统一 SFT/DPO/GRPO/PPO + agent rollout 高吞吐流水线）+ **ROCK**（沙盒环境管理器，一键拉起 terminal/browser/code execution 容器并暴露统一 step API）+ **iFlow CLI**（上层 agent 框架）。基于 ALE 训练 ROME 模型，配套 **IPA（Interaction-Perceptive Agentic Policy Optimization）**——把 credit assignment 从"逐 token"提升到"语义交互块"粒度（一次 tool 调用 + 观察 = 一块），改善长程 trajectory 训练稳定性。100 万+ agent trajectory，发布 Terminal Bench Pro。

**关键结果**：完全开源（代码、权重、轨迹、环境镜像），IPA 显著改善长程训练稳定性、避免格式 token 主导梯度。GitHub github.com/alibaba/ROLL。

**意义**：ALE 填补"训练框架 + 环境管理 + 上层 CLI"三者割裂的空白，IPA 的"语义交互块粒度信用分配"与 GrandCode 的 Agentic GRPO、SDAR 的 token 门控同属"多轮信用分配精细化"主题。本专题作为开源 agent infra + RL 算法列出（非精读，亦见 `huggingface/00` W01）。

### 2026-02（20 命中 / 收录 6）

#### ⭐ #42 SkillRL: Evolving Agents via Recursive Skill-Augmented RL [UNC, 76↑] — arXiv 2602.08234

**问题动机**：LLM agent 常孤立运作、不从过去经验学习。已有 memory-based 方法主要存 raw trajectory——冗长、冗余、噪声大，agent 提不出"高层、可复用的行为模式"，泛化受限。

**核心方法**：SkillRL 用自动技能发现 + 递归进化连接 raw experience 与 policy 改进。① **Experience-based Skill Distillation**——teacher（OpenAI o3）蒸馏 base agent（Qwen2.5-7B）轨迹，**同时保留成功与失败**：成功轨迹提策略模式 s⁺、失败轨迹合成简洁教训 s⁻（识别失败点、错误推理、正确动作、预防原则），10–20× token 压缩；② **层级 SkillBank**——General Skills（探索/状态管理/目标追踪等通用策略）+ Task-Specific Skills（域启发式/前置条件/失败模式），每个 skill 有 name/principle/when_to_apply；③ **Adaptive Retrieval**——通用 skill 恒含、任务特定按语义相似度 TopK（K=6, δ=0.4）；④ **Recursive Skill Evolution**——先 cold-start SFT 教会用 skill（仅给 skill"收益有限"），SFT 后做 RL，验证 epoch 后仅在 Acc(C)<δ 处触发演化（diversity-aware 分层采样失败轨迹 → 生成新 skill），形成良性循环。RL 用 **GRPO**（二元奖励、组归一 advantage、PPO clip + KL，G=8、LR 1e-6）。

**关键结果**：**ALFWorld 89.9%（超 GRPO 77.6，+12.3%）、WebShop 72.7%（超 GRPO 66.1）**；超 GPT-4o 41.9%、Gemini-2.5-Pro 29.6%；子任务 Cool +23.0%/Pick2 +22.8%/Heat +15%。7 个搜索 QA 平均 **47.1%**（超 Search-R1 38.5、EvolveR 43.1），Bamboogle 超 EvolveR +19.4%。**Ablation**：去层级 −13.1%、去 skill 库（用 raw 轨迹）−25%、去 cold-start SFT −20%、去动态演化 −5.5%。**技能库增长**：55 skill（12 通用+43 特定）→ 100 skill（Step 150）；context ~1300 vs raw memory ~1450 token；>80% 成功仅需 60 步（vs 无演化 90 步）。

**意义**：SkillRL 解决 memory-based agent 的核心痛点——raw trajectory 冗余、提不出可复用模式——用"蒸馏成 skill + 递归进化"替代"存原始轨迹"，库与 policy 共进化避免脱节。与 SKILL0 [[15-skill0]] 形成"skill × RL"两条路（外部库共进化 vs 内化进参数）。与同组 Agent0 [[05-agent0]]（能力共进化）一脉。精读见 [[14-skillrl]]。GitHub github.com/aiming-lab/SkillRL。

#### ⭐ #43 Experiential Reinforcement Learning (ERL) [USC / Microsoft, 75↑] — arXiv 2602.13949

**问题动机**：环境反馈通常稀疏且延迟，LM 必须**隐式**推断"观察到的失败该如何转化为下次的行为改变"——这个隐式推断在长程任务极其低效。

**核心方法**：ERL 把 **experience-reflection-consolidation 循环**显式嵌入 RL。给定任务 x：① **首次尝试** y⁽¹⁾~π_θ(·|x)，得反馈 f⁽¹⁾、奖励 r⁽¹⁾；② **门控反思**（仅当 r⁽¹⁾<τ=1，即只对失败/次优轨迹反思）——生成 Δ~π_θ(·|x,y⁽¹⁾,f⁽¹⁾,r⁽¹⁾,m)，m 是跨 episode 反思记忆存成功修正模式；③ **第二次尝试** y⁽²⁾~π_θ(·|x,Δ)，反思奖励 r̃=r⁽²⁾，若 r⁽²⁾>τ 则存反思；④ **Consolidation（内化）**——用"selective distillation"只模仿成功的第二次尝试、且**从输入移除反思**：L_distill=−E[𝕀(r⁽²⁾>0)log π_θ(y⁽²⁾|x)]，训练模型仅凭原始 x 复现改进行为（部署零额外推理开销）。**门控关键**：对成功尝试反思会"鼓励 reward hacking"并让 off-policy 信号主导、破坏训练。RL 用 GRPO（rLLM 栈、8×H100、Olmo-3-7B/Qwen3-4B、LR 1e-6、KL 0.001、clip upper 0.28）。

**关键结果**：基准 FrozenLake/Sokoban（稀疏奖励控制）+ HotpotQA（agentic 推理）。**Sokoban Qwen3-4B 0.06→0.87（+81%，最大效应）**、Olmo3-7B 0.04→0.20；FrozenLake +27%（Qwen3 0.86→0.94）；HotpotQA +11%（0.45→0.56）。**Ablation**：完整 ERL > 无记忆 > 无反思（去反思跌幅最大，Sokoban −0.28）；但 Olmo3-7B Sokoban 上无记忆略胜完整 ERL（+0.04），说明自反思弱时持久记忆会传播早期坏反思。

**意义**：ERL 把"self-refinement"从推理时技巧转为训练时机制——成果内化、部署零开销。与 OpenClaw-RL [[03-openclaw-rl]] 的 directive signal 同源（都抽"该怎么改"，ERL 靠自反思、OpenClaw-RL 靠 next-state hindsight），与 DR Tulu 演化 rubric、MEDS 错误记忆构成"奖励信号丰富化"不同切面。"helps most in environments requiring substantial reasoning about unknown dynamics"。精读见 [[02-experiential-rl]]。GitHub github.com/microsoft/experiential_rl。

#### 📄 #5 Kimi K2.5: Visual Agentic Intelligence [Moonshot, 273↑] — arXiv 2602.02276 🔗

**问题动机**：如何做一个开源多模态 agent 旗舰，让视觉与文本互相增强、并支持自主多步 agentic 行为？

**核心方法**：Kimi K2.5 强调文本与视觉的联合优化。完整 pipeline：joint text-vision pretraining（pretrain 阶段就联合，避免 visual encoder 后挂式对齐瓶颈）+ **Zero-vision SFT**（SFT 阶段不直接拿视觉数据微调，因视觉 SFT 损害预训练视觉鲁棒性，改为先纯文本 SFT）+ **joint text-vision RL**（RL 阶段同时优化文本与视觉任务，让视觉能力在 RLHF 中重新激活强化）。Agent 创新 **Agent Swarm**——autonomous 并行编排，把任务分解为可并行子任务再调度多个 sub-agent，**延迟最高 -4.5×**。

**关键结果**：多模态 agent 评测（VisualWebArena/Mind2Web-Live/MMVet-Agent）开源 SOTA，接近 Claude 4.6 Sonnet。

**意义**：Kimi K2.5 把"多模态 + agent"做成可商用开源旗舰，joint text-vision RL 是其核心训练范式，Zero-vision SFT 是反直觉但被验证的关键技巧。详见 `huggingface/13-kimi-k25.md`，本专题作为旗舰模型（agentic RL 训练）交叉引用。

#### 📄 #22 GLM-5: from Vibe Coding to Agentic Engineering [Zhipu+清华, 151↑] — arXiv 2602.15763 🔗

**问题动机**：如何把"vibe coding"范式升级为"agentic engineering"，并证明开源旗舰可与闭源 frontier 同台竞技？

**核心方法**：GLM-5 是 744B 总参/40B-active MoE + MLA-256 + DSA 稀疏注意力，28.5T tokens 预训练。核心训练创新：① **异步 Agent RL**——基于自研 slime 框架，rollout 和 update 完全异步，512+ GPU 长时稳定运行；② **GRPO + IcePop 不对称 clip**——根据 advantage 符号采用不同 clip 阈值（positive 鼓励、negative 抑制）提升稳定性；③ 适配 7 个国产 GPU 平台。

**关键结果**：**BrowseComp 75.9**（断层领先所有开源，超 Claude Opus）+ **SWE-Verified 77.8** + **AAII v4.0 51.3**（首个 50+ 开源）。

**意义**：GLM-5 证明开源旗舰可与闭源 frontier 同台竞技，**异步 Agent RL + IcePop** 成为后续大模型训练事实标准。详见 `huggingface/06-glm-5.md`，本专题作为旗舰模型（agentic RL 训练）交叉引用——其异步 RL 与 OpenClaw-RL [[03-openclaw-rl]] 的异步架构、GrandCode 的 pipeline-RL 共同指向"rollout 与 update 解耦"成为大规模 agent RL 标配。

#### ⭐ #30 WideSeek-R1: Width Scaling via Multi-Agent RL [清华 / RLinf, 100↑] — arXiv 2602.04634

**问题动机**：LLM 能力扩展过去主要走 depth scaling（单 agent 多轮）。但任务变广时瓶颈从"个体能力"转为"组织能力"，现有多 agent 系统靠手工 workflow + 轮流交互、无法有效并行。

**核心方法**：提出 **width scaling（宽度扩展）**——用一个共享 LLM 实例化为 lead agent + subagents，但**隔离上下文 + 专用工具**。lead agent 用单一工具 `call_subagent`（故意受限以避免 context 污染），每轮生成明确子任务并行派发、闲置等所有 subagent 完成。subagent 用 `search`+`access` 两工具并行跑，结果去掉 thinking 后回传（Discard_think 保持 lead context 短）。**MARL 算法**基于 GRPO 两点设计：① **Multi-Agent Advantage Assignment**——可验证 outcome reward 组归一后，**同一 rollout 的所有 agent、所有 token 共享同一 advantage**（避免复杂 credit assignment 导致 reward hacking）；② **Dual-Level Advantage Reweighting**——token-level（跨 turn，跟 DAPO，长 turn 不被稀释）+ agent-level（"加 agent 只有提升最终 reward 才有益"，防止无意义 spawn）。Reward R_i=r_ans(Item F1)+r_format(0.1)+r_tool(0.05)−r_len。**20k 数据**用 gemini-3-pro 三阶段合成（HybridQA 抽 intent → 10–50 行 query → 自一致性过滤，阈值 0.9、去<3 行表，留存 73.28%，~$0.10/条）。

**关键结果**：WideSearch（200 任务，gpt-4.1 评）**WideSeek-R1-4B Item F1 40.0%（Max@4 51.8）**——比单 agent SingleSeek-R1-4B +11.9%、比多 agent 基础 Qwen3-4B +8.8%，4B/8B 中 6 项指标赢 5 项，**匹敌 DeepSeek-R1-671B（41.3）但参数少近 170×**。**Width scaling 行为**：depth scaling 快速饱和（受固定上下文限制）；base Qwen3-4B 宽度扩展到 10 subagent 时因冲突响应噪声**反而下降**；WideSeek-R1-4B 经 MARL 训练后**持续增益**，10 subagent 时达 40% Item F1。训练时 lead 每轮最多调 10 个并行 subagent。

**意义**：WideSeek-R1 提出与"深度"正交的"宽度"scaling 维度，证明 MARL 能训出真正的并行组织能力（而非 prompt 拼装）。与 HACRL [[08-hacrl]]、MATTRL [[09-mattrl]] 构成 MARL 三种新解法——WideSeek-R1 占"宽度/并行组织"一极。与 IterResearch [[11-iterresearch]]（深度）、SearchSwarm（委派）共同构成 DR scaling 三个新轴。精读见 [[10-wideseek-r1]]，亦是 `deep-research/12`。GitHub via RLinf。

#### 📄 #32 Composition-RL: Compose Your Verifiable Prompts for RLVR [Tencent-Hunyuan, 94↑] — arXiv 2602.12036

**问题动机**：大规模可验证 prompt 支撑 RLVR 成功，但含大量无信息样本、且进一步扩展昂贵。如何更好地利用有限训练数据？

**核心方法**：Composition-RL **组合可验证 prompt** 来提升 RLVR 的数据效率——把现有可验证 prompt 按某种策略组合成新的、更有信息量的训练样本（而非单纯采集更多 prompt），在有限数据下榨取更多训练信号。属于 RLVR 的"数据侧"优化。

**关键结果**：在 RLVR 训练中以更高数据效率提升性能（具体见论文）。GitHub github.com/XinXU-USTC/Composition-RL。

**意义**：Composition-RL 与 F-GRPO（rare 样本）、GrandCode 的 reward 设计同属"RLVR 训练数据/信号侧精细化"，但聚焦"如何组合现有可验证 prompt 提升数据效率"。本专题作为 RLVR-data 工作列出（非精读），与 agentic RL 的关联在于其 tool-use rollout 与可验证奖励的数据构造。

### 2026-03（12 命中 / 收录 5）

#### ⭐ #6 Heterogeneous Agent Collaborative Reinforcement Learning (HACRL) [Beihang 等, 198↑] — arXiv 2603.02604

**问题动机**：传统 multi-agent RL 要么同构 agent（共享参数）要么单向蒸馏（强→弱）。能否让**异构 agent**（不同架构/规模/专长）在训练时双向互学、推理时各自独立？难点是异构带来的能力差异 + 策略分布漂移会让朴素 advantage 共享有偏。

**核心方法**：HACRL 是新的 RLVR 范式——异构 agent **训练时共享已验证 rollout 互相提升、推理时独立**（n-agent 系统每条 rollout 可复用 n 次）。三类异构：Heterogeneous State（同架构不同权重）/ Size（同族不同规模）/ Model（不同架构/tokenizer/目标）。算法 **HACPO** 四机制：① **Agent-Capability-Aware Advantage Estimation**——用能力调整的 baseline，A=（R−μ）/std，μ 把其他 agent 的奖励按能力比 ω^{k,j}=P^k/P^j 重加权；② **Model Capabilities Discrepancy Coefficient**——能力比也调制梯度（放大强 agent 信号、衰减弱 agent）；③ **Exponential Importance Sampling**——在 GSPO 序列级 ratio 上做跨 agent 非梯度指数重加权（tokenizer 不兼容则 detokenize+retokenize，Qwen/Llama 在 MATH 上 token 数仅差 ~4%）；④ **Stepwise Clipping**——cross-agent ratio（约[0.8,1.0]）与 self-ratio（约[0.9997,1.0004]）行为不同，bound 在 step 内渐紧。**理论**：oracle 能力感知 baseline 精确无偏（Theorem D.5：E[μ*]=p^k），有限 batch 给高概率误差界（Hoeffding+union bound）。

**关键结果**：7 个数学基准（MATH-500/MATH/GSM8K/AIME2025/AMC23/Minerva/Olympiad），比 **GSPO×2（双倍 rollout）平均 +3.6%、计算仅一半**。典型：Qwen3-4B State 0.691→0.755（+0.064）、AMC23 0.675→0.85、AIME2025 0.522→0.622；Llama3.2-3B Model 0.334→0.390（+0.056）、AMC23 0.175→0.35。5 seed 可复现（Qwen3-1.7B-Base MATH500 +3.56%）。**Ablation**：去 Advantage Estimation 0.601→0.586；去 Stepwise Clipping → 严重不稳定；去 stepwise → 次优收敛。即使 Heterogeneous State 设定，弱 agent 也提供"alternative reasoning paths and informative errors"使强 agent 受益（非单向）。基于 verl，目前仅评测两 agent（基建限制，n≥3 原理上支持）。

**意义**：HACRL 把"协作"从同构走向异构（不同模型架构/规模/专长），是 multi-agent RL 工程化的重要一步。"训练时协作、推理时独立"是实用解耦——既要协作红利又避开在线多 agent 协调成本。无偏 advantage 共享的理论保证为"异构 rollout 复用"提供地基。与 MATTRL [[09-mattrl]]、WideSeek-R1 [[10-wideseek-r1]] 构成 MARL 三种新解法。与 GrandCode 的 Agentic GRPO、SDAR 共享"多轮/多 agent 信用分配精细化"主题。精读见 [[08-hacrl]]。

#### ⭐ #12 OpenClaw-RL: Train Any Agent Simply by Talking [Princeton, 156↑] — arXiv 2603.10165

**问题动机**：每次 agent 交互都产生 **next-state signal**（用户回复/工具输出/终端 GUI 状态变化），但没有现有 agentic RL 系统把它当作实时在线学习源。且 next-state 里既有评价信息也有改进方向，后者一直被浪费。

**核心方法**：OpenClaw-RL 基于"next-state 信号通用、policy 可对所有信号同时学"。两类信号：**Evaluative**——PRM 每轮查 m 次投票 {+1,−1,0}，多数票为标量奖励 r_t（re-query/纠正请求 = 负反馈）；**Directive**——**Hindsight-Guided OPD**：PRM 检测到有意义纠正时把 s_{t+1} 蒸成 hint h（[HINT_START]…[HINT_END]），拼成 s_t^h，取同模型 teacher 分布 π_T(·|s_t^h)，对每个词 v 算 w_v=softmax(ℓ_old)、Δ_v=clip(ℓ_{T,h}−ℓ_old,−C,C)、**A_v=Δ_v·w_v**（w_v 把 advantage 集中到学生可能采样的 token、clip bound 每 token log-prob 差）。**混合目标** L^hybrid=w_RL·L^GRPO+w_OPD·L^OPD（默认各 1）。**Overlap-Guided Hint Selection**——按 top-k token 重叠 O[h,i]=|S_i^q∩S_{i,h}^p| 选 hint（序列级在 agentic RL 更稳）。**Log-prob-difference clipping**——不 clip 时 Retool 设定截断率 0.5、clip 后 0.2。**全异步架构**：环境 server / PRM judge / Megatron 训练 / SGLang 服务四者完全解耦、基于 slime，请求分 main-line turn（可训）与 side turn（转发不训），权重在同步边界推送、用户不等待。

**关键结果**：**个人 agent**（Qwen3-4B-Thinking，达优化所需最少 session，越低越好）Hybrid 平均 **10.3** vs GRPO 14.1、OPD 29.7、Mem0 14.5、Cognee 14.9——Hybrid 显著最优。**Qwen3-32B ablation**：Seq-optimal 12.5 / Token-optimal 12.4 / Random 16.1 / GRPO 15.8 / OPD 29.9——证明 hint 选择关键、token-level OPD 急剧退化。**通用 agentic RL**（PRM+Outcome vs Outcome-Only）：Tool-call 0.25 vs 0.19、GUI 0.33 vs 0.31。k ablation：k≥4 饱和（默认 k=4）。并行环境 128（terminal）/64（GUI/SWE）/32（tool-call）。

**意义**：OpenClaw-RL 把"训 agent"从"专家手工定义 reward"降到"普通对话/交互即监督"，是 agent democratization 代表。**统一信号视角**——首次明确 terminal/GUI/SWE/对话是同一类 next-state 学习问题，是"首个统一覆盖这些领域的真实 agent RL 框架"。directive 信号与 ERL [[02-experiential-rl]] 殊途同归（都抽"该怎么改"）。异步 RL 与 GLM-5 的 slime、GrandCode 的 pipeline-RL 共同指向"rollout 与 update 解耦"成标配。精读见 [[03-openclaw-rl]]。GitHub github.com/Gen-Verse/OpenClaw-RL。

#### ⭐ #37 CUDA Agent: Large-Scale Agentic RL for CUDA Kernel Generation [ByteDance Seed, 99↑] — arXiv 2602.24286

**问题动机**：GPU kernel 优化是最专家化的系统软件任务，LLM 在通用编程强、但写 CUDA 一直打不过基于编译器的系统（torch.compile）。如何用 agentic RL 反超？

**核心方法**：三件套。① **数据合成 pipeline（CUDA-Agent-Ops-6K）**——Seed Crawling（从 torch/transformers 挖参考算子）→ Combinatorial Construction（LLM 采样 ≤5 个算子堆叠成融合多算子任务，因为"组合问题往往不等于逐个优化再拼接"）→ Filtering（Eager+Compile 都能跑、无随机性、anti-hacking、eager 1–100ms），ground-truth 用每题 5 个随机输入验证（KernelBench 协议），AST 相似度 >0.9 去污。② **技能增强 CUDA 环境**——基于 OpenHands ReAct loop，SKILL.md 规范"profile.py 分析→model_new.py 实现→GPU 沙箱编译评估→迭代直到比 torch.compile 快 ≥5% 且数值正确"，**奖励 r∈{−1,1,2,3}**（正确性失败 −1、同时快过 eager 和 compile 3、快过 eager 2、否则 1），五重 anti-hacking。CPU-GPU 解耦——Docker 编译 + **128 张 H20 GPU** 验证/profiling。③ **高方差奖励稳定化**——根因：初版只稳训 17 步（CUDA 代码 <0.01% 预训练数据、token 概率近精度下限、BF16/FP16 不匹配让 importance ratio 爆炸）。**多阶段 warm-up**（稳训到 200 步）：Single-Turn Warm-up（PPO）→ Actor Init（RFT 只留正奖励轨迹）→ Critic Init（Value Pretraining，γ=1/λ=0.95）。RL 用 PPO 非对称 clip（ε_low=0.2/ε_high=0.28），Base=Seed1.6（23B active/230B MoE）。

**关键结果**：KernelBench **Faster Rate vs torch.compile：L1 97.0%/L2 100%/L3 90%**（overall 96.8%），远超 Claude Opus 4.5（66.4%）/Gemini 3 Pro（69.6%）；**L3 90% vs ~50–52%，约 +40%**。Geomean 加速 overall 2.11×（vs 1.46×/1.42×），L2 达 2.80×。Pass Rate L1/L2 100%、L3 94%。**Ablation**：w/o Agent Loop 96.8%→14.1%、w/o Robust Reward →60.4%、w/o RFT →49.8%（去 RFT"奖励灾难性崩溃 + 熵激增"）、w/o Value Pretraining →50.9%（"轨迹长度爆炸"）。注：GPT-5 系列"一律拒绝回应 CUDA 提示"未评测。

**意义**：CUDA Agent 证明 agent 可在系统软件开发（CUDA 这种最专家化领域）超越 frontier 闭源，预示 AI 自动优化 GPU 代码成为新现实。与 GrandCode 一道确立"**有可靠自动 verifier 的领域，agentic RL 能反超闭源**"——把可验证性作为成功核心前提。与 EvoCUA [[06-evocua]]/Agent-World 共享"可验证环境 + 大规模 agentic RL"范式跨域复制。精读见 [[07-cuda-agent]]。GitHub github.com/BytedTsinghua-SIA/CUDA-Agent。

#### 📄 #24 MetaClaw: An Agent That Meta-Learns and Evolves in the Wild [UNC, 140↑] — arXiv 2603.17187

**问题动机**：部署的 LLM agent 往往保持静态、不随用户需求演化，造成"持续服务 vs 更新能力"的张力——现有方法要么囤积 raw 轨迹、要么用静态 skill 库、要么强制中断停机重训。

**核心方法**：MetaClaw 是**持续 meta-learning 框架**，**联合演化 base LLM policy + 可复用 skill 库**，两个互补机制：① **Skill-driven fast adaptation**——"LLM evolver"检视失败轨迹合成新 skill，**零停机**立即改进；② **Opportunistic policy optimization**——通过 cloud LoRA fine-tuning + **RL-PRM（带 Process Reward Model 的 RL）** 做梯度更新（PRM 给中间推理步打分，而非只奖励最终结果）。二者互相强化（更好 policy 产更好轨迹供 skill 合成，更丰富 skill 喂更高质量数据回 policy 优化）。**OMLS（Opportunistic Meta-Learning Scheduler）** 监控系统不活跃 + 日历数据，在用户空闲窗触发重更新；versioning 分离 support/query 数据防污染；proxy 架构让它"无需本地 GPU 即可扩到生产级 LLM"。

**关键结果**：skill-driven 适应**最高 +32% 相对增益**；Kimi-K2.5 准确率 **21.4%→40.6%**；复合鲁棒性 **+18.3%**。基准 MetaClaw-Bench、AutoResearchClaw。平台处理"20+ 渠道的多样工作负载"。13 作者。

**意义**：MetaClaw 把"部署中持续演化"工程化——零停机 skill 适应 + 空闲窗机会式 policy 优化（RL-PRM）。它与 OpenClaw-RL [[03-openclaw-rl]]"被使用即训练"、同组 Agent0 [[05-agent0]]/SkillRL [[14-skillrl]] 一脉，是"agent 在真实部署中自演化"的代表。本专题作为 agent_train 简述。GitHub github.com/aiming-lab/MetaClaw。

#### 📄 #48 SWE-rebench V2: Language-Agnostic SWE Task Collection at Scale [Nebius, 89↑] — arXiv 2602.23866

**问题动机**：SWE agent 的近期进步很大程度由 RL 驱动，但 **RL 训练受限于"大规模、可复现执行环境、可靠奖励"的任务集稀缺**——尤其跨多语言。

**核心方法**：SWE-rebench V2 是**语言无关、大规模的 SWE 任务集**——为 SWE agent RL 提供可复现执行环境 + 可靠奖励信号的任务。它把真实 issue/repo 自动挖掘、构建可执行验证环境、覆盖多种编程语言，解决 RL 训练 SWE agent 的数据瓶颈。

**关键结果**：提供大规模、语言无关、可复现的 SWE RL 任务集（具体规模见论文）。GitHub github.com/SWE-rebench/SWE-rebench-V2。

**意义**：SWE-rebench V2 是 SWE agent RL 的数据基建——RL 训练的瓶颈是"任务稀缺 + 环境不可复现 + 奖励不可靠"，它三者一并解决并扩到多语言。与 Agent-World、CUDA Agent [[07-cuda-agent]] 的可验证环境合成同属"为 agentic RL 造可验证任务/环境"主线。本专题作为 agent_train 数据侧简述。

### 2026-04（16 命中 / 收录 5）

#### 📄 #1 GrandCode: Grandmaster Competitive Programming via Agentic RL [Deep Reinforce, 632↑] — arXiv 2604.02721 🔗

**问题动机**：竞赛编程是人类对抗 AI 的最后堡垒之一——此前最强 AI（Gemini 3 Deep Think）仅第 8 名且非直播。能否让 AI 在 Codeforces 直播赛击败所有人类？

**核心方法**：GrandCode 是 4 组件多 agent RL 系统：**π_main**（Qwen-3.5-397B 主求解）+ **π_hypothesis**（27B 提结构性猜想，小样例 brute-force 验证，r_verify=通过测试占比，联合训练加下游项 β(S(x,h)−S(x,∅))）+ **π_summary**（27B 压缩超长 reasoning）+ **Test Case Generator**（adversarial/solution-attack/large-size）。核心算法 **Agentic GRPO 两阶段更新**：对 multi-stage rollout s_1,r_1,…,s_N,r_N——**Immediate Reward**（r_t 一可得就更新，A_t=（r_t−μ_t）/σ_t）+ **Delayed Correction**（r_N 到时 δ_t=r_N−r_t，A_t=（δ_t−μ_t）/σ_t）。**三阶段 reward**：Executability（编译失败 0）→ Correctness（输出≠参照 0）→ Efficiency（相对 brute-force 加速比，超时 0.1）。Hypothesis-Guided Generation 可查 OEIS。

**关键结果**：**Codeforces Round 1087/1088/1089 直播全部第一、全部最先完成**。100 题 benchmark：full RL **81% accept/72.3 加权**、+test-time RL **85%/73.5**——超 Gemini 3.1 Pro（64.3）/Claude Opus 4.6（63.7）/GPT-5.4（63.0）。Ablation 进程：base 52.2 → +continue 61.0 → +SFT 62.5 → +Full RL 72.3 → +Test-time RL 73.5。Hypothesis pass@1：34%→52%；测试用例生成 42→48→50。

**意义**：GrandCode 证伪"AI 永远做不过人类竞赛选手"，**Agentic GRPO 成为后续 SDAR/DelTA 等的新基线**——本专题 RL 算法的旗舰参照。"agentic RL + 强验证 + 在线适应能把编程系统推过最强人类"。详见 `huggingface/01-grandcode.md`，本专题交叉引用。

#### 📄 #23 MEDS: Memory-Enhanced Dynamic Reward Shaping [144↑] — arXiv 2604.11297

**问题动机**：RL 训练 LLM 的一个常见失败模式是**采样多样性下降**——policy 反复生成相似的错误行为。经典 entropy 正则鼓励随机性，但**不显式惩罚跨 rollout 反复出现的错误模式**。

**核心方法**：**MEDS（Memory-Enhanced Dynamic reward Shaping）** 把历史行为信号纳入 reward 设计。① 存储并利用历史 rollout 的**中间模型表示**捕捉过去 rollout 的特征；② 用**密度聚类**识别频繁复现的错误模式（如某种数学陷阱反复掉进去）；③ 落入高频错误簇的新 rollout **加重惩罚**——形成"教训越深、避免越急"的动态 reward shaping。

**关键结果**：5 数据集 × 3 base 模型，**pass@1 +4.13、pass@128 +4.37**——单次和多次采样都提升，说明既学到正确答案又增加多样性。GitHub github.com/Linxi000/MEDS。

**意义**：MEDS 把"经验"（历史错误）作为 reward shaping 信号，避免 RL 训练陷入"局部最优 + 重复犯错"死循环。它与 ERL [[02-experiential-rl]]（反思记忆）、SDAR（门控）同属"用 token/历史级统计定位并修正 RL 问题"，是 RLVR 工具箱的实用补丁。本专题作为 rl_algo 简述（亦见 `huggingface/00` W16）。

#### 📄 #46 Agent-World: Scaling Real-World Environment Synthesis [ByteDance Seed, 85↑] — arXiv 2604.18292

**问题动机**：LLM 越来越被期望作为通用 agent 与外部有状态工具环境交互，但**训练受限于真实环境稀缺 + 缺乏有原则的终身学习机制**。MCP 与 agent skill 提供了统一接口，但如何规模化合成可验证环境？

**核心方法**：Agent-World 是**自演化训练竞技场**，通过可扩展环境推进通用 agent 智能。两组件：① **Agentic Environment-Task Discovery**——agent **自主探索数千真实环境主题的主题对齐数据库 + 可执行工具生态**，合成**可验证、难度可控**的任务；② **Continuous Self-Evolving Agent Training**——多环境 RL + **自演化竞技场**（自动通过动态任务合成识别能力缺口），实现 **agent policy 与环境的协同进化**。

**关键结果**：23 个高难 agent 基准，**Agent-World-8B/14B 一致超越强专有模型和环境 scaling 基线**；分析显示性能随环境多样性和自演化轮数的 scaling 趋势。20 作者（Guanting Dong 等）。

**意义**：Agent-World 把"自演化环境合成"工程化为基础设施级 capability——agent 自己发现数据/工具、合成可验证任务、在竞技场识别缺口、policy 与环境共进化。它把 TermiGen/EvoCUA [[06-evocua]] 的"环境合成"推到大规模通用化，与 DreamGym [[01-dreamgym]]（合成经验）共同代表"为 agentic RL 造可验证环境/经验"主线。本专题作为 agent_train 简述。

#### 📄 #49 OpenGame: Open Agentic Coding for Games [81↑] — arXiv 2604.18394

**问题动机**：游戏开发处于创意设计与复杂软件工程的交叉点，需要协调游戏引擎、实时循环、跨多文件紧耦合状态。LLM/code agent 能解孤立编程任务，但被要求从高层设计产出**完整可玩游戏**时持续翻车——崩于跨文件不一致、场景接线断裂、逻辑不连贯。

**核心方法**：OpenGame 是**首个端到端网页游戏创建 agentic 框架**。核心是 **Game Skill**（可复用、可演化能力）：**Template Skill**（从经验积累项目骨架库，scaffold 稳定架构）+ **Debug Skill**（维护"已验证修复的活协议"，系统性修复集成错误而非补孤立语法 bug）。模型 **GameCoder-27B** 三阶段：**CPT（持续预训练）→ SFT → 执行驱动 RL（execution-grounded RL）**。评测 **OpenGame-Bench**——因"验证交互可玩性比检查静态代码本质上更难"，用无头浏览器执行 + VLM 评判三维度：**Build Health / Visual Usability / Intent Alignment**。

**关键结果**：150 个多样游戏 prompt 上 **OpenGame 建立新 SOTA**（具体分数见论文）。

**意义**：OpenGame 把 coding agent 从"补全片段"推到"创造完整产品"，是 generative game 领域代表。其"Template/Debug Skill + 执行驱动 RL"展示了 agentic RL 如何训练"产出完整可运行应用"的能力。与 OpenGame-Bench 的"无头浏览器 + VLM 三维评判"对"难验证产物"的评测有借鉴。本专题作为 agent_train 简述。GitHub github.com/leigest519/OpenGame。

#### ⭐ #39 SKILL0: In-Context Agentic RL for Skill Internalization [浙大, 101↑] — arXiv 2604.02268

**问题动机**：agent skill（推理时动态加载的过程知识包）已成 augment LLM agent 的可靠手段，但 inference-time 加载有三大硬伤——检索噪声、token 开销、模型只"照做"未真正习得（"能力在 context 不在 model"）。

**核心方法**：SKILL0 用 **in-context RL 把 skill 内化进参数**（口号"Skills at training, zero at inference"）。**ICRL**——训练 rollout 时给 skill 作 in-context 指导、推理时完全移除，让 RL 优化直接驱动"从 context 依赖到自主行为"的转变。skill 存成 Markdown（skills/{task}/{category}.md），**context 渲染**把历史+skill 映射成紧凑 RGB 图像由视觉编码器编码 V_t=Enc(h_t,S;c_t)，policy 自生成压缩比 c_t。**复合奖励** r̃_t=r_t+λ·r^comp_t，r^comp_t=ln(c_t)（成功时，反映压缩边际递减）。RL 用 **GRPO**（组归一 advantage + clip + KL）。**动态课程（线性衰减预算）**：|S^(s)|≤M^(s)=⌈N·(N_S−s)/(N_S−1)⌉，逐步减到 ∅（完全自立），保持分布偏移平滑。两阶段：Relevance-Driven Skill Grouping（离线把验证集分成 N 子任务各配一个 skill）+ Helpfulness-Driven Dynamic Curriculum（在线按 Δ_k=Acc^{w/skill}_k−Acc^{w/o skill}_k 每 d 步 Filter（留 Δ_k>0）/Rank/Select top-M）。helpfulness 曲线呈"先升后降"（skill 是"有效但短暂的脚手架"）。

**关键结果**：3B vs AgentOCR baseline——**ALFWorld +9.7（87.9）/Search-QA +6.6（40.8）/WebShop +10.1 准确率（66.4%）**；7B：ALFWorld 89.8/Search-QA 44.4/WebShop 85.1。**token <0.5k/步**（3B ALFWorld 0.38k/Search-QA 0.18k/WebShop 0.49k，比 SkillRL 的 2.21k/0.87k 低 >5×）。**Ablation**：预算[6,3,0]在移除 prompt 时"Fixed Full 和[6,6,6]崩 −12.3/−13.3"而 SKILL0 +1.6；"Filter&Rank&Select"是唯一正迁移（Δ=+1.6%），w/o Rank（随机）严重崩塌（Δ=−13.7%）。Qwen2.5-VL 3B/7B，4×H800，SkillBank 从 SkillRL 初始化。

**意义**：SKILL0 把 skill 从 inference-time 外挂转为 parameter 内化，同时解决检索噪声/token 开销/"只照做未习得"三大问题。**动态课程（逐步撤除 skill 上下文）** 是可复用的知识内化技巧。与 SkillRL [[14-skillrl]] 形成"skill × RL"两条路（外部库共进化 vs 内化进参数）。与 SkillsBench 实证呼应（curated skill 有效→SKILL0 把它内化）。精读见 [[15-skill0]]。GitHub github.com/ZJU-REAL/SkillZero。

### 2026-05（18 命中 / 收录 4）

#### 📄 #26 DVAO: Dynamic Variance-adaptive Advantage Optimization for Multi-reward RL [134↑] — arXiv 2605.25604

**问题动机**：RL 已成对齐 LLM 的标准范式，GRPO 是 value-model-free 的高效 PPO 替代，但**适配到多奖励设定很难**：标准 scalarization 有两个失败模式——Reward Combination 产生"平方幅度过大的 advantage"导致不稳定，Advantage Combination "依赖静态超参且忽略跨目标相关性"。

**核心方法**：**DVAO** 用 rollout 组内每个目标的**经验奖励方差**动态调节组合权重——**上调学习信号强的目标、抑制噪声大的**。数学证明 DVAO 保持**有界 advantage 幅度**以稳定训练，并加**自适应跨目标正则**机制。区别于 GRPO 的静态 scalarization，方差自适应加权同时解决 Reward Combination 的幅度爆炸和 Advantage Combination 的僵化，并考虑目标间相关性。

**关键结果**：数学推理 + **tool-use** 基准（Qwen3/Qwen2.5），DVAO"显著超过基线方法"，达到"更优的多目标 Pareto 前沿 + 鲁棒训练稳定性"（具体数字见论文）。

**意义**：DVAO 把 GRPO 从单奖励推到多奖励设定，用方差自适应解决多目标 RL 的稳定性。它与 GrandCode 的三阶段 reward、HACRL [[08-hacrl]] 的能力感知 advantage 同属"RL advantage/reward 设计精细化"，但聚焦"多奖励组合的方差自适应"。其 tool-use rollout 与 agentic RL 直接相关。本专题作为 rl_algo 简述。

#### 📄 #36 Self-Distilled Agentic RL (SDAR) [浙大+美团+清华, 112↑] — arXiv 2605.15155 🔗

**问题动机**：RL 的轨迹级奖励信号对长程多轮交互只提供粗监督。On-Policy Self-Distillation（OPSD）在 single-turn 推理 RL 有效，但**multi-turn agent 训练时不稳定**——学生一旦偏离 teacher 支持的轨迹，token 级监督就不可靠，per-turn KL 激增、性能灾难性退化。

**核心方法**：SDAR 保留 **GRPO 为主干**、把 OPSD 当**门控辅助损失** L=L_GRPO+λ·L_SDAR。诊断关键——teacher 是同一 policy 加特权上下文（检索的 skill）而非独立更强模型，**positive gap 可靠、negative gap 模糊**，且"negative-gap token 超过 50%"（Qwen2.5-3B），朴素蒸馏有害。**Teacher-Student log-prob gap** Δ_t=sg(log π^+_θ−log π_θ) 在学生采样 token 上算，**sigmoid 门** g_t∈(0,1) 调制每 token（sharpness β）。三种门控：Entropy（高不确定位）/ **Gap（默认最优，直接测 teacher 分歧，上调 positive、衰减 negative）**/ Soft-OR。门用 stop-gradient detach（梯度只过学生 log-prob），证明辅助梯度有界防爆炸；用 **reverse KL**（mode-seeking，自然降权低 teacher 概率 token）。

**关键结果**：超 GRPO（Qwen2.5-3B）：**ALFWorld +9.4（84.4）/Search-QA +7.0（43.4）/WebShop-Acc +4.7（68.0）**，7B WebShop-Acc +10.2。**稳定性**：单独 OPSD"灾难崩溃（Search-QA 近零）"，朴素 GRPO+OPSD 在 Qwen3-1.7B"严重退化（32.0 vs 46.1，无界蒸馏梯度）"。**技能内化**：SDAR 推理时无需 skill 却超 skill-augmented Skill-GRPO*（ALFWorld-1.7B 53.9 vs 28.1）。**Random-Retrieval 发现**：零任务感知的随机检索也超 GRPO（+1.9/+1.6/+1.0），证明增益来自门控蒸馏而非检索保真。Ablation：β=5、λ=0.01 最优，Gap gating 胜 entropy/soft-OR，reverse KL 胜 forward/JSD。

**意义**：SDAR 把 GRPO 系列从 single-turn 推到 multi-turn agent 训练的标准方案——sigmoid 门控的非对称信任（positive 强化、negative 软抑制）。与 GrandCode 的 Agentic GRPO、DelTA 并列 2026 H1 RLVR for agent 三巨头。详见 `huggingface/08-sdar.md`，本专题作为 rl_algo 交叉引用。GitHub github.com/ZJU-REAL/SDAR。

#### 📄 #37 Skill1: Unified Evolution of Skill-Augmented Agents via RL [112↑] — arXiv 2605.06130 🔗

**问题动机**：持久 skill 库让 agent 跨任务复用成功策略，但维护这种库需要三个耦合能力——**选择**相关 skill、执行时**利用** skill、从经验**蒸馏**新 skill。如何用一个 RL 框架统一这三者？

**核心方法**：Skill1 用**单一 policy 同时进化 skill selection / utilization / distillation 三个能力**。流程：agent 生成查询搜 skill 库 → skill 重排选最相关 → 条件求解（用 skill 解任务）→ 成功后蒸馏新技能加入库。低频任务归功于 selection（找对 skill 就赢），高频任务归功于 distillation（创造新 skill 解锁更广任务）——把整个 skill lifecycle 用一个 reward 信号统一驱动。

**关键结果**：ALFWorld + WebShop 上超过先前 skill 工作（具体数字见论文）。

**意义**：Skill1 避免"skill selector/executor/extractor 各训各的"碎片化，把 skill lifecycle 用一个 reward 统一。与 SkillRL [[14-skillrl]]（递归进化）、SKILL0 [[15-skill0]]（内化）、SkillClaw（生态级）、SkillOpt（text-space SGD）构成完整 skill × RL 谱系。详见 `huggingface/00` W19，本专题作为 skill 交叉引用。

#### 📄 #44 OpenSearch-VL: Open Recipe for Multimodal Search Agents [腾讯混元, 102↑] — arXiv 2605.05185 🔗

**问题动机**：top-tier 多模态搜索 agent 仍难复现——缺乏完全开源的训练 recipe（数据 + 算法 + 权重）。

**核心方法**：完全开源的 frontier 多模态 deep search agent recipe（agentic RL）。**数据 pipeline**：Wikipedia path sampling（知识图谱路径生成多跳问题）+ fuzzy entity rewriting（模糊实体改写降低表面匹配作弊）+ source-anchor visual grounding（减少 shortcut）→ SearchVL-SFT-36k + SearchVL-RL-8k。**工具环境**：统一 text/image search + OCR + cropping/sharpening/super-resolution/perspective correction。**算法——multi-turn fatal-aware GRPO**：mask 失败后 token（避免错误污染后续）+ one-sided advantage clamping（保留失败前有用推理）。

**关键结果**：7 个多模态搜索基准平均 **+10pt+**，多任务匹敌商用模型。GitHub github.com/shawn0728/OpenSearch-VL（215★）。

**意义**：OpenSearch-VL 把"开源多模态 DR"整套 pipeline（数据/算法/权重）公开。其 fatal-aware GRPO（失败后 token 掩码 + 单侧 advantage 钳制）是多轮 agentic RL 的实用技巧。详见 `huggingface/00` W19 与 `deep-research/08`，本专题作为 deep_research RL 交叉引用。

#### 📄 #22 AgentDoG 1.5: Alignment Framework for AI Agent Safety [上海AI Lab, 142↑] — arXiv 2605.29801

**问题动机**：OpenClaw 等开放世界 agent 有强大跨环境执行力，却引入广泛新安全风险源；前沿 AI 大幅降低攻击门槛，使现有 agent 对齐框架不足以真实部署。

**核心方法**：**轻量、可扩展的 agent 安全对齐框架**。① 更新 agent 安全 **taxonomy** 以容纳 Codex/OpenClaw 执行场景的新兴风险；② **taxonomy 引导的数据引擎 + influence-function 净化**，仅用约 **1k 样本**训出 AgentDoG 1.5（0.8B/2B/4B/8B），达到与 GPT-5.4 等闭源相当的性能；③ 构建高效 **agentic 安全 SFT + RL 训练环境**，把 Docker 级部署开销降低**两个数量级（~100×）**；④ 部署为 **training-free 在线护栏**做实时安全审核。

**关键结果**：与领先闭源（GPT-5.4）相当，Docker 部署开销 **−100×**，在多样复杂交互 agentic 场景达 SOTA。模型/数据全开源。50 作者（Dongrui Liu 等），44 页。

**意义**：AgentDoG 1.5 把 agent 安全对齐做成轻量、可扩展、含 RL 训练环境的框架，应对开放世界 agent（OpenClaw/Codex）的新兴安全风险。它是本专题唯一的 safety_eval 类工作，与 `huggingface/` 的 AgentDoG/ClawKeeper 安全工作一脉，其"agentic 安全 SFT+RL 训练环境"把 RL 引入安全对齐。本专题作为 safety_eval 简述。

### 2026-06（13 命中 / 收录 6）

#### ⭐ #45 Harness-1: RL for Search Agents with State-Externalizing Harnesses [UIUC / Chroma, 52↑] — arXiv 2606.02373

**问题动机**：搜索 agent 通常被训成"在不断增长的 transcript 上的 policy"——模型既要决定"怎么搜"，又要记住"看过什么、哪些证据有用、哪些约束未满足、哪些 claim 已核实"。这是把**太多例行状态管理塞进 policy**，逼 RL 同时学"语义搜索决策"和"可恢复的记账"，样本效率低、泛化差。

**核心方法**：Harness-1 是 **20B 检索 subagent**，在 **stateful search harness** 内用 RL 训练。harness 在**环境侧**维护 working memory：**candidate pool**、**importance-tagged curated set**、**compact evidence links**、**verification records**（哪些 claim 已核查）、**压缩去重的 observations**、**budget-aware context rendering**（按 token 预算决定给 policy 看什么）。policy 只保留真正需要智能的**语义决策**：搜什么、留/弃哪些文档、验证什么、何时停。核心论点——RL 否则被迫同时优化"语义检索决策"和"可恢复的记账"，把后者外置让 RL 集中火力在前者。作者 Jiawei Han 组（UIUC）+ Chroma（向量数据库公司）。

**关键结果**：8 个检索基准（web/finance/patents/multi-hop QA）平均 curated recall **0.730**，比次优开源 subagent **+11.4pt**；在 **held-out transfer（未见过的迁移基准）上增益尤强**——说明"语义决策"比"记账"更可迁移（记账是任务特定的，语义能力通用）。GitHub 548★。

**意义**：Harness-1 给出可操作的设计原则——**把"可恢复的状态"外置给环境，只让 RL 优化"不可恢复的语义决策"**，对所有长程 agent（不只搜索）有借鉴意义，是"harness engineering"（系统级设计与权重训练同等重要）的精确实证。与 LLM-in-Sandbox [[04-llm-in-sandbox]] 形成"环境职责切分"两极（做薄 vs 做厚），与 IterResearch [[11-iterresearch]]"policy 周期性重建"是长程状态管理的两条路。精读见 [[13-harness-1]]，亦是 `memory/01-harness-1`、`deep-research/09` 核心。GitHub github.com/pat-jj/harness-1。

#### 📄 #12 GrepSeek: Training Search Agents for Direct Corpus Interaction [UMass, 106↑] — arXiv 2605.29307 🔗

**问题动机**：传统 retriever 把语料压缩成"固定 top-k 相似度接口"，对 agentic search 是瓶颈——精确词约束、稀疏线索合取、被早期滤掉的证据都无法被下游推理找回。如何用 RL 训练 agent 直接用 shell 与语料交互？

**核心方法**：GrepSeek 让 search agent 把语料本身当搜索环境，用可执行 shell 命令（grep/find/sed/awk）找证据。**两阶段训练**：① cold-start 数据集——用 **answer-aware Tutor（知答案的导师解释 shell 用法）+ answer-blind Planner（不知答案的规划器拆解检索意图）** 生成 verified、因果 grounded 的轨迹；② **GRPO** refine policy（reward = 找到正确答案 + 用了多少 shell 命令）。**工程**：**semantics-preserving sharded-parallel execution engine**——把 shell 检索加速**最多 7.6×**且 byte-exact 等价于顺序执行。

**关键结果**：7 个开放域 QA 基准 token-level F1 与 EM 最强；指出纯 lexical 交互在 surface-form 变化大的 query 上的局限。GitHub 42★。

**意义**：GrepSeek 把 DCI（Direct Corpus Interaction）从论点变成可训练、可部署的系统，预示 retrieval 范式从"embed + ANN"转向"agent + shell"。详见 `deep-research/10`，本专题作为 deep_research RL 交叉引用——其 GRPO + shell 检索是"agent 用工具直接交互 + RL 训练"的代表。

#### 📄 #20 Role-Agent: Bootstrapping LLM Agents via Dual-Role Evolution [AMAP, 73↑] — arXiv 2606.10917

**问题动机**：LLM agent 的学习常受限于**低效交互反馈 + 静态训练环境**，阻碍泛化。能否让一个 LLM 同时当 agent 和环境、自举共进化？

**核心方法**：Role-Agent 用**单一 LLM 同时充当 agent 和 environment**，实现自举共进化。两个协同组件：① **World-In-Agent (WIA)**——LLM 当 agent，每个动作后**预测未来状态**，预测与实际状态的对齐度作为 **process reward**，鼓励"环境感知推理"（给出密集的逐步反馈而非稀疏的任务末信号）；② **Agent-In-World (AIW)**——LLM **分析失败轨迹的失败模式、检索相似失败模式的任务**，重塑训练数据分布做针对性练习（解决静态训练环境问题）。

**关键结果**：多个基准上一致提升，**平均比强基线 +4%**。

**意义**：Role-Agent 用"单 LLM 双角色"（WIA 给 process reward + AIW 重塑数据分布）解决"低效反馈 + 静态环境"两个限制。它与 Agent0 [[05-agent0]]（出题者-解题者）、Role 的"预测未来状态作 process reward"与 DreamGym [[01-dreamgym]]（经验模型预测状态转移）思路相通，是"agent 自举共进化"的代表。本专题作为 agent_train 简述。GitHub github.com/AMAP-ML/roleagent。

#### 📄 #28 Masking Stale Observations Helps Search Agents — Until It Doesn't [McAuley Lab, 62↑] — arXiv 2606.00408

**问题动机**：长程 search agent 跨多次 tool call 积累大量检索内容，context 预算效率重要。最小干预是 mask 掉过期观察，但何时有用、为何有用不清楚。

**核心方法**：**系统实验**——扫 4B–284B backbone × 3 retriever × 离线/实时 web 基准。**核心发现**：masking 的准确率增益（相对"无 context 管理"）呈**非对称倒 U 形 regime**——弱 retriever 下平台、强 retriever 配中等模型时峰值、模型饱和时骤降。**机制**：masking 是 **token-for-turn 权衡**（移除模型已不 attend 的观测换更多 turn），增益取决于 retriever recall 与模型 implicit filtering capacity 的交互。把 context 管理重定义为 **regime-dependent**。

**关键结果**：把 context 管理重新定义为 regime-dependent 干预——没有银弹。GitHub 16★。

**意义**：这篇是对所有"激进 context 压缩"工作的重要反思——证明 context 管理策略必须匹配 (retriever recall × model filtering capacity) 的具体 regime。它与 RL 的关联在于：长程 search agent RL 的上下文管理（如 Harness-1 [[13-harness-1]] 的 budget-aware rendering）需要这种 regime 意识。本专题作为 deep_research（上下文管理）简述，亦是 `memory/02-masking`、`deep-research/09` 核心。

#### 📄 #40 DRIFT: Span-Level Error Localization in Agent Trajectories [NJU, 54↑] — arXiv 2606.02060

**问题动机**：基于最终答案的评测只能看"成没成功"，看不出"轨迹哪部分让答案不可靠"——DR agent 的长轨迹到底是哪一步开始错的？这对 RL 奖励设计有直接启示（密集的过程级信号比稀疏的最终奖励更有用）。

**核心方法**：收集 2 个 agent 框架 × 3 backbone × 3 基准的 **2,790 条真实轨迹**，转成 semantic span，LLM-assisted 专家标注 harmful error span，建 **TELBench（1000 实例）**。提出 **DRIFT**——**claim-centric 审计框架**：追踪 agent 的 claim、核查其在轨迹证据中的支撑、标记影响答案路径的 unsupported/conflicting span。

**关键结果**：DRIFT 把 span-level error localization 与 first-error accuracy 提升**最多 30pp**，提供 DR agent 可靠性的 process-level 视角。GitHub 21★。

**意义**：DRIFT 把 DR 评测从"最终答案对错"推进到"span 级错误定位"，回答"哪一步开始错"。它对 agentic RL 的价值在于——**过程级错误定位可转化为更密的 RL 奖励信号**（呼应 OpenClaw-RL [[03-openclaw-rl]]/CUDA Agent [[07-cuda-agent]] 的 process reward）。本专题作为 deep_research（评测/过程奖励）简述，亦是 `deep-research/13` 核心。

#### 📄 #48 SearchSwarm: Delegation Intelligence in Agentic LLMs [49↑] — arXiv 2606.09730

**问题动机**：长程 deep research 的上下文需求可无界增长，但模型上下文窗口有限。如何让 agent 通过"委派"分摊上下文压力？

**核心方法**：核心能力是 **delegation intelligence（委派智能）**——main agent **分解任务、决定何时委派什么给 subagent、整合 subagent 返回的摘要结果**，以节省主上下文预算。设计 **harness 引导模型做高质量任务分解与委派**，并约束 subagent 正确返回结果；harness 引导的轨迹**天然编码正确委派决策**，用作 SFT 数据把 delegation intelligence **内化进权重**。

**关键结果**：SearchSwarm-30B-A3B 在 **BrowseComp 68.1 / BrowseComp-ZH 73.3**，同级最佳。GitHub 63★。

**意义**：SearchSwarm 把"何时委派什么给 subagent"作为可训练能力（delegation intelligence），是 DR scaling 第三轴"委派"的代表。它与 WideSeek-R1 [[10-wideseek-r1]]（宽度并行）是 multi-agent 组织的两种形态（委派 vs 并行），与 MiroThinker（交互深度）共同构成 DR scaling 三个新轴。本专题作为 multi_agent 简述，亦是 `deep-research/12` 核心。GitHub github.com/Search-Swarm/SearchSwarm。

## 三、研究现状与趋势（2025 H2 → 2026 H1）

### 1️⃣ Rollout 成本是 Agentic RL 的第一性约束，"经验合成"成为主流解法

真实环境 rollout 慢、贵、奖励噪声大、基建复杂，是 agent RL 落地的最大障碍。2025 H2 起出现一批"**绕过真实 rollout**"的工作：

- **DreamGym**（[[01-dreamgym]]）把环境动态蒸馏成 reasoning-based experience model，纯合成交互媲美 GRPO/PPO；
- **EvoCUA**（[[06-evocua]]）/ **Agent-World** 走另一条路——**自动合成可验证环境 + 数万异步 sandbox 并行**把真实 rollout 规模化；
- 共同点：**environment/experience-as-data**，把"造数据"和"训 policy"合成一个自维持循环。

### 2️⃣ 从"标量奖励"到"信号丰富化"——next-state / 反思 / rubric

标量奖励对长程多轮 agent 太稀疏。三条丰富化路线：

- **next-state signal**（**OpenClaw-RL** [[03-openclaw-rl]]）——用户回复 / 工具输出 / 状态变化里既有 evaluative 又有 directive 信号，后者经 hindsight OPD 抽成 token 级方向监督；
- **反思 consolidation**（**ERL** [[02-experiential-rl]]）——把"反思→改进→内化"显式嵌入 RL loop；
- **演化 rubric**（**DR Tulu** [[12-dr-tulu]]）——长答案不可验证时，让 rubric 与 policy 共进化提供判别性反馈。

### 3️⃣ 多轮信用分配 = 2026 H1 RLVR 算法进步的共同语言

GRPO 在 single-turn 推理有效，但 multi-turn agent 轨迹里"部分 turn 对、部分错"会让朴素信用分配信号互相抵消：

- **GrandCode 的 Agentic GRPO**（🔗）——两阶段更新（immediate + delayed correction）解决 multi-stage rollout off-policy 漂移；
- **SDAR**（🔗）——sigmoid 门控非对称信任；
- **HACRL/HACPO**（[[08-hacrl]]）——异构 agent 间无偏 advantage 共享；
- 都在用 **token/turn 级统计指标**定位并修正信用分配偏差。

### 4️⃣ Multi-Agent RL 的三种新解法：异构互学 / 测试时 / 宽度扩展

MARL 训练贵且不稳定（非平稳 + 高方差），2026 H1 出现三种绕法：

- **异构互学**（**HACRL** [[08-hacrl]]）——训练时共享 rollout、推理时独立，避免协同部署；
- **测试时**（**MATTRL** [[09-mattrl]]）——不更新权重，只在推理时注入结构化文本经验做多 agent 审议；
- **宽度扩展**（**WideSeek-R1** [[10-wideseek-r1]]）——lead-subagent MARL 把"广度信息检索"并行化，4B 媲美 671B 单 agent。

### 5️⃣ Skill 从"inference-time 检索"走向"RL 内化"

Skill 作为 frozen-model 时代的能力载体，2026 H1 的关键转向是**把 skill 训进权重**：

- **SkillRL**（[[14-skillrl]]）——experience distillation 建 SkillBank + 递归进化与 policy 共进；
- **SKILL0**（[[15-skill0]]）——in-context RL **内化** skill，训练时逐步撤除 skill 上下文直到 zero-shot，消除检索噪声与 token 开销；
- 与 `huggingface/` 库的 Skill1 / SkillClaw / SkillOpt 一道，构成"skill × RL"完整谱系。

### 6️⃣ "环境/Harness 设计"与"权重训练"同等重要

- **LLM-in-Sandbox**（[[04-llm-in-sandbox]]）——极简 code sandbox + 非 agentic 数据 RL 就能激发通用 agent，质疑"复杂 harness"假设；
- **Harness-1**（[[13-harness-1]]）——反过来**把状态管理外置到 harness**，让 RL 只优化语义决策；
- **IterResearch**（[[11-iterresearch]]）——周期性重建 workspace 解决 mono-context 膨胀；
- 三者共识：**RL 该优化什么、不该优化什么，取决于环境/harness 怎么切分职责**。

### 7️⃣ Agentic RL 已在"最专家化领域"超越 frontier 闭源

- **CUDA Agent**（[[07-cuda-agent]]）在 GPU kernel 生成上 KernelBench L3 比 Claude Opus 4.5 / Gemini 3 Pro 高 ~40%；
- **GrandCode**（🔗）在 Codeforces 直播击败所有人类；
- 证明 agentic RL 不只是"追平"，在**有可靠自动验证器**的领域已能**反超**最强闭源模型。

### 8️⃣ 自演化 / 零数据 是 agent 摆脱"人类数据天花板"的方向

- **Agent0**（[[05-agent0]]）零外部数据、双 agent 共进化；
- **MetaClaw** / **Role-Agent** 部署中持续演化；
- 与 `huggingface/` 的 SkillClaw（生态级）、`evolve/` 专题一道，指向"agent 自己造课程、自己出题、自己进化"的范式。

---

## 四、收录清单速查（39 篇）

| #  | 论文                                                                                                              | 月/票           | 类 | 精读    |
| -- | ----------------------------------------------------------------------------------------------------------------- | --------------- | -- | ------- |
| 01 | DreamGym                                                                                                          | 2025-11 / 83↑  | A  | ⭐      |
| 02 | Experiential RL (ERL)                                                                                             | 2026-02 / 75↑  | A  | ⭐      |
| 03 | OpenClaw-RL                                                                                                       | 2026-03 / 156↑ | A  | ⭐      |
| 04 | LLM-in-Sandbox                                                                                                    | 2026-01 / 87↑  | A  | ⭐      |
| 05 | Agent0                                                                                                            | 2025-11 / 110↑ | B  | ⭐      |
| 06 | EvoCUA                                                                                                            | 2026-01 / 92↑  | B  | ⭐      |
| 07 | CUDA Agent                                                                                                        | 2026-03 / 99↑  | B  | ⭐      |
| 08 | HACRL                                                                                                             | 2026-03 / 198↑ | C  | ⭐      |
| 09 | MATTRL                                                                                                            | 2026-01 / 92↑  | C  | ⭐      |
| 10 | WideSeek-R1                                                                                                       | 2026-02 / 100↑ | C  | ⭐      |
| 11 | IterResearch                                                                                                      | 2025-11 / 80↑  | D  | ⭐      |
| 12 | DR Tulu                                                                                                           | 2025-11 / 63↑  | D  | ⭐      |
| 13 | Harness-1                                                                                                         | 2026-06 / 52↑  | D  | ⭐      |
| 14 | SkillRL                                                                                                           | 2026-02 / 76↑  | E  | ⭐      |
| 15 | SKILL0                                                                                                            | 2026-04 / 101↑ | E  | ⭐      |
| 📄 | MEDS · DVAO                                                                                                      | 04/05           | A  | 简述    |
| 📄 | MetaClaw · Agent-World · GroundCUA · GeoVista · Role-Agent · OpenGame · Thinking-with-Map · SWE-rebench-V2 | —              | B  | 简述    |
| 📄 | SearchSwarm                                                                                                       | 2026-06         | C  | 简述    |
| 📄 | OpenSearch-VL · GrepSeek · Observation-Masking · DRIFT · MiroThinker                                          | —              | D  | 简述/🔗 |
| 📄 | Skill1                                                                                                            | 2026-05         | E  | 🔗      |
| 📄 | GLM-5 · Kimi K2.5 · Youtu-Agent · LongCat · GrandCode · SDAR · Let-It-Flow                                  | —              | F  | 简述/🔗 |
| 📄 | AgentDoG 1.5                                                                                                      | 2026-05         | G  | 简述    |

> 🔗 交叉引用 `huggingface/` 周榜库已有精读：GrandCode→01｜SDAR→08｜GLM-5→06｜Kimi K2.5→13｜Youtu-Agent→05｜MiroThinker→12｜OpenSearch-VL/GrepSeek/Skill1/Let-It-Flow→见 `huggingface/00-summary`。
