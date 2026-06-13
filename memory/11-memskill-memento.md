# MemSkill + Memento-Skills — 把"记什么、怎么记"做成可学习、可演化的记忆技能

> **本笔记合并两篇同主线论文（记忆操作 = 可学习/可演化的 skill）：**
> - **MemSkill: Learning and Evolving Memory Skills for Self-Evolving Agents** — arXiv 2602.02474（2026.02，NTU，63↑）｜[GitHub](https://github.com/ViktorAxelsen/MemSkill)（506★）
> - **Memento-Skills: Let Agents Design Agents** — arXiv 2603.18743（2026.03，UCL，58↑）｜[GitHub](https://github.com/Memento-Teams/Memento-Skills)（1478★）
> **关键词**：learnable/evolvable memory skills · controller-executor-designer · Read–Write Reflective Learning · stateful prompts · skill library as memory · closed-loop self-evolution

---

## 1. 这篇（合并）笔记为什么重要

**一句话**：多数 agent 记忆系统靠**一小撮静态、手工设计的操作**来提取记忆——这些固定流程**把"该存什么、怎么改"的人类先验硬编码进去**，在多样交互下僵化、在长历史上低效；MemSkill 与 Memento-Skills 不约而同地把这些操作**重构为可学习、可演化的"记忆技能（memory skills）"**，用闭环让 agent **自己学会怎么记、并持续改进记法**。

它们直击用户最关心的"**怎么更新** working memory"维度里最深的一层——**不是预先规定更新规则，而是让"更新规则"本身可学习、可演化**：

- **戳破"静态记忆操作"的僵化**：extract / consolidate / prune 这些操作若是 hand-designed，就**锁死了人类对"什么重要"的先验**——换个任务分布就失效。
- **把记忆操作 skill 化**：记忆的"提取/巩固/剪枝"变成**结构化、可复用的 routine（skill）**，可被选择、可被评估、可被新增/改写。
- **闭环自演化**：不仅学"选哪个 skill"，还学"**这套 skill 本身够不够好**"——复盘难例、提出改进、新增 skill。
- **Memento-Skills 更进一步**：**不更新 LLM 参数**，全靠**外化的 skill/prompt 演化**实现持续学习，且让 agent **端到端为新任务设计 agent**。

它们与 [[01-harness-1]] 形成关键对话：Harness-1 把记账外化给**人工设计的** harness；MemSkill/Memento 主张**连这套记账规则也学出来、演化出来**——回应了 Harness-1 局限里"确定性记账规则是 hand-crafted"的隐忧。也与 [[08-evoarena-evomem]]（演化对环境的记录）、[[10-externalization-efficiency]]（self-evolving harness 方向）同属"动态/自演化记忆"主线。

---

## 2. 核心方法

### 2.1 共同诊断：静态、手工记忆操作的僵化

| 现状 | 问题 |
| ---- | ---- |
| 一小撮**静态、手工设计**的记忆操作（extract / revise） | **硬编码人类先验**——什么该存、怎么改被写死；多样交互下**僵化**，长历史上**低效** |
| 解法（两篇共识） | 把记忆操作**重构为可学习、可演化的 memory skills** |

### 2.2 MemSkill：controller + executor + designer 的闭环

把记忆操作重构为**可学习、可演化的 memory skills**——提取/巩固/剪枝信息的**结构化、可复用 routine**：

| 角色 | 职责 |
| ---- | ---- |
| **Controller（控制器）** | **学习选择**一小组相关 skill（受 agent skill 设计哲学启发） |
| **Executor（执行器，LLM）** | 产出 **skill-guided 的记忆** |
| **Designer（设计器）** | **周期性复盘难例**（选中的 skill 产出错误/不完整记忆处），**演化 skill 集**——提出改进与新 skill |

$$
\underbrace{\text{Controller 选 skill}}_{\text{学 skill-selection 策略}} \to \underbrace{\text{Executor 产记忆}}_{\text{skill-guided memory}} \to \underbrace{\text{Designer 复盘难例}}_{\text{演化 skill 集}} \to \text{（闭环回 Controller）}
$$

> **双重学习**：既学 **skill-selection 策略**，又学 **skill 集本身**——形成"改进选择 + 改进技能库"的闭环。

### 2.3 Memento-Skills：Read–Write 反思式学习 + 不动参数

一个**通用、可持续学习**的 agent 系统，充当**"设计 agent 的 agent"**：

| 要素 | 做法 |
| ---- | ---- |
| **基底** | **memory-based RL + stateful prompts**；可复用 skill 存为**结构化 markdown 文件**，充当**持久、演化的记忆**（同时编码行为与上下文） |
| **起点** | 从简单基础 skill（Web 搜索、终端操作）出发 |
| **Read 阶段** | **行为可训练的 skill router** 根据当前 stateful prompt **选最相关 skill** |
| **Write 阶段** | 根据新经验**更新/扩充 skill 库** |
| **关键** | **不更新 LLM 参数**——所有适应都通过**外化 skill/prompt 的演化**实现 |

> **Read–Write Reflective Learning**（承自 Memento 2）：读阶段选 skill、写阶段写 skill，闭环持续学习；并能**端到端为新任务设计 agent**，无需人工设计。

### 2.4 用户关心的三维度拆解：表示 / 更新 / 使用

| 维度 | MemSkill / Memento-Skills 的做法 |
| ---- | -------------------------------- |
| **① 怎么表示** | 记忆 = **结构化、可复用的 skill**（MemSkill 的 routine / Memento 的 markdown 文件，编码行为+上下文）；skill 库本身就是持久记忆。 |
| **② 怎么更新（核心）** | **更新规则本身可学习、可演化**：MemSkill 的 Designer 复盘难例演化 skill 集；Memento 的 Write 阶段按经验扩充 skill 库。**这是"怎么更新"的最高阶答案——元级地学习"如何更新记忆"。** |
| **③ 怎么使用** | **按当前状态选 skill 来指导记忆/行为**：MemSkill 的 Controller、Memento 的 skill router 根据 stateful prompt 选最相关 skill 注入。 |

> 两篇都强调**不动 LLM 参数**（Memento 明示），靠外化 skill 的演化实现持续学习——呼应 [[10-externalization-efficiency]] 的 externalization 主线。

---

## 3. 关键实验结果

| 论文 | 评测 | 结果 |
| ---- | ---- | ---- |
| **MemSkill** | **LoCoMo / LongMemEval / HotpotQA / ALFWorld** | 任务性能**超过强基线**，且**跨设置良好泛化**；分析展示 skill 如何演化 |
| **Memento-Skills** | （摘要末尾截断，原文在 GAIA 等通用 agent 基准验证；1478★） | 通过迭代 skill 生成/精炼**渐进提升自身能力**；端到端为新任务设计 agent |

> **核心信号**：MemSkill 在**对话记忆（LoCoMo/LongMemEval）+ 多跳 QA（HotpotQA）+ 具身（ALFWorld）** 四类异质任务上都超基线，证明"**把记忆操作 skill 化并演化**"具备跨域适应性——这正是静态手工操作做不到的。
> 两篇摘要均未给完整数值表（Memento 摘要被截断）；逐基准分数需读 PDF。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **把"记忆操作"从硬编码升级为可学习/可演化**：这是对"该存什么、怎么改"问题的**元级**回答——不再由人预设规则，而是让 agent 在复盘中演化出自己的记法。直接回应 [[01-harness-1]] 等"harness 记账规则是手工设计"的局限。
2. **skill = 持久演化记忆的载体**：Memento 用 markdown skill 文件同时编码"行为 + 上下文"，**不动参数**就实现持续学习——把"记忆"与"技能"统一为同一种外化工件。
3. **闭环自演化范式**：controller/router 选 skill + designer/write 演化 skill，形成"用→复盘→改进"的闭环，是自演化 agent 的可复用骨架。

### ⚠ 局限

- **skill 库自身的增长问题**：skill 不断新增/扩充，**skill 库本身也会膨胀**——这把 working memory 的"上下文增长"问题搬到了"skill 库管理"层（如何检索/去重/淘汰 skill 未充分讨论）。
- **演化稳定性**：Designer/Write 自动改写 skill，长期是否会引入坏 skill、产生漂移或灾难性遗忘，摘要未给收敛性保证。
- 复盘"难例"依赖**正确性信号**——在无 ground-truth 的开放任务上如何判定 skill 产出"错误/不完整"，是隐含挑战。
- 两篇都偏"经验/技能记忆"，与本专题核心的"运行时 working memory"有交集但不完全等同——它们更偏 **experiential memory 的演化**。

### 🔮 揭示的趋势

1. **元学习记忆操作**：从"学怎么解题"到"学怎么记"——记忆策略本身成为学习对象。
2. **skill 即记忆即能力**：markdown skill 把记忆、技能、行为统一为可读、可外化、可演化的工件（与 `evolve/` 专题的 skill 主线交汇）。
3. **不动参数的持续学习**：靠外化 skill 演化实现 lifelong learning，避免微调代价与遗忘——可能成为部署期适应的主流路径。

### 📊 同方向工作

- [[01-harness-1]]：把记账外化给**人工设计**的 harness；MemSkill/Memento 主张**连记法也学/演化**——后者补前者之憾。
- [[08-evoarena-evomem]]：EvoMem 演化"对环境的记录"，MemSkill 演化"记忆操作 skill"——两种演化对象。
- [[10-externalization-efficiency]]：self-evolving harness / shared infrastructure 方向的直接实例。
- `evolve/` 专题（SkillRL、SkillsVote、Ctx2Skill 等）：skill 的发现/治理/演化主线——与本笔记的"记忆 skill 化"高度交叉，可串读。

---

## 5. 资源

- **MemSkill** — arXiv: https://arxiv.org/abs/2602.02474 ｜ HF: https://huggingface.co/papers/2602.02474 （63↑，NTU）｜ GitHub: https://github.com/ViktorAxelsen/MemSkill （506★）
- **Memento-Skills** — arXiv: https://arxiv.org/abs/2603.18743 ｜ HF: https://huggingface.co/papers/2603.18743 （58↑，UCL）｜ GitHub: https://github.com/Memento-Teams/Memento-Skills （1478★）
- **定位**: 本专题"怎么更新"维度的最高阶答案——记忆操作 = 可学习、可演化的 skill
