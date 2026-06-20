# Agent Evolve 论文库（HuggingFace 月榜 2025-11 ~ 2026-06）

> **数据范围**：HuggingFace Papers 月榜 2025-11 至 2026-06（8 个月），每月 **top 100**
> **筛选主题**：**Agent Evolve / 自演化 Agent**
> **共收录**：**50 篇**（深读 14 📖 + 简述 34 📄 + 跨库引用 2 🔗）
> **目录定位**：`Q:/awesome-agent-paper/evolve/`

> 📌 **精读 vs 简述**：14 篇 📖 精读（独立 md，5 章节）取自每月 top-50 的核心工作；其余 36 篇（top-50 的 6 篇 + top-100 扩展的 28 篇）为 📄 简述，**统一汇总在 [`00-summary`](00-summary-2025-11-2026-06.md) 按月给出要点**，未单独建 md。完整分类、按月详表、研究现状与未来方向，全部见 00-summary。

---

## 📚 收录概览（50 篇，8 大分类）

| 分类 | 篇数 | 精读代表 |
|------|------|---------|
| ① 自演化 Agent 训练框架 | 8 | [Agent0](01-agent0.md)·[EvoCUA](02-evocua.md)·[Agent-World](03-agent-world.md)·[DreamGym](04-dreamgym.md) |
| ② Skill 进化 | 13 | [SkillRL](05-skillrl.md)·[Skill1](06-skill1.md)·[PSN](07-psn.md)·[Ctx2Skill](08-ctx2skill.md)·[SkillsVote](09-skillsvote.md) |
| ③ 演化式优化（LLM×Evolution） | 6 | [CSE](13-cse.md)·[QuantaAlpha](14-quantaalpha.md) |
| ④ 环境/课程协同进化 | 5 | （见 [Agent-World](03-agent-world.md) + 00-summary） |
| ⑤ 记忆进化 | 6 | [EvoArena](18-evoarena.md) |
| ⑥ 组织级/元自改进 | 5 | （见 00-summary：Hyperagents/Memento/CORAL…） |
| ⑦ 多模态/GUI 自演化 | 5 | （见 00-summary：Agent0-VL/MM-Zero/SpatialEvo…） |
| ⑧ 综述 | 2 | [Agentic Reasoning](21-agentic-reasoning-survey.md)·[Agentic Env](22-agentic-env-survey.md) |

---

## 📖 14 篇精读笔记（按主题分类，编号即文件号）

> **自演化训练框架：01-04**　|　**Skill 进化：05-12**　|　**演化式优化：13-15**
> **环境/课程协同进化：16-17**　|　**记忆进化：18**　|　**组织级自改进：19-20**　|　**综述：21-22**
> （编号 10/15/16/17/19/20 为 📄 简述，见 [00-summary](00-summary-2025-11-2026-06.md)）

---

### 🔥 自演化 Agent 训练框架（4 篇）

> agent 不再"喂数据"，而是自己造课程 / 经验 / 环境，靠 co-evolution 持续突破能力天花板。

| #            | 论文                                  | 月/票        | 机构         | 核心贡献 |
| ------------ | ------------------------------------- | ------------ | ------------ | -------- |
| **01** | **[Agent0](01-agent0.md)** 📖        | 25-11 / 110↑ | UNC          | 零数据双 agent 共生竞争（curriculum↔executor），tool-integrated，数学 +18%/通用 +24% |
| **02** | **[EvoCUA](02-evocua.md)** 📖        | 26-01 / 92↑  | 美团         | CUA 数据生成⇄策略优化自循环，失败转富监督，OSWorld **56.7%** 刷新开源 SOTA 反超闭源 |
| **03** | **[Agent-World](03-agent-world.md)** 📖 | 26-04 / 85↑  | 字节 Seed    | 自演化竞技场，**agent-环境协同进化**，揭示"环境多样性×演化轮数"scaling |
| **04** | **[DreamGym](04-dreamgym.md)** 📖    | 25-11 / 83↑  | Meta         | 首个经验合成框架，reasoning-based experience model，WebArena +30%，sim-to-real warm-start |

