# 2025.11–2026.06 HuggingFace 月榜 — Agentic RL 专题汇总

> **数据范围**：HuggingFace Papers **月榜 top 50**，2025-11 至 2026-06（共 8 个月）
> **筛选**：标题/摘要/关键词**同时**命中 **Agent 信号**（agent / agentic / tool use / multi-turn / trajectory / long-horizon / search agent …）**与 RL 信号**（reinforcement learning / RLVR / GRPO / policy optimization / verifiable reward / agentic RL …）
> **命中规模**：8 个月 top50 去重后 400 篇 → **agent+RL 双命中 126 篇** → 人工剔除 world-model / VLA 机器人 / 视频图像生成 / 纯 RLVR 推理（非 agentic）/ 纯 memory-benchmark 后，**收录 39 篇**（15 篇精读 ⭐ + 24 篇简述 📄）
> **标记**：⭐ = 本专题精读｜📄 = 基于摘要的简述｜🔗 = 已在 `huggingface/` 周榜库精读，此处交叉引用

---

## 一、分类总览

> Agentic RL = 「**把 RL 的信用分配/奖励/探索机制，适配到 agent 的多轮、长程、工具交互轨迹上**」。2025 H2–2026 H1 这条线已从「能不能用 RL 训 agent」走到「**怎么把 rollout 成本、稀疏延迟奖励、多轮信用分配、技能内化做精细**」。

| 主题 | 精读 | 简述 | 一句话定位 |
| --- | --- | --- | --- |
| **A. RL 算法 / 训练范式** | 01 DreamGym · 02 ERL · 03 OpenClaw-RL · 04 LLM-in-Sandbox | MEDS · DVAO · GrandCode🔗 · SDAR🔗 | 把"经验合成 / 反思 / next-state 信号 / 极简环境"做进 RL loop |
| **B. Agent 自演化 & 经验** | 05 Agent0 · 06 EvoCUA · 07 CUDA Agent | MetaClaw · Agent-World · GroundCUA · GeoVista · Role-Agent · OpenGame · Thinking-with-Map · SWE-rebench-V2 | 自动合成环境/任务 + 大规模 rollout + 失败回收的自维持循环 |
| **C. Multi-Agent RL** | 08 HACRL · 09 MATTRL · 10 WideSeek-R1 | SearchSwarm | 异构互学 / 测试时协作 / 宽度扩展——MARL 的三种新解法 |
| **D. Deep Research / Search Agent RL** | 11 IterResearch · 12 DR Tulu · 13 Harness-1 | OpenSearch-VL🔗 · GrepSeek🔗 · Observation-Masking · DRIFT · MiroThinker🔗 | 长程上下文管理 + 可验证奖励 + 状态外置，是 DR 训练的三条主线 |
| **E. Skill RL** | 14 SkillRL · 15 SKILL0 | Skill1🔗 | 把 skill 的发现/检索/内化统一进 RL，取代 inference-time 检索 |
| **F. 旗舰模型（Agent RL 训练）** | — | GLM-5🔗 · Kimi K2.5🔗 · Youtu-Agent🔗 · LongCat-Flash-Thinking | 工业旗舰里 agentic RL 的工程化落地（异步 RL / Agent Swarm / Training-free GRPO） |
| **G. Agent 安全 + RL** | — | AgentDoG 1.5 | 安全对齐里用到 RL training |

---

## 二、按月列出（摘要均基于 arXiv 原文）

### 2025-11（14 命中 / 收录 9）

- ⭐ **#17 Agent0: Self-Evolving Agents from Zero Data via Tool-Integrated Reasoning** [UNC, 110↑] — arXiv 2511.16043
  - 从**同一 base LLM** 初始化两个 agent，构成 **symbiotic competition**：**curriculum agent**（出题，不断提更难的 frontier 任务）vs **executor agent**（解题，集成外部工具）。executor 变强 → 反过来逼 curriculum 出更复杂、更"工具感知"的任务，形成自强化循环。**零外部数据**、多步共进化。Qwen3-8B-Base 上数学 +18%、通用推理 +24%。精读见 [[05-agent0]]。
