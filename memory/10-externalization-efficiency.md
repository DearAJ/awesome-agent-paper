# Externalization + Efficient Agents — 两套统一框架：把记忆"外化"出去，并算清效率账

> **本笔记合并两篇互补的框架/综述论文 + 关联一篇离线巩固工作：**
> - **Externalization in LLM Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering** — arXiv 2604.08224（2026.04，SJTU，52↑）
> - **Toward Efficient Agents: Memory, Tool learning, and Planning** — arXiv 2601.14192（2026.01，Shanghai AI Lab，57↑）｜[GitHub](https://github.com/yxf203/Awesome-Efficient-Agents)（261★）
> - **关联**：Language Models Need Sleep: Learning to Self-Modify and Consolidate Memories — arXiv 2606.03979（2026.06，Google，29↑）
> **关键词**：externalization · weights→context→harness · cognitive artifacts · efficiency Pareto · bounding context · memory consolidation · Sleep / Dreaming

---

## 1. 这篇（合并）笔记为什么重要

**一句话**：这两篇综述/框架给本专题提供了**两套互补的"上层语言"**——**Externalization** 回答"**为什么要把记忆外化出去、外化成什么**"（表示与机制），**Efficient Agents** 回答"**外化/管理记忆值不值、怎么算账**"（成本与权衡）；外加 **Sleep** 给出"**外化的短期记忆如何巩固回长期参数**"（更新侧的离线一环）。

它们与 [[09-memory-survey]] 的 forms×functions×dynamics 三透镜并列，构成本专题的**理论支柱**：

- **Externalization 框架**直接命中用户"**怎么表示/怎么更新**"的本质——agent 能力越来越多地**不是改权重，而是重组运行时**：把状态外化进 memory、把过程外化进 skills、把交互外化进 protocols，再用 harness 统一协调。这正是 [[01-harness-1]]/[[03-fs-researcher]] 的理论母体。
- **Efficient Agents** 提醒：working memory 管理的**意义在于成本**——"bounding context via compression and management（用压缩与管理界定上下文）"是记忆侧效率的核心手段，而效率必须放在 **effectiveness-cost Pareto** 上衡量（[[02-masking]]/[[05-swe-pruner]] 的剪枝/mask 本质都是在这条 Pareto 上挪点）。
- **Sleep** 补上一块缺失拼图：working memory（短期、脆弱）如何**蒸馏进长期参数**——对应 [[09-memory-survey]] dynamics 里 formed↔evolved 之间的巩固环节。

---

## 2. 核心框架

### 2.1 Externalization：从 weights → context → harness

核心主张：**LLM agent 越来越靠"重组模型周围的运行时"来获得能力，而非改权重。** 以前指望模型**内部恢复**的能力，现在被**外化**成四类组件：

| 外化对象 | 外化成什么 | 借用的概念 |
| -------- | ---------- | ---------- |
| **Memory** | 把**状态跨时间外化**（externalizes state across time） | cognitive artifacts（认知工件） |
| **Skills** | 把**过程性专长外化**（procedural expertise） | 同上 |
| **Protocols** | 把**交互结构外化**（interaction structure） | 同上 |
| **Harness engineering** | **统一层**——把上述模块协调成**可治理的执行** | 统一/协调 |

历史演进脉络：

$$
\underbrace{\text{weights}}_{\text{改权重}}\ \longrightarrow\ \underbrace{\text{context}}_{\text{改上下文}}\ \longrightarrow\ \underbrace{\text{harness}}_{\text{改运行时/协调层}}
$$

> 关键洞见：agent 基础设施之所以重要，**不是因为多加了辅助组件，而是因为它把"难的认知负担"转化成模型能更可靠解决的形式**——这正是 [[01-harness-1]] "把记账外化给环境、policy 只留语义决策"的理论依据。论文还讨论 **parametric vs externalized capability** 的权衡，以及 **self-evolving harness / shared infrastructure** 等新方向。

### 2.2 Efficient Agents：把记忆管理放上 Pareto 前沿

从 **memory / tool learning / planning** 三组件审视 agent **效率**（latency / tokens / steps 等成本）：

| 组件 | 高层共识原则（不同实现却殊途同归） |
| ---- | ---------------------------------- |
| **Memory** | **bounding context via compression and management**——用压缩与管理**界定上下文** |
| **Tool learning** | 设计 RL reward **最小化工具调用** |
| **Planning** | 用**受控搜索机制**提升效率 |

效率的两种刻画方式（互补）：

- **固定成本预算下比 effectiveness**；
- **同等 effectiveness 下比 cost**；
- 统一视角 = **effectiveness-cost Pareto 前沿**。

> 对本专题的意义：[[02-masking]]（mask 换 turn）、[[05-swe-pruner]]（23–54% token 缩减）、[[01-harness-1]]（budget-aware rendering）本质都是在**这条 Pareto 上做权衡**——"怎么使用 working memory"在效率视角下 = "在 Pareto 上选哪个点"。

### 2.3 Sleep：把短期工作记忆巩固进长期参数（更新侧离线一环）

受人类学习启发的 **Sleep 范式**，补上"短期→长期"的巩固：

| 阶段 | 做什么 |
| ---- | ------ |
| **Memory Consolidation（记忆巩固）** | **向上蒸馏（Knowledge Seeding）**：把 smaller-self 的记忆蒸馏进**更大网络**以扩容量同时保知识；用 **Generalized Distillation**（on-policy 蒸馏 + RL 模仿学习） |
| **Dreaming（做梦）** | **自我提升**：用 RL **自生成合成数据课程**复习新知识、精炼已有能力，无需人类监督 |

> 它填补了 [[09-memory-survey]] dynamics 里 **formed → evolved** 之间"短期脆弱记忆如何固化为长期参数"的空白——working memory 不只在上下文里流转，最终可经"睡眠"沉淀进权重。

### 2.4 用户关心的三维度：三篇如何分工

| 维度 | Externalization | Efficient Agents | Sleep |
| ---- | --------------- | ---------------- | ----- |
| **① 怎么表示** | 记忆=跨时间外化的状态（cognitive artifact）；token-level/外部存储 | （不限形式，关注成本） | 短期记忆→长期参数 |
| **② 怎么更新** | 外化模块 + harness 协调更新 | 压缩与管理来 bound 上下文 | **离线 Sleep 巩固**（蒸馏 + 做梦） |
| **③ 怎么使用** | harness 把外化状态协调进执行 | 在 Pareto 上选用量 | 巩固后参数化复用 |

---

## 3. 关键内容

| 论文 | 性质 | 核心交付 |
| ---- | ---- | -------- |
| **Externalization** | 统一综述 | weights→context→harness 演进；memory/skills/protocols/harness 四类外化；parametric vs externalized 权衡；self-evolving harness 方向 |
| **Efficient Agents** | 效率综述 | memory/tool/planning 三组件效率原则；effectiveness-cost Pareto；效率基准与指标整理 |
| **Sleep** | 方法（proof-of-concept） | Sleep（Consolidation via Knowledge Seeding + Dreaming via RL）；在 long-horizon/持续学习/知识吸收/few-shot 任务验证巩固阶段重要性 |

> 前两篇为综述/框架，不含单一新方法数字；Sleep 为方法论 proof-of-concept，摘要未给具体基准分数（**需读 PDF**）。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **Externalization 给"为什么外化"立了论**：把 Harness-1/FS-Researcher/GAM 等散点工作收编进"weights→context→harness"的统一叙事——agent 能力的增长正从"改权重"转向"改运行时"。
2. **Efficient Agents 给"记忆管理"算了账**：明确 working-memory 管理的价值在 effectiveness-cost Pareto 上——没有免费的压缩，所有 mask/prune/render 都是权衡。
3. **Sleep 桥接短期↔长期**：把"工作记忆如何沉淀为长期能力"做成可操作的离线流程，填补 dynamics 空白。

### ⚠ 局限

- 两篇综述**不提供解法**，只给框架与地图；落地仍需 primary work。
- Externalization 的"parametric vs externalized 何时用哪个"缺乏定量指导。
- Sleep 是 **proof-of-concept**，向上蒸馏（小→大）的稳定性、Dreaming 自生成课程的质量控制未充分验证；规模化代价未知。

### 🔮 揭示的趋势

1. **self-evolving harness / shared infrastructure**：harness 本身将可学习、可演化、可共享（呼应 [[11-memskill-memento]]、RHO）。
2. **效率成为一等指标**：working-memory 研究会越来越多地报告 token/step/latency 而非只报准确率。
3. **短期↔长期巩固闭环**：Sleep 类机制让 agent 把一次任务的工作记忆沉淀成持久能力，逼近持续学习。

### 📊 同方向工作

- [[09-memory-survey]]：forms×functions×dynamics——与 externalization 框架互补（一个看"形式/功能/动态"，一个看"外化/协调"）。
- [[01-harness-1]] / [[03-fs-researcher]]：externalization 框架最直接的实例（memory 外化为 harness 对象 / 文件系统）。
- [[02-masking]] / [[05-swe-pruner]]：Efficient Agents 的 "bounding context" 原则的具体落地与边界。
- [[11-memskill-memento]]：self-evolving harness/skill 方向的代表。

---

## 5. 资源

- **Externalization** — arXiv: https://arxiv.org/abs/2604.08224 ｜ HF: https://huggingface.co/papers/2604.08224 （52↑，SJTU）
- **Efficient Agents** — arXiv: https://arxiv.org/abs/2601.14192 ｜ HF: https://huggingface.co/papers/2601.14192 （57↑，Shanghai AI Lab）｜ GitHub: https://github.com/yxf203/Awesome-Efficient-Agents （261★）
- **Sleep** — arXiv: https://arxiv.org/abs/2606.03979 ｜ HF: https://huggingface.co/papers/2606.03979 （29↑，Google）
- **定位**: 本专题理论支柱——外化（为什么/外化成什么）+ 效率（值不值）+ 巩固（短期→长期）
