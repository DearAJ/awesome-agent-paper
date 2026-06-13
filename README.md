# Awesome Agent Paper

> 个人维护的 **AI Agent / Deep Research / Agentic RL** 论文阅读笔记库。
>
> 目标：跟踪 2025 H2 – 2026 H1 LLM Agent 领域的关键工作，按"主题 + 系列"两种方式组织，做到**可检索、可串读、可复用**。

---

## 仓库结构

```
awesome-agent-paper/
├── README.md                      # 本文件，总索引
│
├── huggingface/                   # HuggingFace Daily Papers 周榜精选（按主题分类）
│   ├── README.md                  # 20 篇精读 + 10 大趋势观察
│   ├── 00-summary-2026-W01-W24.md # 24 周 top10 命中论文汇总
│   └── 01~20-*.md                 # 20 篇精读笔记
│
├── agentic-rl/                    # Agentic RL 专题（月榜 top50，agent+RL 双命中）
│   ├── README.md                  # 39 篇收录（15 精读 + 24 简述）+ 8 大趋势
│   ├── 00-summary-2025.11-2026.06.md  # 8 个月命中论文汇总（按月）+ 研究现状与趋势
│   └── 01~15-*.md                 # 15 篇精读笔记
│
├── deep-research/                 # Deep Research 专题（月榜 top50，DR 主线深挖）
│   ├── README.md                  # 13 篇精读 + 7 大趋势
│   ├── 00-summary-2025.11-2026.06.md  # 8 个月 top50 命中 DR 论文汇总 + 研究现状与趋势
│   └── 01~13-*.md                 # 13 篇精读笔记
│
├── evolve/                        # Agent Evolve / 自演化 Agent 专题（月榜 top100）
│   ├── README.md                  # 50 篇收录（14 精读 + 34 简述 + 2 跨库引用）+ 12 条趋势
│   ├── 00-summary-2025-11-2026-06.md  # 8 个月命中论文汇总 + 研究现状与 10 大方向
│   └── 01~22-*.md                 # 14 篇精读笔记
│
├── memory/                        # Agent Memory 专题（月榜 top50，聚焦 working memory）
│   ├── README.md                  # 11 篇精读（9 文件）+ 表示/更新/使用 三维地图
│   ├── 00-summary-2025.11-2026.06.md  # 8 个月命中论文汇总（按类别）+ 趋势
│   └── 01~11-*.md                 # 11 篇精读笔记（9 文件）
│
└── tongyi-deepresearch/           # 通义 DeepResearch 系列（按时间演化）
    ├── README.md                  # 11 篇系列论文笔记 + 演化脉络
    └── papers/                    # 原始 PDF
```

---

## 六条子线索

> 两条**横向**全景（huggingface 周榜、evolve 月榜）+ 四条**纵深**专题（agentic-rl、deep-research、memory、tongyi）。

### 1. [huggingface/](./huggingface/) — 横向：2026 H1 全景

**数据范围**：HuggingFace Papers 周榜 2026-W01 ~ W24（共 24 周）

**筛选方法**：每周 top 10 中标题/摘要明确涉及 Agent / Deep Research / Agentic RL / Tool Use 的论文

**组织方式**：按主题分 7 类，共 20 篇精读

| 主题 | 篇数 | 代表论文 |
|---|---|---|
| Agentic RL & 训练算法 | 4 | GrandCode, SDAR, DelTA, Weak-Driven |
| 数据 / Skill 合成 | 3 | TermiGen, SkillOpt, SkillClaw |
| Deep Research | 3 | VideoDR, ARIS, MiroThinker |
| 旗舰模型 & Agent 框架 | 3 | GLM-5, Youtu-Agent, Kimi K2.5 |
| Multi-Agent & World Model | 3 | Gamma-World, Agentic World Modeling, Recursive MAS |
| Agent Benchmarks | 3 | SkillsBench, ClawBench, Agents' Last Exam |
| Harness Engineering | 1 | Code as Agent Harness |

→ 详见 [huggingface/README.md](./huggingface/README.md)

### 2. [agentic-rl/](./agentic-rl/) — 纵深：Agentic RL 一条线挖到底

**数据范围**：HuggingFace Papers **月榜每月 top50**，2025-11 ~ 2026-06（共 8 个月）

**筛选方法**：标题/摘要/关键词**同时**命中 Agent 信号与 RL 信号（双命中），再剔除 world-model / VLA 机器人 / 视频图像生成 / 纯 RLVR 推理（非 agentic）；已被 huggingface/ 收录者只交叉引用、不重复