- ⭐ **#33 DreamGym: Scaling Agent Learning via Experience Synthesis** [Meta, 83↑] — arXiv 2511.03773
  - **首个用合成经验做 online agent RL** 的统一框架。核心：把环境动态**蒸馏成一个 reasoning-based experience model**，用 step-by-step 推理产生一致的状态转移 + 反馈信号，**替代昂贵的真实 rollout**。配 experience replay buffer（离线真实数据初始化 + 持续补充）+ 自适应课程（生成挑战当前 policy 的新任务）。WebArena 非 RL-ready 设定超所有基线 30%+；RL-ready 设定下纯合成交互即可媲美 GRPO/PPO。精读见 [[01-dreamgym]]。
- ⭐ **#36 IterResearch: Long-Horizon Agents via Markovian State Reconstruction** [Tongyi Lab, 80↑] — arXiv 2511.07327
  - 诊断 deep research agent 的 **mono-contextual 范式**——把所有信息堆进单一膨胀上下文，导致"context 窒息 + 噪声污染"。**IterResearch** 用 MDP 思路**周期性重建 workspace**，维护一份**演化的 report 作为 memory**，定期合成洞察而非无脑追加。训练用 **EAPO（Efficiency-Aware Policy Optimization）**——几何奖励折扣激励高效探索 + 自适应下采样稳定分布式训练。6 个基准平均 +14.5pp，交互可扩展到 2048 步（3.5%→42.5%）。精读见 [[11-iterresearch]]。
- ⭐ **#47 DR Tulu: RL with Evolving Rubrics for Deep Research** [AI2 / UW, 63↑] — arXiv 2511.19399
  - 指出开源 DR 模型多在**易验证的短答案 QA** 上用 RLVR 训练，无法迁移到真实长答案任务。提出 **RLER**——**rubric 与 policy 共进化**：训练中 rubric 不断吸收新检索到的信息 + 对比模型回答，提供更有判别力的 on-policy 反馈。DR Tulu-8B 在科学/医疗/通用 4 个长答案基准超 Tongyi DR 平均 +15.6%，与 OpenAI DR 持平但**每 query 便宜 1000×**。精读见 [[12-dr-tulu]]。
- 📄 **#4 MiroThinker v1.0: Interactive Scaling for Research Agents** [MiroMind, 196↑] — arXiv 2511.11793 🔗
  - tool-augmented research agent，主打**交互维度扩展**（不只是堆模型/上下文）。RL 后训练提升单步工具使用可靠性。H1 版（验证嵌入推理）已在 `huggingface/12-mirothinker.md` 精读。
- 📄 **#20 GroundCUA: Grounding Computer Use Agents on Human Demonstrations** [ServiceNow, 107↑] — arXiv 2511.07332
  - 桌面环境的高质量 grounding 数据集（人类演示）+ GroundNext 模型，RL 用于把自然语言指令精确连到屏幕元素。偏数据基建。
- 📄 **#24 GeoVista: Web-Augmented Agentic Visual Reasoning for Geolocalization** [Tencent-Hunyuan, 98↑] — arXiv 2511.15705
  - agentic 视觉推理 + web 工具，地理定位任务用 RL 训多步工具使用，超越只做图像操作工具的方法。
- 📄 *（剔除）* General Agentic Memory / HaluMem / P1 / V-Thinker / PAN / MMaDA-Parallel / π_RL — memory-benchmark / 纯推理 RL / world-model / VLA，RL 或非 agentic 或非主线。

### 2025-12（17 命中 / 收录 2）

