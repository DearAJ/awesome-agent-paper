# HuggingFace Daily Papers (2026 W01–W24) — Agent / Deep Research 论文库

> **生成时间**：2026-06-10
> **数据范围**：HuggingFace Papers 周榜 2026-W01 至 2026-W24（共 24 周）
> **筛选范围**：每周 **top 10** 中标题/摘要明确涉及 **AI Agent / Agentic System / Deep Research / LLM Agent / Tool Use / Agentic RL** 的论文
> **目录定位**：`Q:/papers/`

---

## 📚 20 篇精读笔记（按主题分类，编号同时就是分类顺序）

> **Agentic RL：		01-04**
>
> **数据/skill 合成：	02-03+11**
>
> **Deep Research：	04+07+12**
>
> **旗舰模型：			05+06+13**
>
> **World Model：	14-16**
>
> **Benchmarks：	17-19**
>
> **Harness：			20**

---

### 🔥 Agentic RL & 训练算法（4 篇）

> 2026 H1 的 Agentic RL 已进入精细化阶段——不再是 GRPO vs PPO 的粗对比，而是 token 级、stage 级的细致 credit assignment 与不稳定性修复。

| #            | 论文                                             | 周/票          | 机构                    | 核心贡献                                                                                                                     |
| ------------ | ------------------------------------------------ | -------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **01** | **[GrandCode](01-grandcode.md)**              | W15 / 632↑ 🔥 | Deep Reinforce          | **Agentic GRPO**（两阶段更新：Immediate + Delayed Correction），首个在 Codeforces 直播击败所有人类的 AI（3 场全部 #1） |
| **08** | **[SDAR](08-sdar.md)**                        | W20 / 111↑    | 浙大 + 美团 + 清华      | OPSD multi-turn 不稳定的诊断 + 修复：sigmoid 门控的**非对称信任**（positive gap 强化、negative gap 软抑制）            |
| **09** | **[DelTA](09-delta.md)**                      | W21 / 204↑    | 3 作者（Yankai Lin 等） | **RLVR 的判别器视角**——把 update 写成"正负质心之差"，token 系数重加权稀释方向                                        |
| **10** | **[Weak-Driven Learning](10-weak-driven.md)** | W07 / 290↑    | 12 作者                 | 突破后训练饱和——用模型**自己的弱 checkpoint** + entropy dynamics 找回失去的多样性，零额外推理代价                    |

### 🎯 数据 / skill 合成（3 篇）

> "Data Synthesis Is the New Model Architecture" —— 2026 H1 开源追赶闭源的核心战场。

| #            | 论文                                | 周/票       | 机构       | 核心贡献                                                                                                   |
| ------------ | ----------------------------------- | ----------- | ---------- | ---------------------------------------------------------------------------------------------------------- |
| **02** | **[TermiGen](02-termigen.md)**   | W07 / 210↑ | UCSB MLSec | Generator-Critic 主动**错误注入** + Docker 验证循环；32B 开源在 TerminalBench 拿 31.3%，击败 o4-mini |
| **03** | **[SkillOpt](03-skillopt.md)**   | W22 / 224↑ | MSR + 上交 | 把 SGD 整套机制搬到**text-space** 优化 agent skill——52/52 全胜，跨模型/harness/基准均迁移          |
| **11** | **[SkillClaw](11-skillclaw.md)** | W15 / 293↑ | 8 作者     | 多用户生态级**集体技能进化**——把跨用户 trajectory 作为改进 skill 的主要信号                        |

### 📚 Deep Research / Research Agent（3 篇）

> 从单 agent 长程任务的"goal drift" 问题出发，到把 verification 嵌入推理、到跨 model family 对抗审查。

| #            | 论文                                       | 周/票       | 机构               | 核心贡献                                                                                                                 |
| ------------ | ------------------------------------------ | ----------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| **04** | **[VideoDR](04-videodr.md)**            | W03 / 214↑ | 兰大 / QuantaAlpha | **首个** video deep research benchmark；揭示反直觉发现：**Agentic 不必更好**（弱模型反退化 9 分）            |
| **07** | **[ARIS](07-aris.md)**                  | W19 / 131↑ | SJTU + SII         | **跨 model family 对抗式协作** harness——executor 与 reviewer 强制不同模型族 + 三阶段证据-声明审计                |
| **12** | **[MiroThinker H1](12-mirothinker.md)** | W12 / 187↑ | MiroMind AI        | **Verification 嵌入推理过程**——local（中间决策评估）+ global（证据链一致性审计），正面回应 VideoDR 的 goal drift |

