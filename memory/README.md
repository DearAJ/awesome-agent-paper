# Agent Memory 专题 — HuggingFace 月榜精选（2025-11 ~ 2026-06）

> **专题定位**：从 HuggingFace Papers **月榜每月 top50** 中筛出 **Agent Memory** 主线论文做深度阅读，尤其聚焦 **working memory（工作记忆）**——即 **agent 在多轮 / 长程任务中，随着上下文不断增长，那团"当前状态"该怎么表示、更新、使用**。
> **数据范围**：2025-11、2025-12、2026-01、2026-02、2026-03、2026-04、2026-05、2026-06（共 8 个月）
> **分析三维度（贯穿全专题）**：**① 怎么表示（Representation）｜② 怎么更新（Update）｜③ 怎么使用（Usage）**
> **与其他专题的区别**：`deep-research/` 聚焦 web/search agent；`evolve/` 聚焦自演化/skill；`huggingface/` 是每周 top10 全主题。本专题只聚焦 **memory（尤其 working memory）** 一条线，挖得更专。与 `deep-research/` 重叠的 **GAM**、**IterResearch** 仅交叉引用、不重复精读。
> **目录定位**：`Q:/awesome-agent-paper/memory/`

---

## 📚 11 篇精读（9 个文件）— 按"表示 / 更新 / 使用"三维度组织

> **核心论文：[[01-harness-1]]**（三维全覆盖，state externalization 的范式之作）
>
> **怎么表示：		01 · 03 · 07**（环境对象 / 文件系统 / latent 状态矩阵）
>
> **怎么使用：		02 · 04 · 05 · 06**（mask / 编译监督 / 剪枝 / 模式切换）
>
> **怎么更新：		08 · 11**（patch 演化 / skill 化可学）
>
> **理论框架：		09 · 10**（三透镜 / 外化 + 效率）

---

### 🧠 核心：working memory 的"状态外化"范式

> 一句话主线：**不要让 policy 既当大脑又当记账员**——把可恢复的工作记忆外化出去。

| #            | 论文                                       | 月/票      | 机构    | 三维主贡献 | 核心贡献                                                                 |
| ------------ | ------------------------------------------ | ---------- | ------- | ---------- | ------------------------------------------------------------------------ |
| **01** ⭐ | **[Harness-1](01-harness-1.md)**          | 06 / 52↑  | Chroma  | 表示·更新·使用 | **环境侧维护 working memory**（candidate pool / evidence links / verification records）+ budget-aware rendering；policy 只留语义决策；held-out transfer 增益尤强 |

### 📦 怎么**表示**（Representation）— 工作记忆以什么形式承载

| #            | 论文                                       | 月/票      | 机构    | 表示形式 | 核心贡献                                                                 |
| ------------ | ------------------------------------------ | ---------- | ------- | -------- | ------------------------------------------------------------------------ |
| **03** ⭐ | **[FS-Researcher](03-fs-researcher.md)**  | 02 / 52↑  | muset.ai | **文件系统** | 文件系统 = 持久外部记忆 + 跨 agent/session 协调介质；知识库可远超 context length；test-time scaling 有效 |
| **07** ⭐ | **[δ-mem](07-delta-mem.md)**              | 05 / 128↑ | declare-lab | **latent 状态矩阵** | **8×8** 在线关联记忆 + delta-rule 更新 + 对注意力低秩修正；MemoryAgentBench 1.31×，不动 backbone |

### ✂️ 怎么**使用**（Usage）— 工作记忆如何回到上下文供推理

| #            | 论文                                       | 月/票      | 机构    | 使用策略 | 核心贡献                                                                 |
| ------------ | ------------------------------------------ | ---------- | ------- | -------- | ------------------------------------------------------------------------ |
| **02** ⭐ | **[Masking Stale Observations](02-masking.md)** | 06 / 62↑  | McAuley-Lab | **mask 旧观测** | mask 增益呈**非对称倒 U**（regime-dependent）；机制=token-for-turn 权衡；**没有银弹** |
| **05** ⭐ | **[SWE-Pruner](05-swe-pruner.md)**        | 01 / 92↑  | —       | **task-aware 剪枝** | 按当前 goal 用 0.6B 神经 skimmer 选相关代码行；23–54% token 缩减、单轮最高 14.84× |
| **06** ⭐ | **[QwenLong-L1.5](06-qwenlong-l15.md)**   | 12 / 113↑ | Tongyi  | **单遍⇄迭代记忆切换** | 承认"窗口装不下任意长"，多阶段融合 RL 在单遍推理与迭代记忆处理间无缝切换，撑 >4M token |
| **04** ⭐ | **[ACC](04-acc.md)**                      | 05 / 59↑  | USTC    | **编译成监督** | 把 agent 散落多 turn 的证据编译成 long-context QA，训练跨段整合能力；MRCR +18.1 |

