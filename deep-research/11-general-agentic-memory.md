# General Agentic Memory (GAM) — 把"记忆检索"重新定义为一次深度研究

> **arXiv**：2511.18423（2025.11）｜**机构**：BAAI（北京智源人工智能研究院 / Beijing Academy of Artificial Intelligence）
> **HF 月榜**：2025-11 月榜 #5，171↑
> **关键词**：JIT Compilation · Memorizer/Researcher · Page-store · Memory-as-DeepResearch
> **GitHub**：[VectorSpaceLab/general-agentic-memory](https://github.com/VectorSpaceLab/general-agentic-memory)（855★）

---

## 1. 这篇论文为什么重要

**一句话**：GAM 指出主流的**静态记忆（static memory）**——"提前把现成可用的记忆构建好"——必然伴随**严重的信息损失**；它转而借用编译器的 **JIT（Just-In-Time，即时）编译原则**，把记忆系统拆成 **Memorizer + Researcher** 二元结构，在**运行时**为每个 client 请求**现场编译**出最优上下文，而离线阶段只保留"简单但有用"的记忆。

为什么这篇值得收进**深度研究**合集，而不是单纯的"记忆系统"论文：

- 它的核心洞见是 **"记忆检索本身就是一项深度研究任务"**——面对一个 online 请求，与其依赖提前压缩好的静态记忆条目，不如让一个 **Researcher agent** 在完整历史里**主动检索、整合、推理**，像做 deep research 一样把答案"研究"出来。
- 这正面利用了 frontier LLM 的 **agentic 能力 + test-time scalability**：记忆不再是被动的 KV 查表，而是一次可以投入更多推理算力、随难度伸缩的主动探索。
- 整个 duo-design 还支持 **端到端 RL 优化**——把"如何记 + 如何研究"作为一个可训练的整体目标。

这把记忆系统从"**检索范式**"拉向了"**研究范式**"，与 [[02-iterresearch]] 把"演进报告当记忆"、[[09-harness-1]] 把"工作记忆外化到环境侧"属于同一条主线：**状态/记忆不该被提前压死，而应在运行时按需重构**。

---

## 2. 核心方法

### 2.1 诊断：静态记忆的根本缺陷

| 范式 | 做法 | 代价 |
| ---- | ---- | ---- |
| **静态记忆（static memory）** | 离线阶段提前把"现成可用"的记忆构建好（摘要、抽取、索引） | **不可避免的严重信息损失**——离线压缩时不知道未来请求要什么，被丢弃的信息无法恢复 |
| **GAM（JIT 编译）** | 离线只留"简单但有用"的轻量记忆；**运行时**才为具体请求现场编译最优上下文 | 把"压缩决策"推迟到信息需求已知的那一刻 |

类比编译器：静态记忆 ≈ **AOT（提前编译）**，对所有未来输入做一次性优化，难免次优；GAM ≈ **JIT**，等真正的调用上下文出现，再针对性地生成最优代码。

### 2.2 Duo-design：Memorizer + Researcher

GAM 用两个组件分工，对应"离线轻量记 + 在线深度研究"：

| 组件 | 角色 | 离线 / 在线 | 关键设计 |
| ---- | ---- | ---- | ---- |
| **Memorizer**（记忆者） | 高亮关键历史信息 | 离线 | 用一份**轻量记忆（lightweight memory）**标记重点，同时把**完整历史**原样存进 **universal page-store（通用页存储）** |
| **Researcher**（研究者） | 为 online 请求检索 + 整合有用信息 | 在线 | 在 page-store 上**检索并整合**，由 Memorizer 预构建的记忆**引导**这次研究 |

关键点在于 **page-store 保留完整历史**：Memorizer 不做有损压缩，只做"加索引/打高亮"，因此**没有信息损失**；真正的"提炼"延后到 Researcher 在运行时按请求执行——这就是 JIT 的精髓。

$$
\underbrace{\text{Memorizer}}_{\text{离线：轻量记忆} + \text{完整 page-store}} \;\xrightarrow{\text{预构建记忆引导}}\; \underbrace{\text{Researcher}}_{\text{在线：检索} + \text{整合} = \text{一次 deep research}} \;\Rightarrow\; \text{为该请求现场编译的最优上下文}
$$

### 2.3 为什么能"即时研究"：借力 frontier LLM

- **agentic capability**：Researcher 能像研究 agent 一样多步检索、整合 page-store 中的证据。
- **test-time scalability**：难的请求可以投入更多推理 / 检索步数，记忆质量随算力伸缩。
- **端到端 RL 可优化**：Memorizer 怎么记、Researcher 怎么研究，可作为一个整体目标用强化学习联合优化。

> 摘要未披露 page-store 的具体数据结构、Memorizer 轻量记忆的实现形式（embedding？摘要？图？）、以及 RL 的具体算法与奖励设计，需读 PDF。

---

## 3. 关键实验结果

| 评测设置 | 结果 | 说明 |
| -------- | ---- | ---- |
| 多种 **memory-grounded task completion** 场景（vs. 既有记忆系统） | **"substantial improvement"（大幅提升）** | 摘要仅作定性声明 |

> 摘要未披露具体 benchmark 名称、对比的记忆系统基线、绝对分数与提升幅度，以及所用 LLM backbone 与尺寸——**摘要未披露，需读 PDF**。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **重画"记忆 vs 检索"的边界**：传统观点把"记忆"（提前压缩存储）和"检索"（运行时查取）当两件事；GAM 主张二者应当融合——**离线只负责无损保管 + 轻量索引，运行时才用一次深度研究来"按需检索 + 整合"**。这把记忆系统的智能重心从"压缩算法"转移到了"运行时的 research agent"。
2. **把 deep research 的方法论反哺记忆**：一旦承认"记忆检索 = 深度研究任务"，则所有 DR 领域的进展（多步推理、test-time scaling、RL 训练）都能直接迁移到记忆系统。
3. **JIT 作为 agent 上下文工程的统一隐喻**：与 [[02-iterresearch]] 的"工作区重构"、[[09-harness-1]] 的"环境侧外化状态"共同指向——**不要在不知道需求时就把上下文压死**。

### ⚠ 局限

- "substantial improvement" 缺具体数字，可复现性待 PDF 验证。
- 运行时跑一次 Researcher（深度研究）的**延迟与算力成本**未量化——JIT 省了信息损失，但每次请求都要现场"研究"，开销可能显著高于静态查表。
- universal page-store 在长历史下的**存储与检索可扩展性**未讨论。
- Memorizer 轻量记忆与 Researcher 的**职责边界**如何划分、RL 联合训练是否稳定，摘要未给细节。

### 🔮 揭示的趋势

1. **记忆系统"agent 化"**：记忆模块本身正从静态数据结构演化为内含 LLM agent 的子系统。
2. **"无损保管 + 运行时提炼"成为模板**：离线尽量不丢信息、把有损决策推迟到需求明确时——这一 JIT 思路可能成为长程 agent 上下文管理的通用范式。
3. **记忆 = 一种受控的内部检索式 DR**：未来可能出现"记忆 benchmark 与 DR benchmark 趋同"的现象。

### 📊 同方向工作

- [[02-iterresearch]]：把"演进报告当记忆"+ 周期性重构工作区，与 GAM "运行时按需编译上下文"同属"主动管理而非被动累积"。
- [[09-harness-1]]：把候选池 / 证据链 / 验证记录等**工作记忆外化到环境侧**，与 GAM 的 page-store（环境侧完整历史）思路同源。
- [[01-mirothinker-v1]]：interaction scaling 与 test-time scalability 是 Researcher "按难度伸缩研究深度"的能力来源。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2511.18423
- **HF Papers**: https://huggingface.co/papers/2511.18423（171↑）
- **机构**: BAAI（北京智源人工智能研究院）
- **GitHub**: https://github.com/VectorSpaceLab/general-agentic-memory （855★）
- **开源内容**: GAM 框架（Memorizer + Researcher + universal page-store）