**组织方式**：按主题分 5 类，共 15 篇精读（+ 24 简述）

| 主题 | 篇数 | 代表论文 |
|---|---|---|
| RL 算法 / 训练范式 | 4 | DreamGym, ERL, OpenClaw-RL, LLM-in-Sandbox |
| Agent 自演化 & 经验 | 3 | Agent0, EvoCUA, CUDA Agent |
| Multi-Agent RL | 3 | HACRL, MATTRL, WideSeek-R1 |
| Deep Research / Search Agent RL | 3 | IterResearch, DR Tulu, Harness-1 |
| Skill RL | 2 | SkillRL, SKILL0 |

→ 详见 [agentic-rl/README.md](./agentic-rl/README.md)

### 3. [deep-research/](./deep-research/) — 纵深：Deep Research 一条线挖到底

**数据范围**：HuggingFace Papers **月榜每月 top50**，2025-11 ~ 2026-06（共 8 个月）

**筛选方法**：每月 top50 中只取 Deep Research / Search Agent / Web Agent / Agentic Search 主线（比周榜库的 top10 挖得更深），已被 huggingface/ 收录者只交叉引用、不重复

**组织方式**：按主题分 6 类，共 13 篇精读

| 主题 | 篇数 | 代表论文 |
|---|---|---|
| Scaling 新轴（交互/宽度/委派） | 3 | MiroThinker v1.0, IterResearch, SearchSwarm/WideSeek |
| 开源 DR 系统 & 训练范式 | 5 | DR Tulu, Step-DeepResearch, OpenSeeker, OpenResearcher, Harness-1 |
| 多模态 DR | 2 | Vision-DeepResearch, OpenSearch-VL |
| 检索接口（DCI） | 1 | DCI & GrepSeek |
| 记忆即研究 | 1 | General Agentic Memory |
| DR 评测 | 1 | DeepResearchEval / DRIFT / K-BrowseComp |

→ 详见 [deep-research/README.md](./deep-research/README.md)

### 4. [evolve/](./evolve/) — 横向：自演化 Agent 全景

**数据范围**：HuggingFace Papers **月榜每月 top100**，2025-11 ~ 2026-06（共 8 个月）

**筛选方法**：每月 top100 中 Agent Evolve / 自演化 Agent 主线（比 DR 专题更宽的 top100，覆盖 skill 进化、协同进化、记忆进化等全部自演化形态），已被 huggingface/ 收录者只交叉引用、不重复

**组织方式**：按主题分 8 类，共 **50 篇**（14 篇精读 📖 + 34 篇简述 📄 + 2 篇跨库引用 🔗）

| 主题 | 篇数 | 代表论文 |
|---|---|---|
| 自演化 Agent 训练框架 | 8 | Agent0, EvoCUA, Agent-World, DreamGym |
| Skill 进化 | 13 | SkillRL, Skill1, PSN, Ctx2Skill, SkillsVote |
| 演化式优化（LLM×Evolution） | 6 | CSE, QuantaAlpha |
| 环境/课程协同进化 | 5 | Agent-World, AutoEnv, OpenGame |
| 记忆进化 | 6 | EvoArena |
| 组织级/元自改进 | 5 | Hyperagents, Memento, CORAL |
| 多模态/GUI 自演化 | 5 | Agent0-VL, MM-Zero, SpatialEvo |
| 综述 | 2 | Agentic Reasoning, Agentic Env Engineering |

→ 详见 [evolve/README.md](./evolve/README.md)

### 5. [memory/](./memory/) — 纵深：Agent Memory（聚焦 working memory）

**数据范围**：HuggingFace Papers **月榜每月 top50**，2025-11 ~ 2026-06（共 8 个月）

**筛选方法**：每月 top50 中 Agent Memory 主线，**尤其聚焦 working memory**——agent 多轮/长程任务中上下文不断增长时，那团"当前状态"该怎么**表示 / 更新 / 使用**；与 deep-research/ 重叠者（GAM、IterResearch）只交叉引用

**组织方式**：按"表示 / 更新 / 使用"三维度组织，共 **11 篇精读**（9 个文件）