### 🔄 怎么**更新**（Update）— 何时、由谁、按什么策略写入/修订

| #            | 论文                                       | 月/票      | 机构    | 更新机制 | 核心贡献                                                                 |
| ------------ | ------------------------------------------ | ---------- | ------- | -------- | ------------------------------------------------------------------------ |
| **08** ⭐ | **[EvoArena / EvoMem](08-evoarena-evomem.md)** | 06 / 90↑  | MIT     | **patch 化演化历史** | 当**环境会变**：用 patch-based 记忆记录 update history；当前 agent 仅 39.6%，揭示动态环境软肋 |
| **11** ⭐ | **[MemSkill + Memento-Skills](11-memskill-memento.md)** | 02·03 / 63↑·58↑ | NTU·UCL | **skill 化可学/可演化** | 把"提取/巩固/剪枝记忆"重构为**可学习、可演化的 memory skills**；不动 LLM 参数靠 skill 演化持续学习 |

### 🗺 理论框架（读懂全专题的坐标系）

| #            | 论文                                       | 月/票      | 机构    | 提供的框架 | 核心贡献                                                                 |
| ------------ | ------------------------------------------ | ---------- | ------- | ---------- | ------------------------------------------------------------------------ |
| **09** ⭐ | **[Memory in the Age of AI Agents](09-memory-survey.md)** | 12 / 157↑ | 社区(2122★) | **forms×functions×dynamics** | agent memory 综述；working memory 定位于 functions；dynamics(formed/evolved/retrieved)≈更新+使用 |
| **10** ⭐ | **[Externalization + Efficient Agents](10-externalization-efficiency.md)** | 04·01 / 52↑·57↑ | SJTU·上海AI Lab | **externalization + 效率 Pareto** | weights→context→harness 演进；memory/skill/protocol/harness 四类外化；+ 关联 Sleep（短期→长期巩固） |

> **去重交叉引用**（已在 `deep-research/` 精读，本专题不重复）：
> - **GAM — General Agentic Memory**（`deep-research/11`）：JIT 编译式记忆，运行时按需检索 = 一次 deep research。
> - **IterResearch**（`deep-research/02`）：Markovian 工作区重构，演进报告作记忆。

---

## 📂 目录结构

```
memory/
├── README.md                          # 本文件 — 总索引（按表示/更新/使用三维度组织）
├── 00-summary-2025.11-2026.06.md      # 8 个月 top50 命中 memory 论文汇总（按月）+ 三维归类 + 趋势
│
│ ── 核心：状态外化 ──
├── 01-harness-1.md                    # ⭐ 环境侧 working memory，policy 只留语义决策
│
│ ── 怎么表示 ──
├── 03-fs-researcher.md                # 文件系统 = 持久外部记忆 + 协调介质
├── 07-delta-mem.md                    # 8×8 latent 状态矩阵 + delta 更新 + 低秩注入
│
│ ── 怎么使用 ──
├── 02-masking.md                      # mask 旧观测的 regime 倒 U（没有银弹）
├── 05-swe-pruner.md                   # task-aware 神经 skimmer 剪枝
├── 06-qwenlong-l15.md                 # 单遍⇄迭代记忆切换，撑 >4M token
├── 04-acc.md                          # 把散落 turn 编译成 long-context 监督
│
│ ── 怎么更新 ──
├── 08-evoarena-evomem.md              # patch 化记忆记录环境演化 + 评测
├── 11-memskill-memento.md             # 记忆操作 = 可学习/可演化的 skill
│
│ ── 理论框架 ──
├── 09-memory-survey.md                # forms × functions × dynamics 三透镜
└── 10-externalization-efficiency.md   # 外化框架 + 效率 Pareto + Sleep 巩固
```

> 文件编号 01–11 按"主题归位"而非时间——同一维度的论文聚在一起，便于按"表示/更新/使用"串读。

---

## 🎯 阅读顺序建议

### 最短路径（4 篇抓住 working-memory 主线）

1. **[09-memory-survey](09-memory-survey.md)** — 先用 forms×functions×dynamics 三透镜建立坐标系
2. **[01-harness-1](01-harness-1.md)** — 看 working memory 的"状态外化"范式（三维全覆盖）
3. **[02-masking](02-masking.md)** — 看"怎么使用"最尖锐的发现：删上下文是 regime-dependent，没有银弹
4. **[11-memskill-memento](11-memskill-memento.md)** — 看"怎么更新"的最高阶答案：记忆操作本身可学/可演化

### 按"表示 / 更新 / 使用"三维串读

**① 怎么表示**（载体谱系）：
- [01-harness-1](01-harness-1.md)（环境结构化对象）→ [03-fs-researcher](03-fs-researcher.md)（文件系统）→ [07-delta-mem](07-delta-mem.md)（latent 状态矩阵）
- 对照轴：**token-level 显式可审计**（01/03）↔ **latent 紧凑不可读**（07）

