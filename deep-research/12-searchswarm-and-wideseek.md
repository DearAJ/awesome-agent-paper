# SearchSwarm & WideSeek-R1 — 委派智能 vs 宽度扩展：横向扩展 agent 的两条路

> **arXiv**：2606.09730（SearchSwarm，2026.06）｜2602.04634（WideSeek-R1，2026.02）
> **机构**：SearchSwarm ｜ RLinf
> **HF 月榜**：SearchSwarm 2026-06 月榜 #48，49↑ ｜ WideSeek-R1 2026-02 月榜 #30，100↑
> **关键词**：Delegation Intelligence · Width Scaling · MARL vs SFT · Lead-Subagent
> **GitHub**：[Search-Swarm/SearchSwarm](https://github.com/Search-Swarm/SearchSwarm)（63★）｜[RLinf/RLinf](https://github.com/RLinf/RLinf)（3.7k★）

---

## 1. 这篇论文为什么重要

**一句话**：这两篇都主张把 agent **横向扩展（多个 subagent 并行/委派）**而非把单一上下文做得更深，但走了截然不同的两条路——**WideSeek-R1 用多智能体强化学习（MARL）联合训练 lead + 并行 subagent 追求"宽度"**，**SearchSwarm 用 harness 引导的轨迹做 SFT、把"委派决策"内化进权重追求"委派智能"**。

为什么把这两篇放在一起读：

- 主流 deep research 的进展集中在**深度扩展（depth scaling）**——单 agent 靠多轮推理 + 工具调用解决长程问题（见 [[01-mirothinker-v1]] 的 interaction scaling、[[02-iterresearch]] 的 2048 次交互）。
- 但当任务**变宽**（需要同时覆盖大量并列实体/子问题）时，瓶颈从"单体能力"转移到"**组织能力**"——再深的单上下文也会被有限的 context window 卡住。
- 两篇给出了横向扩展的互补答案：**WideSeek 解决"如何把宽任务并行铺开"**，**SearchSwarm 解决"主 agent 何时、把什么委派出去并整合回来"**。

它们与 [[01-mirothinker-v1]] / [[02-iterresearch]] 的**深度/交互扩展**构成正交的另一根轴；而其多 agent 协作的内核又与 [[recursive-mas]]（huggingface/16，多 agent 潜空间通信）形成对照——后者把消息传递压进潜空间，这两篇仍以自然语言摘要在 agent 间通信。

---

## 2. 核心方法

### 2.1 共同前提：context window 有限，宽任务必须横向拆

两篇的出发点一致：现实长程任务的上下文需求**可以无界增长**，但模型 context window **天然有限**。于是采用 **主 agent 分解任务 → 派发给 subagent 执行 → subagent 只回传摘要结果**的范式，以节省主 agent 的上下文预算。区别在于"怎么把这个范式做对"。

### 2.2 SearchSwarm — 委派智能（Delegation Intelligence），靠 SFT 内化

**核心问题**：把上述范式做好需要 **delegation intelligence**——(1) 分解复杂任务、(2) 判断**何时**以及**把什么**委派出去、(3) 把回传结果整合进正在进行的工作流。而这种能力在自然文本里**几乎没有训练数据**。

**解法**：用 **harness（脚手架）** 引导模型走出高质量的"分解 + 委派"轨迹，同时**约束 subagent 把结果以正确格式回传**以支撑主 agent 工作流。

| 步骤 | 做法 |
| ---- | ---- |
| 1. Harness 引导 | 脚手架引导主 agent 做高质量任务分解与委派，并规范 subagent 的结果回传 |
| 2. 轨迹即数据 | harness 引导出的轨迹**天然编码了正确的委派决策** |
| 3. SFT 内化 | 把这些轨迹当作 **监督微调（SFT）数据**，把委派智能**写进模型权重** |

要点：**委派智能不是靠 RL 探索出来的，而是靠 harness 把"正确做法"演示出来、再用 SFT 蒸馏进权重**。这与 [[09-harness-1]] "用 harness 承载状态管理"的工程哲学同源，但目标不同——Harness-1 外化的是 bookkeeping 状态，SearchSwarm 外化的是**委派决策的示范**。

### 2.3 WideSeek-R1 — 宽度扩展（Width Scaling），靠 MARL 联合训练

**核心论点**：研究界过去聚焦 **depth scaling**（单 agent 多轮推理）；当任务变宽，瓶颈从**个体能力**转向**组织能力**。既有多 agent 系统多依赖**手工编排的 workflow + 轮流式交互**，无法有效并行。

**解法**：**lead-agent–subagent 框架**，用 **多智能体强化学习（MARL）** 同时协同"可扩展编排"与"并行执行"。

| 设计 | 内容 |
| ---- | ---- |
| 架构 | lead agent 编排 + 多个 subagent **并行执行** |
| 参数共享 | **共享一个 LLM**，但各 subagent 拥有**隔离的上下文（isolated contexts）** + 专用工具 |
| 训练 | 在 **20k 条宽信息检索任务** 上，**联合优化** lead agent 与并行 subagent（MARL）|
| 缩放性质 | 随**并行 subagent 数量增加**，性能**持续提升** |

要点：**lead + subagent 是被一起训练出来的协同体**，而非事后拼装的固定流程——这正是它区别于"手工 workflow 多 agent 系统"之处。

### 2.4 关键对比：两条横向扩展路线

| 维度 | **SearchSwarm** | **WideSeek-R1** |
| ---- | ---- | ---- |
| 目标 | **委派智能**（何时/委派什么 + 整合）| **宽度扩展**（把宽任务并行铺开）|
| 训练方法 | **SFT**（harness 引导轨迹蒸馏）| **MARL**（lead + subagent 联合 RL）|
| 主 agent–subagent 关系 | 主 agent **决策委派**，subagent 规范回传 | lead **编排**，subagent **并行执行**，共享 LLM + 隔离上下文 |
| 扩展方向 | **委派**（depth-of-delegation）| **并行宽度**（breadth）|
| 核心卖点 | 把"正确委派决策"写进权重 | 并行 subagent 越多越好 |

一句话区分：**SearchSwarm 学的是"该不该派、派给谁"，WideSeek 学的是"如何让一群 subagent 并行高效协作"**——前者是 SFT × 委派，后者是 MARL × 宽度。

---

## 3. 关键实验结果

### SearchSwarm-30B-A3B

| Benchmark | 成绩 | 说明 |
| --------- | ---- | ---- |
| **BrowseComp** | **68.1** | 同等规模模型中**最佳** |
| **BrowseComp-ZH** | **73.3** | 同等规模模型中**最佳** |

> 摘要明确为"the best results among all models of comparable scale"。具体对比的同规模基线清单、激活参数（A3B 指约 3B 激活）下的成本未在摘要逐项展开。

### WideSeek-R1-4B

| Benchmark | 成绩 | 说明 |
| --------- | ---- | ---- |
| **WideSearch（item F1）** | **40.0%** | 与单 agent **DeepSeek-R1-671B** 相当——4B vs 671B |
| 并行 subagent 数量 ↑ | **性能持续提升** | 印证 width scaling 的有效性 |

> 关键看点：仅 **4B** 的多 agent 系统在宽信息检索（WideSearch）上**追平 671B 单 agent**，说明"组织能力"可在小模型上通过 MARL 补偿"个体能力"的不足。20k 训练任务的来源构成、具体 subagent 数量-性能曲线明细需读 PDF。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **确立"宽度"为与"深度"正交的扩展轴**：WideSeek 明确把 **width scaling** 与 depth scaling 对立提出，并指出瓶颈在宽任务下从"个体能力"转向"组织能力"——这是对 [[01-mirothinker-v1]] interaction-depth scaling 的互补叙事。
2. **委派智能成为可训练目标**：SearchSwarm 把"何时/委派什么"从工程直觉变成可用 SFT 数据合成与训练的能力，并指出这类训练数据在自然文本中稀缺、此前在开源社区"largely unexplored"。
3. **MARL vs SFT 的方法论分野**：两篇正好示范了多 agent 协作的两种训练取向——**联合 RL 协同优化**（WideSeek）vs **示范蒸馏内化**（SearchSwarm），为后续多 agent 训练提供了清晰的两极参照。

### ⚠ 局限

- **SearchSwarm**：纯 SFT 内化的委派决策能否泛化到 harness 未覆盖的任务分布？摘要自称"preliminary exploration"，且 harness 设计细节、subagent 回传格式约束的具体形式未在摘要展开。
- **WideSeek-R1**：共享 LLM + 隔离上下文在并行 subagent 极多时的**显存/调度成本**未量化；item F1 40.0% 的绝对水平仍不高，宽任务整体仍困难。
- 两篇都未充分讨论**横向（宽度/委派）与纵向（深度/交互）如何组合**——真实长程研究往往两者皆需。

### 🔮 揭示的趋势

1. **agent 扩展从"单轴"走向"二维"**：深度（交互/推理）× 宽度（并行/委派）共同构成性能平面，未来工作可能同时沿两轴扩展。
2. **多 agent 训练范式分化**：MARL（联合优化）与 SFT（轨迹蒸馏）各有适用面——宽度并行偏向前者，委派决策偏向后者。
3. **小模型多 agent 追平大模型单 agent**：WideSeek-4B≈R1-671B 的结果，暗示"组织"可在一定程度替代"规模"，为低成本部署提供新思路。

### 📊 同方向工作

- [[01-mirothinker-v1]] / [[02-iterresearch]]：**深度/交互扩展**——本文宽度/委派扩展的正交互补轴。
- [[recursive-mas]]（huggingface/16）：同样研究多 agent 协作，但把 agent 间通信压进**潜空间（latent communication）**，而本文两篇仍以自然语言摘要通信——通信媒介上的对照。
- [[09-harness-1]]：harness 承载状态管理的工程哲学，与 SearchSwarm 用 harness 引导委派轨迹同源（外化对象不同）。

---

## 5. 资源

### SearchSwarm
- **arXiv**: https://arxiv.org/abs/2606.09730
- **HF Papers**: https://huggingface.co/papers/2606.09730（49↑）
- **机构**: SearchSwarm
- **GitHub**: https://github.com/Search-Swarm/SearchSwarm （63★）
- **开源内容**: harness + 模型权重（SearchSwarm-30B-A3B）+ 训练数据

### WideSeek-R1
- **arXiv**: https://arxiv.org/abs/2602.04634
- **HF Papers**: https://huggingface.co/papers/2602.04634（100↑）
- **机构**: RLinf
- **GitHub**: https://github.com/RLinf/RLinf （3.7k★）
- **开源内容**: RLinf 框架（lead-agent–subagent，MARL 训练）/ WideSeek-R1-4B