### 🏛 旗舰模型 & Agent 框架（3 篇）

> 国产开源旗舰从"追赶" 走向"差异化定位"——GLM-5 拼规模、Youtu-Agent 拼框架、Kimi K2.5 拼推理时并行。

| #            | 论文                                    | 周/票                     | 机构        | 核心贡献                                                                                               |
| ------------ | --------------------------------------- | ------------------------- | ----------- | ------------------------------------------------------------------------------------------------------ |
| **05** | **[Youtu-Agent](05-youtu-agent.md)** | W02 / 119↑               | 腾讯优图    | **Training-free GRPO**——用 LLM evaluator 蒸馏"语义 advantage"做 textual LoRA，$18 击败 $10K RL |
| **06** | **[GLM-5](06-glm-5.md)**             | W08 /**3.39k↑** 🔥 | 智谱 + 清华 | 744B MoE + DSA 稀疏注意力 + 异步 Agent RL（slime + IcePop），**首个 AAII v4.0 破 50 的开源旗舰** |
| **13** | **[Kimi K2.5](13-kimi-k25.md)**      | W06 / 273↑               | Moonshot    | **Agent Swarm**——自主并行 agent 编排，延迟最高 -4.5x                                           |

### 🌐 Multi-Agent & World Model（3 篇）

> 2026 H1 集中爆发的新主题——从 levels × laws 框架到 multi-agent video generation 到 latent space 通信。

| #            | 论文                                                          | 周/票       | 机构                   | 核心贡献                                                                                                                                                        |
| ------------ | ------------------------------------------------------------- | ----------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **14** | **[Gamma-World](14-gamma-world.md)**                       | W22 / 422↑ | NVIDIA                 | **多 agent 生成式 world model**——SRAE (置换等变 agent encoding) + Sparse Hub Attention，24 FPS 实时，训 2 player 泛化到 4 player                        |
| **15** | **[Agentic World Modeling](15-agentic-world-modeling.md)** | W18 / 227↑ | 50 作者                | World model 主题**奠基性 manifesto**——**Levels (L1 Predictor / L2 Simulator / L3 Evolver) × Laws (Physical/Digital/Social/Scientific)** 二维分类 |
| **16** | **[Recursive Multi-Agent Systems](16-recursive-mas.md)**   | W18 / 276↑ | 12 作者（Stanford 等） | 把**递归 LM 扩展原则**推到多 agent——RecursiveLink + latent communication，+8.3% 准确率、1.2-2.4× 加速、最多 -75.6% token                               |

### 📊 Agent Benchmarks（3 篇）

> Benchmark 从"学术难题"走向"经济价值真实部署"——HLE/GAIA 已饱和，真实环境的 ClawBench / ALE 才是新前线。

| #            | 论文                                                   | 周/票                     | 机构            | 核心贡献                                                                                                  |
| ------------ | ------------------------------------------------------ | ------------------------- | --------------- | --------------------------------------------------------------------------------------------------------- |
| **17** | **[SkillsBench](17-skillsbench.md)**                | W08 /**1.33k↑** 🔥 | BenchFlow       | Agent skill 评测**事实标杆**——11 × 86 任务 × 7,308 轨迹；揭示 "self-generated skill 平均无收益" |
| **18** | **[ClawBench](18-clawbench.md)**                    | W15 / 263↑               | NAIL-Group      | **在 144 个真实生产网站**上测，轻量 interception 拦截最终提交；Claude 4.6 仅 33.3%                  |
| **19** | **[Agents&#39; Last Exam](19-agents-last-exam.md)** | W23 / 155↑               | UC Berkeley RDI | 经济价值真实部署的极限——O\*NET/SOC 2018 职业分类 × 1000+ 任务，主流模型平均完整通过率仅 **2.6%** |

### 🛠 Harness Engineering（1 篇）

> "Harness engineering" 是 2026 H1 才正式被命名的新主题——系统级 logic 的设计与模型权重同等重要。

| #            | 论文                                                  | 周/票       | 机构    | 核心贡献                                                                                                                     |
| ------------ | ----------------------------------------------------- | ----------- | ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **20** | **[Code as Agent Harness](20-code-as-harness.md)** | W21 / 215↑ | 42 作者 | **Harness engineering 奠基综述**——把代码作为 agent 的 operational substrate，按 interface/mechanism/scaling 三层组织 |

---

## 目录结构

```
Q:/papers/
├── README.md                            # 本文件 — 总索引（按分类组织）
├── 00-summary-2026-W01-W24.md           # 24 周 top10 命中论文汇总（按周次）
│
│ ── Agentic RL & 训练算法 ──
├── 01-grandcode.md                      # Agentic GRPO 拿编程 Grandmaster
├── 08-sdar.md                           # Self-Distilled Agentic RL（OPSD 修复）
├── 09-delta.md                          # 判别器视角 + token credit
├── 10-weak-driven.md                    # 弱 checkpoint 突破后训练饱和
│
│ ── 数据 / 技能合成 ──
├── 02-termigen.md                       # 终端 agent 环境 + 错误注入轨迹合成
├── 03-skillopt.md                       # MSR 自演化 agent 技能 (text-space SGD)
├── 11-skillclaw.md                      # 集体技能进化（多用户生态）
│
│ ── Deep Research ──
├── 04-videodr.md                        # Video Deep Research 基准
├── 07-aris.md                           # 上交对抗式多 agent 自主科研
├── 12-mirothinker.md                    # Verification 嵌入推理过程
│
│ ── 旗舰模型 & Agent 框架 ──
├── 05-youtu-agent.md                    # 腾讯优图 / Training-free GRPO
├── 06-glm-5.md                          # 智谱 GLM-5 旗舰
├── 13-kimi-k25.md                       # Moonshot Kimi K2.5 + Agent Swarm
│
│ ── Multi-Agent & World Model ──
├── 14-gamma-world.md                    # NVIDIA 多 agent world model
├── 15-agentic-world-modeling.md         # Levels × Laws 立论文
├── 16-recursive-mas.md                  # Stanford 递归多 agent
│
│ ── Agent Benchmarks ──
├── 17-skillsbench.md                    # Agent skill 评测标杆
├── 18-clawbench.md                      # 真实生产网站日常任务
├── 19-agents-last-exam.md               # 经济价值未饱和（UC Berkeley）
│
│ ── Harness Engineering ──
└── 20-code-as-harness.md                # Code 作为 agent 基础设施
```

---

## 阅读顺序建议

### 🎯 最短路径（4 篇看完抓住主线）

1. **[GLM-5](06-glm-5.md)** — 看 2026 H1 工业界天花板长什么样
2. **[GrandCode](01-grandcode.md)** — 看 Agentic RL 的极致成就
3. **[TermiGen](02-termigen.md)** — 看数据合成范式
4. **[Youtu-Agent](05-youtu-agent.md)** — 看 agent 自动化的工业落地

### 📚 按分类完整路径

**算法 / RL 进展（4 篇）**：GrandCode → SDAR → DelTA → Weak-Driven

**数据 / 技能合成（3 篇）**：TermiGen → SkillOpt → SkillClaw

**Deep Research 三部曲**（推荐串读）：

- VideoDR 揭示问题（goal drift）
- ARIS 用跨模型对抗审查解决
- MiroThinker 把 verification 嵌入推理过程

**旗舰模型对比**：GLM-5（拼规模）→ Youtu-Agent（拼框架）→ Kimi K2.5（拼推理时并行）

**World Model 三件套**（推荐串读）：

- Agentic World Modeling 给框架（levels × laws）
- Gamma-World 是 L2 Simulator 旗舰落地
- Recursive MAS 提供 multi-agent 的 latent communication

**Benchmark 三角对照**：

- SkillsBench（skill 维度）
- ClawBench（真实日常 web）
- Agents' Last Exam（经济价值真实部署）

**Harness Engineering**：Code as Agent Harness（综述，配合 ARIS 读）

---

## 2026 H1 关键趋势观察

### 1️⃣ Agentic RL 已经"赢了"

- GrandCode 在 Codeforces 直播击败所有人类 grandmaster
- 不再是"能不能赢"的问题，而是"什么场景下赢、赢多少"

### 2️⃣ Data Synthesis Is the New Model Architecture

- TermiGen、Youtu-Agent、CUA-Suite、OpenResearcher 等都聚焦数据/环境合成
- 开源追赶闭源的关键不是模型架构，而是**高保真合成数据 + 自动验证**

### 3️⃣ Training-free 范式崛起

- Youtu-Agent 的 Training-free GRPO：$18 vs $10K 传统 RL
- SkillOpt 的 text-space 优化：完全不更新权重
- 都是在 frontier 闭源模型时代的必然适配

### 4️⃣ Cross-Model Adversarial 取代 Self-Refinement

- ARIS 首次把 "single-model self-review" 形式化为不可靠
- 强制 executor 与 reviewer 来自不同 model family
- MiroThinker 进一步把 verification 嵌入推理过程
- 类似 GAN 思路在科研 / 评估场景被验证有效

### 5️⃣ Skill / Text-Space 成为新的能力载体

- SkillOpt、AgentSPEX、Code as Agent Harness、COLLEAGUE.SKILL、LatentSkill
- 取代 fine-tuning，成为 frozen-model 时代的主流适配方法
- **SkillsBench 实证揭示**：self-generated skill 平均无收益——需要 validation gate + strong evaluator

### 6️⃣ Deep Research 多模态化

- VideoDR（视频）、Vision-DeepResearch（多模态）、OpenSearch-VL（视觉搜索）
- 文本 deep research 已"卷"得差不多，多模态是新蓝海

### 7️⃣ 国产大厂集体上桌

- **腾讯优图**：W01 Youtu-LLM、W02 Youtu-Agent、W15 HY-Embodied
- **智谱**：W08 GLM-5（半年最高票）、W18 GLM-5V-Turbo
- **Moonshot**：W06 Kimi K2.5（差异化打 Agent Swarm 并行）
- **阿里**：W08 RynnBrain、W22 Qwen-VLA
- **字节**：W10 Heterogeneous Agent Collaborative RL、W10 CUDA Agent
- **美团**：W04 EvoCUA、W20 SDAR、W21 AutoResearchClaw

### 8️⃣ "Harness Engineering" 成为新研究主题

- ARIS、Meta-Harness、Code as Agent Harness
- 系统级 logic（context、storage、retrieval）的设计与模型权重同等重要

### 9️⃣ World Model 从孤岛走向统一框架

- 之前 world model 散落在 MBRL / video gen / game sim 等领域
- Agentic World Modeling 用 levels × laws 框架统一
- Gamma-World 把 multi-agent 推到 N>2 player
- Recursive MAS 提出 latent communication 范式

### 🔟 Token 级精细化 RL 调优

- DelTA（判别器视角）、SDAR（sigmoid 门控）、Weak-Driven（entropy dynamics）
- 三个工作从三个角度做 RLVR 精细化——共同语言是**用 token 级统计指标定位问题**

---

## 报告结构说明

每篇精读报告都包含 5 个标准章节：

1. **这篇论文为什么重要** — 一句话核心 + 学界影响背景
2. **核心方法** — 关键技术贡献，配图/公式/伪代码
3. **关键实验结果** — 重要数字、ablation
4. **对领域的影响 / 后续方向** — 学界冲击、局限、揭示的趋势、并行工作
5. **资源** — arXiv / GitHub / 作者 / 联系方式

---

## 文档定位

- 适合**研究人员 + 工程师**快速建立 2026 H1 agent / deep research 领域全景
- 不针对任何特定研究方向，**客观呈现技术本身的贡献和影响**
- 想找特定主题论文，先看本 README 的分类表 或 [`00-summary`](00-summary-2026-W01-W24.md) 的周次表
- 想精读核心工作，按上面分类表的顺序读