### 🎯 Skill 进化（8 篇）

> 2026 H1 最密集的战场——三大流派（神经 RL / 程序化 / 自然语言文档）并立，关键词从"产生 skill"转向"治理 skill"。

| #            | 论文                                | 月/票         | 机构          | 核心贡献 |
| ------------ | ----------------------------------- | ------------- | ------------- | -------- |
| **05** | **[SkillRL](05-skillrl.md)** 📖    | 26-02 / 76↑   | UNC           | 经验蒸馏成 SkillBank（存模式不存轨迹）+ skill 库与 policy **递归共进化**，+15.3% |
| **06** | **[Skill1](06-skill1.md)** 📖      | 26-05 / 112↑  | —             | 单 policy + 单信号统一进化 skill 选择/使用/蒸馏，**频率分解 credit** |
| **07** | **[PSN](07-psn.md)** 📖            | 26-01 / 88↑   | 蒙特利尔大学  | **程序化 skill 进化**（可执行符号程序网络），maturity-aware gating，与神经网络训练同构 |
| **08** | **[Ctx2Skill](08-ctx2skill.md)** 📖 | 26-05 / 166↑  | —             | **无监督无外部反馈**三方自博弈（Challenger-Reasoner-Judge），Cross-time Replay 防崩溃 |
| **09** | **[SkillsVote](09-skillsvote.md)** 📖 | 26-05 / 126↑  | 记忆张量      | **skill 生态治理**（采集→推荐→进化），**evidence-gated update** 防污染，Terminal-Bench +7.9pp |
| **10** | **[SkillNet](#)** 📄（见 00）       | 26-03 / 94↑   | 浙大          | **20 万 skill** 开源基础设施，统一 ontology + 5 维评估，奖励 +40% 步数 -30% |
| **11** | **[SkillClaw](../huggingface/11-skillclaw.md)** 🔗 | 26-04 / 293↑  | 阿里 AMAP     | **多用户生态级集体技能进化**（已在 huggingface/ 精读） |
| **12** | **[SkillOpt](../huggingface/03-skillopt.md)** 🔗 | 26-05 / 227↑  | Microsoft     | **text-space skill 优化器**，52/52 全胜（已在 huggingface/ 精读） |

### 🧬 演化式优化 / 搜索（LLM × Evolution，3 篇）

> 把遗传算法的变异/交叉搬到 LLM——从盲目随机升级为反馈引导的受控演化。

| #            | 论文                                  | 月/票         | 机构        | 核心贡献 |
| ------------ | ------------------------------------- | ------------- | ----------- | -------- |
| **13** | **[CSE](13-cse.md)** 📖              | 26-01 / 115↑  | QuantaAlpha | **受控自演化**代码优化，多样化初始化+反馈引导遗传进化+层级记忆，EffiBench-X SOTA |
| **14** | **[QuantaAlpha](14-quantaalpha.md)** 📖 | 26-02 / 190↑  | QuantaAlpha | 演化式 alpha 挖掘，轨迹级变异/交叉，CSI 300 IC 0.1501，跨市场迁移超额 160%/137% |
| **15** | **[GigaEvo](#)** 📄（见 00）          | 25-11 / 121↑  | AIRI        | **AlphaEvolve 开源复现**，MAP-Elites + LLM 变异算子 + multi-island |

### 🌐 环境 / 课程协同进化（2 篇）

> 让环境/任务也跟着 agent 一起进化——环境从"背景"变成"一等公民"。

| #            | 论文                            | 月/票        | 机构            | 核心贡献 |
| ------------ | ------------------------------- | ------------ | --------------- | -------- |
| **16** | **[AutoEnv](#)** 📄（见 00）    | 25-11 / 92↑  | FoundationAgents| 跨环境学习基准，环境=可因子化分布，$4.12 生成异构世界，AutoEnv-36 |
| **17** | **[OpenGame](#)** 📄（见 00）   | 26-04 / 81↑  | —               | 端到端 web 游戏创建，**Game Skill 从经验进化**（Template + Debug Skill）|

### 🧠 记忆进化（1 篇）

> 把"环境怎么变"显式编码进结构化记忆。

| #            | 论文                              | 月/票        | 机构 | 核心贡献 |
| ------------ | --------------------------------- | ------------ | ---- | -------- |
| **18** | **[EvoArena](18-evoarena.md)** 📖 | 26-06 / 90↑  | NUS等  | 首个**环境演化评测**基准（Terminal/SWE/Persona 三域“演化化”） + **EvoMem**（append-only patch 记忆进化），GAIA +6.1% |

### 🏢 组织级 / 多 Agent 自改进（2 篇）

> 把进化的单位从"单 agent"提升到"agent 组织 / 数据工程"。

| #            | 论文                                | 月/票         | 机构        | 核心贡献 |
| ------------ | ----------------------------------- | ------------- | ----------- | -------- |
| **19** | **[OneManCompany](#)** 📄（见 00）  | 26-04 / 121↑  | —           | 多 agent 升级到**组织层**，Talent Market 动态招募，E²R 树搜索，self-improving organizations |
| **20** | **[ProDa](#)** 📄（见 00）          | 26-04 / 89↑   | OpenDataLab | **数据工程自改进闭环**，失败驱动数据修复（=debugging），fine-tuning 有反馈化 |

### 📖 综述（2 篇）

> agent 视角（能力分层）+ 环境视角（生命周期）双框架——整个 evolve 主题的学术坐标系。

| #            | 论文                                                       | 月/票         | 机构   | 核心贡献 |
| ------------ | ---------------------------------------------------------- | ------------- | ------ | -------- |
| **21** | **[Agentic Reasoning 综述](21-agentic-reasoning-survey.md)** 📖 | 26-01 / 206↑  | UIUC   | 三层框架（foundational/**self-evolving**/collective），确立自演化层 |
| **22** | **[Agentic Env Engineering 综述](22-agentic-env-survey.md)** 📖 | 26-06 / 58↑   | 中科院 | 环境工程生命周期 + **agent 进化四路径** + 环境演化三范式 |

---

## 目录结构

```
Q:/awesome-agent-paper/evolve/
├── README.md                              # 本文件 — 总索引（按分类组织）
├── 00-summary-2025-11-2026-06.md          # 8 个月 top50 命中论文汇总（按月）+ 研究现状与方向
│
│ ── 自演化训练框架 ──
├── 01-agent0.md                           # 零数据双 agent 共生竞争（UNC）
├── 02-evocua.md                           # 演化式 CUA / 合成经验自循环（美团）
├── 03-agent-world.md                      # agent-环境协同进化竞技场（字节）
├── 04-dreamgym.md                         # 经验合成 / reasoning-based experience model（Meta）
│
│ ── Skill 进化 ──
├── 05-skillrl.md                          # skill 库与 policy 递归共进化（UNC）
├── 06-skill1.md                           # 单 policy 统一进化 选择/使用/蒸馏
├── 07-psn.md                              # 程序化 skill 网络进化（蒙特利尔）
├── 08-ctx2skill.md                        # 三方自博弈无监督 skill 进化
├── 09-skillsvote.md                       # skill 生态全生命周期治理（记忆张量）
│   （10 SkillNet / 11 SkillClaw🔗 / 12 SkillOpt🔗 见 00-summary 与 huggingface/）
│
│ ── 演化式优化（LLM × Evolution）──
├── 13-cse.md                              # 受控自演化代码优化（QuantaAlpha）
├── 14-quantaalpha.md                      # 演化式金融 alpha 挖掘（QuantaAlpha）
│   （15 GigaEvo 见 00-summary）
│
│ ── 环境/课程协同进化 ──（16 AutoEnv / 17 OpenGame 见 00-summary）
│
│ ── 记忆进化 ──
├── 18-evoarena.md                         # 环境演化评测 + EvoMem（NUS等）
│
│ ── 组织级自改进 ──（19 OneManCompany / 20 ProDa 见 00-summary）
│
│ ── 综述 ──
├── 21-agentic-reasoning-survey.md         # agent 视角：能力三层框架（UIUC）
└── 22-agentic-env-survey.md               # 环境视角：生命周期 + 四路径（中科院）
```

---

## 阅读顺序建议

### 🎯 最短路径（4 篇抓住主线）

1. **[Agentic Reasoning 综述](21-agentic-reasoning-survey.md)** — 先建立"self-evolving layer"框架
2. **[Agent0](01-agent0.md)** — 看零数据自演化的极致（双 agent 共生竞争）
3. **[SkillRL](05-skillrl.md)** — 看 skill 进化主流范式（库与 policy 共进化）
4. **[Agentic Env Engineering 综述](22-agentic-env-survey.md)** — 用四路径把全部论文串成地图

### 📚 按分类完整路径

- **自演化训练**：Agent0 → DreamGym → EvoCUA → Agent-World（数据→经验→GUI→环境，进化单位逐步上移）
- **Skill 进化三流派**：
  - 神经 RL 派：SkillRL → Skill1
  - 程序化派：PSN
  - 自然语言文档派：SkillsVote →（huggingface/ SkillOpt、SkillClaw）
- **演化式优化**：CSE（代码）→ QuantaAlpha（金融）→ GigaEvo（开源基建）
- **协同进化 / 记忆 / 组织**：Agent-World → EvoArena（记忆）→ OneManCompany（组织）
- **综述对读**：Agentic Reasoning（agent 视角）↔ Agentic Env Engineering（环境视角）

---

## 标记说明

- 📖 = 本库精读（含 5 章节：为什么重要 / 核心方法 / 实验 / 影响 / 资源）
- 📄 = 月榜简述（在 [`00-summary`](00-summary-2025-11-2026-06.md) 按月给出要点，未单独建 md）
- 🔗 = 已在 [`../huggingface/`](../huggingface/) 目录精读，本库只做索引与交叉引用

---

## 关键趋势速览（详见 [00-summary](00-summary-2025-11-2026-06.md) 第二、三节，共 12 条 + 研究现状与 10 个未来方向）

1. **Zero-Data / 自造数据 / Open-World Bootstrap**成为主旋律（自造课程/经验/环境，甚至自造验证信号）
2. **Skill 进化**是最密集战场（13 篇；神经 / 程序化 / 自然语言文档三流派）
3. **Co-Evolution（协同进化）**取代单向训练（GAN 思路的各种变体）
4. **防对抗崩溃 / 防污染**成为自演化系统标配（Cross-time Replay / evidence-gated / maturity-gating / R-Few）
5. **LLM × 进化算法**持续升温（反馈引导取代盲目随机；BES / TTT-Discover / CoPD）
6. 进化的**单位不断上移**（能力→skill→工具→环境→记忆→组织→agent 设计 agent→元进化）
7. **环境**从背景变成一等公民（含百万级 SWE-Universe、确定性几何 SpatialEvo）
8. 多篇综述（agent 视角 + 环境视角 + 外部化视角）定调
9. **Frozen model + 外部状态进化**（不更权重也能变强）
10. **"失败"成为被重视的养料**
11. **自演化向多模态 / 空间 / GUI 全面扩散**（Agent0-VL / MM-Zero / SpatialEvo / UI-Voyager）
12. **元 / 自指自改进成为新前沿**（Hyperagents 的 Darwin Gödel Machine / Memento 设计 agent / Need Sleep 自我修改）

> **未来 10 大方向**（详见 00-summary 3.3）：① 终身跨域 ② 可治理/可对齐 ③ 进化单位上移到生态/元机制 ④ 环境即服务 ⑤ 理论化 ⑥ world model 融合 ⑦ 动态评测 ⑧ 低成本 ⑨ 递归/元自改进 ⑩ 开放世界 bootstrap + 防塌缩