- 📄 **#1 From Code Foundation Models to Agents: A Practical Guide to Code Intelligence** [Beihang, 305↑] — arXiv 2511.18538
  - code intelligence 大综述，含 coding agent 的 RL 训练章节。偏 survey，列为 reference。
- 📄 **#13 ToolOrchestra: Elevating Intelligence via Efficient Model and Tool Orchestration** [NVIDIA, 128↑] — arXiv 2511.21689
  - 小 orchestrator 调度大模型 + 工具，用 RL 训练编排策略以低成本攻 HLE。偏 orchestration。
- 📄 *（剔除）* DeepSeek-V3.2 / Qwen3-VL / Memory in the Age of AI Agents / Step-GUI / Native Parallel Reasoner / Deep Research Survey 等——旗舰报告 / 综述 / OPD / world-model，agentic RL 非核心贡献（DeepSeek-V3.2、Deep Research Survey 可在其他库查）。

### 2026-01（16 命中 / 收录 7）

- ⭐ **#35 EvoCUA: Evolving Computer Use Agents via Scalable Synthetic Experience** [Meituan, 92↑] — arXiv 2601.15876
  - 把数据生成与 policy 优化合并为**自维持进化循环**。三件套：① **Verifiable Synthesis Engine**——自动生成任务 + 可执行 validator；② **数万异步 sandbox rollout** 并行采经验；③ **Iterative Evolving Learning**——动态识别能力边界调节更新 + **失败轨迹回收**（错误分析 + 自我纠正转为监督）。OSWorld **56.7%** 开源 SOTA，超 UI-TARS-2 (53.1%) / OpenCUA-72B (45.0%)。精读见 [[06-evocua]]。
- ⭐ **#36 MATTRL: Collaborative Multi-Agent Test-Time RL for Reasoning** [NUS / MIT, 92↑] — arXiv 2601.09667
  - 针对 MARL 训练"资源重 + 不稳定（co-adapting 非平稳 + 奖励稀疏高方差）"，提出**测试时**多 agent RL——**不更新权重**，而是把结构化文本经验注入多 agent 审议。流程：组建多专家团队 → 多轮讨论 → 检索整合测试时经验 → 达成共识。研究 **turn-level 经验池的 credit assignment** 再注入对话。医/数/教基准比多 agent 基线 +3.67%、比单 agent +8.67%。精读见 [[09-mattrl]]。
- ⭐ **#38 LLM-in-Sandbox: Computer Environments Elicit General Agentic Intelligence** [Microsoft Research, 87↑] — arXiv 2601.16206
  - 把计算机虚拟化为**仅含基础功能的 code sandbox**，激发三大元能力：① 外部资源访问；② 文件管理；③ 代码执行。**LLM-in-Sandbox-RL** 关键——**只用非 agentic 数据**（普通问答/代码）在 sandbox 内训练，就能让弱模型学会驾驭环境。强模型免训练即 +15.5% 任务通过率，**token 消耗 -8×**。精读见 [[04-llm-in-sandbox]]。
- 📄 **#9 LongCat-Flash-Thinking-2601 Technical Report** [Meituan-LongCat, 181↑] — arXiv 2601.16725
  - 560B MoE 推理旗舰，**agentic reasoning** 为核心卖点，RL 训练 multi-turn tool use。开源 SOTA 级 agentic 推理。
- 📄 **#11 Thinking with Map: Reinforced Parallel Map-Augmented Agent** [AMAP, 170↑] — arXiv 2601.05432
  - 让 VLM 像人一样**显式调用地图工具**做地理定位，agentic RL 训 multi-step 地图使用 + parallel TTS 投票。Acc@500m 8%→22.1%。（`huggingface/00` W03 已提及）
- 📄 **#22 Youtu-Agent: Automated Generation + Hybrid Policy Optimization** [Tencent, 119↑] — arXiv 2512.24615 🔗
  - **Training-free GRPO**（LLM evaluator 蒸馏"语义 advantage"做 textual LoRA，$18 击败 $10K RL）。已在 `huggingface/05-youtu-agent.md` 精读。