| 维度 | 篇数 | 代表论文 |
|---|---|---|
| 核心（三维全覆盖） | 1 | Harness-1（状态外化范式） |
| 怎么表示 | 2 | FS-Researcher（文件系统）, δ-mem（latent 状态矩阵） |
| 怎么使用 | 4 | Masking, SWE-Pruner, QwenLong-L1.5, ACC |
| 怎么更新 | 2 | EvoArena/EvoMem, MemSkill+Memento |
| 理论框架 | 2 | Memory survey, Externalization+Efficient Agents |

→ 详见 [memory/README.md](./memory/README.md)

### 6. [tongyi-deepresearch/](./tongyi-deepresearch/) — 纵向：通义 DR 系列演化

**对象**：阿里通义实验室开源的 Web Agent / Deep Research 系列（对标 OpenAI Deep Research）

**配套模型**：Tongyi-DeepResearch-30B-A3B（MoE，30B 总参 / 3B 激活）

**11 篇系列论文**，按时间演化串读：

```
WebWalker → WebDancer → WebSailor → WebShaper → WebWatcher
   → WebResearcher → ReSum → WebWeaver → WebSailor-V2
   → AgentFounder（CPT）→ AgentScaler（环境扩展）
```

主线：**数据合成 → Agentic CPT → Agentic SFT → Agentic RL** 端到端方法学

→ 详见 [tongyi-deepresearch/README.md](./tongyi-deepresearch/README.md)

---

## 阅读建议

### 想快速了解 2026 H1 Agent 领域全景
→ 直接看 [huggingface/README.md](./huggingface/README.md) 的"最短路径 4 篇"

### 想把 Agentic RL（agent + 强化学习）一条线挖到底
→ 看 [agentic-rl/README.md](./agentic-rl/README.md)，先读 00-summary 的"研究现状与趋势 8 条"再按 5 类（算法/自演化/MARL/DR-RL/Skill-RL）串读

### 想把 Deep Research 一条线挖到底
→ 看 [deep-research/README.md](./deep-research/README.md)，先读 00-summary 的"研究现状与趋势"再按主题串读

### 想了解自演化 / Skill 进化 Agent
→ 看 [evolve/README.md](./evolve/README.md) 的"最短路径 4 篇"，再按三流派（神经 RL / 程序化 / 自然语言文档）串读 Skill 进化

### 想理解长程 Agent 的上下文/记忆怎么管
→ 看 [memory/README.md](./memory/README.md)，按"表示 / 更新 / 使用"三维度串读，核心读 Harness-1 + Masking

### 想理解一个完整的开源 Deep Research 体系如何搭建
→ 按时间顺序通读 [tongyi-deepresearch/README.md](./tongyi-deepresearch/README.md)

### 想找特定主题
- **Agentic RL（agent + RL 训练）**：**整个 agentic-rl/ 专题**（DreamGym, OpenClaw-RL, HACRL, CUDA Agent…）+ huggingface/01, 08, 09, 10
- **RL 训练算法**：huggingface/01, 08, 09, 10 + agentic-rl/01-04, 08-10 + deep-research/03, 09 + evolve/05, 06
- **数据合成**：huggingface/02, 03, 11 + deep-research/04, 05, 06, 09 + tongyi WebShaper / AgentFounder
- **Deep Research**：huggingface/04, 07, 12 + **整个 deep-research/ 专题** + 整个 tongyi 系列
- **多模态 DR**：deep-research/07, 08
- **检索接口 / DCI**：deep-research/10
- **自演化 / Skill 进化**：**整个 evolve/ 专题** + huggingface/03, 11（SkillOpt, SkillClaw）
- **Agent Memory / 长上下文管理**：**整个 memory/ 专题**（Harness-1, Masking, FS-Researcher, δ-mem…）+ deep-research/11
- **旗舰模型**：huggingface/05, 06, 13
- **World Model**：huggingface/14, 15, 16
- **Benchmarks**：huggingface/17, 18, 19 + deep-research/13 + evolve/18（EvoArena）
- **Harness Engineering**：huggingface/20 + deep-research/09, 10 + memory/01（Harness-1）, 10

---

## 笔记格式约定

每篇精读笔记统一 5 个章节：

1. **为什么重要** — 一句话核心 + 学界影响
2. **核心方法** — 关键技术贡献（配图/公式/伪代码）
3. **关键实验结果** — 重要数字、ablation
4. **领域影响 / 后续方向** — 学界冲击、局限、并行工作
5. **资源** — arXiv / GitHub / 作者

---

## License

笔记内容仅供学习交流，原始论文版权归各作者所有。