**② 怎么更新**（写入/修订策略）：
- 环境自动 bookkeeping（[01](01-harness-1.md)）→ patch 化记录演化（[08](08-evoarena-evomem.md)）→ skill 化可学可演化（[11](11-memskill-memento.md)）→ 离线 Sleep 巩固（[10](10-externalization-efficiency.md) 关联）
- 趋势：从**硬编码规则** → **可学习、可演化、可离线巩固**

**③ 怎么使用**（如何回到上下文）：
- mask 旧观测（[02](02-masking.md)）→ task-aware 剪枝（[05](05-swe-pruner.md)）→ 单遍⇄迭代切换（[06](06-qwenlong-l15.md)）→ 编译成训练监督（[04](04-acc.md)）
- 关键张力：**删/缩上下文有收益但强烈 regime-dependent**（[02](02-masking.md) 的倒 U）

### 按技术路线对比串读

- **外化 vs 内化**：[01](01-harness-1.md)/[03](03-fs-researcher.md)（外化到环境/文件，可审计）↔ [07](07-delta-mem.md)（内化到注意力，紧凑）
- **应对"窗口装不下"的三条路**：[03](03-fs-researcher.md)（文件系统扩容）｜[06](06-qwenlong-l15.md)（单遍⇄迭代切换）｜[07](07-delta-mem.md)（固定状态避免增长）｜IterResearch（周期重置，`deep-research/02`）
- **删上下文的两种粒度**：[02](02-masking.md)（删"过时"·按时间）↔ [05](05-swe-pruner.md)（删"任务无关"·按目标）

---

## 📈 2025-11 → 2026-06 关键趋势速览

> 完整论述见 [00-summary 第四节](00-summary-2025.11-2026.06.md)。

1. **【表示】working memory 正在"离开 transcript"**：转向环境侧结构化状态（[01](01-harness-1.md)）、文件系统（[03](03-fs-researcher.md)）、演进报告（IterResearch）、固定状态矩阵（[07](07-delta-mem.md)）。**可恢复的状态不该塞在 policy 里。**
2. **【更新】"谁来记、何时记"成为设计核心**：环境自动 bookkeeping → 周期重构 → 在线 delta → patch 演化 → **RL 学 memorization policy** → **离线 Sleep 巩固**。趋势是**把"怎么记"从硬编码人类先验变成可学习/可演化**（[11](11-memskill-memento.md)）。
3. **【使用】上下文从"全量塞回"变成"主动调度的资源"**：budget rendering（[01](01-harness-1.md)）、mask（[02](02-masking.md)）、剪枝（[05](05-swe-pruner.md)）、按需检索（GAM）、低秩注入（[07](07-delta-mem.md)）、编译监督（[04](04-acc.md)）。**关键发现：没有银弹**——删上下文的收益 = retriever 召回 × 模型隐式过滤容量的交互（[02](02-masking.md)）。
4. **【RL 视角】把记账从 policy 剥离，RL 才学得动**：state externalization 让 held-out transfer 增益尤强（[01](01-harness-1.md)）。
5. **【评测视角】从"静态环境"走向"演化环境"**：[08](08-evoarena-evomem.md) 指出真实部署是动态的，记忆必须记录环境演化；HaluMem 关注记忆幻觉。
6. **【统一框架成型】**：[09](09-memory-survey.md)（forms×functions×dynamics）+ [10](10-externalization-efficiency.md)（externalization）给出两套互补语言。

**仍待解决**：① working memory 的**最优粒度**未有定论；② 在线 delta/patch/skill 库的**更新稳定性**（漂移、膨胀）；③ 删/缩上下文的 **regime 自适应开关**仍缺；④ working↔long-term 的**巩固边界**（Sleep）仍是 PoC；⑤ 记忆**评测**仍多依赖 LLM-judge。

---

## 📝 报告结构说明

每篇精读统一结构（与 `deep-research/`、`huggingface/` 库一致），并额外含 **"表示/更新/使用"三维拆解小节**：

1. **这篇论文为什么重要** — 一句话核心 + 在三维度中的定位
2. **核心方法** — 关键技术贡献，配表/公式/流程图 + **三维度拆解**
3. **关键实验结果** — 重要数字（摘要未披露处标注"需读 PDF"）
4. **对领域的影响 / 后续方向** — 学界冲击、局限、趋势、同方向工作
5. **资源** — arXiv / HF Papers / GitHub / 机构

> **数据来源说明**：论文元数据来自 HuggingFace Papers 月榜（`huggingface.co/papers/month/YYYY-MM`），2025-11 ~ 2026-06 共 8 个月、每月 top50；笔记内容基于 arXiv 摘要（部分含 GitHub README）撰写，规模数字以摘要为准，未披露处标注待查 PDF。与 `deep-research/`、`evolve/`、`huggingface/` 去重，重叠论文仅交叉引用。