- 📄 **#27 Let It Flow / ROME: Open Agentic Learning Ecosystem** [AGI Lab, 109↑] — arXiv 2512.24873
  - 端到端开源 agent infra（ROLL 后训练 + ROCK 沙盒 + iFlow CLI）+ **IPA** 把信用分配提到"语义交互块"粒度。（`huggingface/00` W01 已提及）

### 2026-02（20 命中 / 收录 6）

- ⭐ **#42 SkillRL: Evolving Agents via Recursive Skill-Augmented RL** [UNC, 76↑] — arXiv 2602.08234
  - 诊断 memory-based 方法"存 raw trajectory 冗余且噪声大、提不出可复用模式"。**SkillRL** 用 **experience-based distillation** 建**层级 SkillBank** + 自适应检索（通用 + 任务特定启发式）+ **递归进化**让 skill 库**与 policy 在 RL 中共进化**。ALFWorld/WebShop + 7 个搜索任务超强基线 +15.3%。精读见 [[14-skillrl]]。
- ⭐ **#43 Experiential Reinforcement Learning (ERL)** [USC / Microsoft, 75↑] — arXiv 2602.13949
  - 把 **experience-reflection-consolidation 循环**显式嵌入 RL，应对稀疏延迟奖励。四步：初次尝试 → 环境反馈 → **反思**（指导改进）→ 精修第二次尝试，**成功的第二次被强化并内化进 base policy**。把反馈转为"结构化行为修正"，**部署时零额外推理开销**。复杂多步环境 +81%、工具推理 +11%。精读见 [[02-experiential-rl]]。
- 📄 **#5 Kimi K2.5: Visual Agentic Intelligence** [Moonshot, 273↑] — arXiv 2602.02276 🔗
  - joint text-vision RL + **Agent Swarm**（自主并行编排，延迟 -4.5×）。已在 `huggingface/13-kimi-k25.md` 精读。
- 📄 **#22 GLM-5: from Vibe Coding to Agentic Engineering** [Zhipu+清华, 151↑] — arXiv 2602.15763 🔗
  - **异步 Agent RL**（slime 框架 + IcePop 不对称 clip），744B MoE，首个 AAII v4.0 破 50 开源旗舰。已在 `huggingface/06-glm-5.md` 精读。
- 📄 **#30 WideSeek-R1: Width Scaling via Multi-Agent RL** [清华 / RLinf, 100↑] — arXiv 2602.04634 ⭐
  - 见下方精读引用（移至 C 类 #10）。lead-subagent MARL，宽度扩展信息检索。精读见 [[10-wideseek-r1]]。
- 📄 **#32 Composition-RL: Compose Your Verifiable Prompts for RLVR** [Tencent-Hunyuan, 94↑] — arXiv 2602.12036
  - RLVR 训练数据侧——组合可验证 prompt 提升数据效率。偏 RLVR-data，列 reference。
- 📄 *（剔除）* Green-VLA / Code2World / Step 3.5 Flash / UI-Venus-1.5 / Vision-DeepResearch / F-GRPO / MedXIAOHE 等——VLA / GUI-world-model / 旗舰报告 / 纯 RLVR / 多模态模型，主线非 agentic RL（部分在其他库）。

### 2026-03（12 命中 / 收录 5）

- ⭐ **#6 Heterogeneous Agent Collaborative Reinforcement Learning (HACRL)** [Beihang 等, 198↑] — arXiv 2603.02604
  - 新的 **RLVR 范式**：异构 agent **训练时共享已验证 rollout 互相提升、推理时各自独立**。区别于 LLM-MARL（无需协同部署）与单向蒸馏（实现**双向互学**）。**HACPO** 算法做有原则的 rollout 共享，4 个机制处理能力差异 + 策略分布漂移，**理论保证无偏 advantage 估计**。比 GSPO（双倍 rollout）**+3.6%、计算仅一半**。精读见 [[08-hacrl]]。
- ⭐ **#12 OpenClaw-RL: Train Any Agent Simply by Talking** [Princeton, 156↑] — arXiv 2603.10165
  - 核心观察：每次 agent 交互都产生 **next-state signal**（用户回复 / 工具输出 / 终端 GUI 状态变化），且**对所有信号同时学**。两类信息：**evaluative**（PRM judge 抽成标量奖励）+ **directive**（**Hindsight-Guided OPD** 抽成 token 级方向性 advantage——比标量奖励更丰富）。**异步设计**：服务请求 / PRM 评判 / trainer 更新同时进行零协调开销。统一 terminal/GUI/SWE/tool-call。精读见 [[03-openclaw-rl]]。
- ⭐ **#37 CUDA Agent: Large-Scale Agentic RL for CUDA Kernel Generation** [ByteDance Seed, 99↑] — arXiv 2602.24286
  - 大规模 agentic RL 自动写高性能 CUDA。三件套：① 可扩展数据合成 pipeline（含 ground-truth 校验）；② **技能增强 CUDA 开发环境**（自动验证 + profiling 给可靠奖励）；③ 稳定 RL 算法技术（应对性能跨数量级的高方差奖励）。KernelBench 超 `torch.compile`：**L1 100% / L2 100% / L3 92%**，L3 比 Claude Opus 4.5 / Gemini 3 Pro 高 ~40%。精读见 [[07-cuda-agent]]。
- 📄 **#24 MetaClaw: An Agent That Meta-Learns and Evolves in the Wild** [UNC, 140↑] — arXiv 2603.17187
  - 部署中持续 meta-learn + 演化，用 process reward。解决"部署 agent 静态、不随用户需求演化"。
- 📄 **#48 SWE-rebench V2: Language-Agnostic SWE Task Collection at Scale** [Nebius, 89↑] — arXiv 2602.23866
  - 为 SWE agent RL 提供大规模、可复现执行环境、可靠奖励的任务集——RL 训练的瓶颈是任务稀缺。
- 📄 *（剔除）* MiroThinker-1.7&H1 (🔗 已精读) / Intern-S1-Pro / 各 world-model — 旗舰 / world-model。

### 2026-04（16 命中 / 收录 5）

- ⭐ **#39 SKILL0: In-Context Agentic RL for Skill Internalization** [浙大, 101↑] — arXiv 2604.02268
  - 质疑 inference-time skill 检索（检索噪声 + token 开销 + 模型只"照做"未真正习得）。**SKILL0** 用 **in-context RL 把 skill 内化进参数**，零运行时检索。训练课程：从**完整 skill 上下文逐步撤除**（线性衰减预算）直到全 zero-shot；**动态课程**按 on-policy helpfulness 只保留当前 policy 仍受益的 skill 文件。ALFWorld +9.7% / Search-QA +6.6% / WebShop +10.1%，每步 <0.5k token。精读见 [[15-skill0]]。
- 📄 **#1 GrandCode: Grandmaster Competitive Programming via Agentic RL** [Deep Reinforce, 632↑] — arXiv 2604.02721 🔗
  - **Agentic GRPO**（两阶段更新：immediate + delayed correction）——本专题 RL 算法的旗舰参照。首个在 Codeforces 直播击败所有人类。已在 `huggingface/01-grandcode.md` 精读。
- 📄 **#23 MEDS: Memory-Enhanced Dynamic Reward Shaping** [144↑] — arXiv 2604.11297
  - 把**历史 rollout 的错误模式**用密度聚类识别，对落入高频错误簇的新 rollout 加重惩罚，提升采样多样性。pass@1 +4.13 / pass@128 +4.37。（`huggingface/00` W16 已提及）
- 📄 **#46 Agent-World: Scaling Real-World Environment Synthesis** [ByteDance Seed, 85↑] — arXiv 2604.18292
  - 自演化环境合成基建——agent 自主发现主题数据 + 可执行工具生态合成可验证任务 + 多环境 RL + 自演化竞技场识别能力缺口。
- 📄 **#49 OpenGame: Open Agentic Coding for Games** [81↑] — arXiv 2604.18394
  - 端到端网页游戏生成 agent，GameCoder-27B 三阶段训练 CPT→SFT→**执行驱动 RL**（reward = 能否 build + 能否玩 + 视觉合理）。
- 📄 *（剔除）* Recursive Multi-Agent Systems (🔗) / ClawBench (🔗) / Agentic World Modeling (🔗) / Self-Distilled RLVR / AgentSPEX / GLM-5V-Turbo 等——已在其他库 / world-model / 纯 RLVR / DSL。

### 2026-05（18 命中 / 收录 4）

- 📄 **#26 DVAO: Dynamic Variance-adaptive Advantage Optimization for Multi-reward RL** [134↑] — arXiv 2605.25604
  - 多奖励 RL，针对 GRPO 在多奖励下的方差问题做动态自适应 advantage 优化，含 tool-use rollout。
- 📄 **#36 Self-Distilled Agentic RL (SDAR)** [浙大+美团+清华, 112↑] — arXiv 2605.15155 🔗
  - **OPSD 在 multi-turn agent 训练的不稳定**诊断 + 修复：sigmoid 门控**非对称信任**（positive gap 强化、negative gap 软抑制）。已在 `huggingface/08-sdar.md` 精读。
- 📄 **#37 Skill1: Unified Evolution of Skill-Augmented Agents via RL** [112↑] — arXiv 2605.06130 🔗
  - 单 policy 统一 skill selection/utilization/distillation。（`huggingface/00` W19 已提及）
- 📄 **#44 OpenSearch-VL: Open Recipe for Multimodal Search Agents** [Tencent-Hunyuan, 102↑] — arXiv 2605.05185 🔗
  - **多轮 fatal-aware GRPO**（失败后 token 掩码 + 单侧 advantage 钳制）。（`huggingface/00` W19 已提及）
- 📄 **#22 AgentDoG 1.5: Alignment Framework for AI Agent Safety** [上海AI Lab, 142↑] — arXiv 2605.29801
  - 轻量可扩展 agent 安全对齐框架，含 RL training，应对 open-world agent 的安全风险。
- 📄 *（剔除）* Gamma-World / Code as Agent Harness (🔗) / Qwen-VLA / ARIS (🔗) / 各 OPD / 视频生成——world-model / VLA / 已在其他库 / 视频生成。

### 2026-06（13 命中 / 收录 6）

- ⭐ **#45 Harness-1: RL for Search Agents with State-Externalizing Harnesses** [UIUC / Chroma, 52↑] — arXiv 2606.02373
  - 论点：把搜索 agent 训成"transcript 上的 policy"会逼模型同时管语义决策 + 琐碎状态记账。**Harness-1**（20B 检索子 agent）在**有状态 harness** 内 RL——harness 维护**环境侧 working memory**（candidate pool / 重要性标注的 curated set / 证据链接 / 验证记录 / 去重压缩观察 / 预算感知渲染），policy 只保留语义决策（搜什么 / 留弃哪些 / 验证什么 / 何时停）。8 个检索基准 curated recall 0.730，比次强开源 +11.4。精读见 [[13-harness-1]]。
- 📄 **#12 GrepSeek: Training Search Agents for Direct Corpus Interaction** [UMass, 106↑] — arXiv 2605.29307 🔗
  - shell（grep/find/sed/awk）直接与语料交互 + GRPO，挑战 embedding 检索范式。（`huggingface/00` W22 已提及）
- 📄 **#20 Role-Agent: Bootstrapping LLM Agents via Dual-Role Evolution** [AMAP, 73↑] — arXiv 2606.10917
  - 双角色演化 + process reward 自举，解决交互反馈低效 + 静态训练环境。
- 📄 **#28 Masking Stale Observations Helps Search Agents — Until It Doesn't** [McAuley Lab, 62↑] — arXiv 2606.00408
  - 长程搜索 agent 上下文裁剪的 **regime map**——掩码陈旧观察何时有益何时有害的机制分析。
- 📄 **#40 DRIFT: Span-Level Error Localization in Agent Trajectories** [NJU, 54↑] — arXiv 2606.02060
  - deep research agent 轨迹的 **span 级错误定位**——不只看最终答案对错，而是定位哪段轨迹让答案不可靠（对 RL 奖励设计有启示）。
- 📄 **#48 SearchSwarm: Delegation Intelligence in Agentic LLMs** [49↑] — arXiv 2606.09730
  - 主 agent 委派子 agent 做长程 deep research，应对无界上下文 vs 有限窗口。
- 📄 *（剔除）* Kwai Keye-VL / Cosmos 3 / Audio Interaction / InterleaveThinker / On-Policy Distillation 几何 / Mellum2 — 多模态模型 / world-model / 音频 / 图像生成 / OPD / 代码模型报告。

---

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

| # | 论文 | 月/票 | 类 | 精读 |
| --- | --- | --- | --- | --- |
| 01 | DreamGym | 2025-11 / 83↑ | A | ⭐ |
| 02 | Experiential RL (ERL) | 2026-02 / 75↑ | A | ⭐ |
| 03 | OpenClaw-RL | 2026-03 / 156↑ | A | ⭐ |
| 04 | LLM-in-Sandbox | 2026-01 / 87↑ | A | ⭐ |
| 05 | Agent0 | 2025-11 / 110↑ | B | ⭐ |
| 06 | EvoCUA | 2026-01 / 92↑ | B | ⭐ |
| 07 | CUDA Agent | 2026-03 / 99↑ | B | ⭐ |
| 08 | HACRL | 2026-03 / 198↑ | C | ⭐ |
| 09 | MATTRL | 2026-01 / 92↑ | C | ⭐ |
| 10 | WideSeek-R1 | 2026-02 / 100↑ | C | ⭐ |
| 11 | IterResearch | 2025-11 / 80↑ | D | ⭐ |
| 12 | DR Tulu | 2025-11 / 63↑ | D | ⭐ |
| 13 | Harness-1 | 2026-06 / 52↑ | D | ⭐ |
| 14 | SkillRL | 2026-02 / 76↑ | E | ⭐ |
| 15 | SKILL0 | 2026-04 / 101↑ | E | ⭐ |
| 📄 | MEDS · DVAO | 04/05 | A | 简述 |
| 📄 | MetaClaw · Agent-World · GroundCUA · GeoVista · Role-Agent · OpenGame · Thinking-with-Map · SWE-rebench-V2 | — | B | 简述 |
| 📄 | SearchSwarm | 2026-06 | C | 简述 |
| 📄 | OpenSearch-VL · GrepSeek · Observation-Masking · DRIFT · MiroThinker | — | D | 简述/🔗 |
| 📄 | Skill1 | 2026-05 | E | 🔗 |
| 📄 | GLM-5 · Kimi K2.5 · Youtu-Agent · LongCat · GrandCode · SDAR · Let-It-Flow | — | F | 简述/🔗 |
| 📄 | AgentDoG 1.5 | 2026-05 | G | 简述 |

> 🔗 交叉引用 `huggingface/` 周榜库已有精读：GrandCode→01｜SDAR→08｜GLM-5→06｜Kimi K2.5→13｜Youtu-Agent→05｜MiroThinker→12｜OpenSearch-VL/GrepSeek/Skill1/Let-It-Flow→见 `huggingface/00-summary`。
