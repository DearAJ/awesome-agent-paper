# ACC — 把 agent 散落多 turn 的"工作记忆"编译成长上下文监督

> **arXiv**：2605.21850（2026.05）｜**机构**：USTC（ustc-community）
> **HF 月榜**：2026-05 月榜 #33，59↑
> **关键词**：Agent Context Compilation · long-context training · scattered evidence · cross-turn coreference · supervision blind spot · MRCR / GraphWalks
> **GitHub**：（摘要未给定，HF 页面注明开源）

---

## 1. 这篇论文为什么重要

**一句话**：agent 解题时产生**海量轨迹**——跨很多 turn 地调工具、收观测，回答原问题所需的证据因此**散落在各个 turn**；但标准 agent SFT **mask 掉 tool 响应、只训练 turn 级工具选择**，造成一个**监督盲区（supervision blind spot）**——这些散落的信号白白浪费。ACC 把轨迹**编译成 long-context QA**，让"跨段整合工作记忆"这件事**第一次被显式监督**。

它从一个独特角度切入用户关心的"**怎么使用** working memory"——**不是在推理时用，而是把'用工作记忆'这件能力本身训练进模型**：

- **指出一个被忽视的事实**：agent 轨迹**本身就是天然的长上下文训练数据**——证据散落多 turn、需要跨距离整合，正是 long-context 推理要练的东西。
- **戳破标准 agent SFT 的盲区**：常规做法 mask tool 响应、只学"下一步调哪个工具"，于是**散落在 tool 响应/环境观测里的证据从不参与监督**——模型没被教过"把这些跨 turn 的碎片拼起来回答原问题"。
- **解法极简且可叠加**：把轨迹转成"原问题 + 跨 turn 的 tool 响应/观测 → 直接作答（不调工具）"的 QA 对，**显式暴露问题与证据的依赖**，可与任何现有 long-context 方法叠加。

它是 working-memory **使用侧** 的训练向解法，与 [[01-harness-1]]（推理时把工作记忆渲染回上下文）形成"训练 vs 推理"的互补：Harness-1 让**运行时**把工作记忆用好，ACC 让模型**先天就会**把散落工作记忆整合好。也与 [[06-qwenlong-l15]] 的长上下文数据合成、daVinci-Agency 的 PR 序列监督同属"**为长程能力造监督信号**"这一族。

---

## 2. 核心方法

### 2.1 诊断：标准 agent SFT 的监督盲区

agent 轨迹的结构与标准 SFT 的处理：

| 轨迹里的内容 | 标准 agent SFT 怎么处理 | 后果 |
| ------------ | ----------------------- | ---- |
| 模型的工具选择 / 动作 | **训练**（turn-level tool selection） | 学会"下一步调哪个工具" |
| tool 响应 / 环境观测（证据所在） | **mask 掉**（不计入 loss） | **散落的证据信号从不被监督** |

> 核心洞见：**"the evidence needed to answer the original question is thus scattered throughout these turns"**，而标准 SFT 把承载这些证据的 tool 响应 mask 掉，于是出现 **supervision blind spot**——模型从未被显式教过"跨 turn 整合证据直接作答"。

### 2.2 Agent Context Compilation（ACC）

把已有 agent 轨迹**编译**成长上下文 QA 对：

$$
\text{轨迹} \;\xrightarrow{\text{ACC 编译}}\; \big(\underbrace{\text{原问题}}_{Q} + \underbrace{\text{跨多 turn 的 tool 响应 / 环境观测}}_{\text{散落证据拼成的长上下文}}\big)\;\Rightarrow\;\underbrace{\text{直接作答（不调工具）}}_{A}
$$

- **数据来源**：search、software engineering、database querying 三类 agent 的轨迹——覆盖多种工具交互形态。
- **关键变换**：把"原问题"与"跨 turn 收集到的 tool 响应/观测"**拼进同一个长上下文**，训练模型**不调工具、直接作答**。
- **效果**：让"问题 ↔ 证据"的**跨段依赖变得显式**，从而**直接监督长上下文推理**（跨 turn coreference resolution、graph traversal），无需额外标注。
- **正交可叠加**：ACC 只是一种**造 SFT 数据**的方法，可与任何 long-context 扩展/训练方法组合。

### 2.3 用户关心的三维度拆解：表示 / 更新 / 使用

| 维度 | ACC 的落点 |
| ---- | ---------- |
| **① 怎么表示** | 把散落多 turn 的工作记忆**重新表示**为"单一长上下文（原问题 + 拼接的跨 turn 证据）"——从"时序分散"压成"空间集中"的一段长 context。 |
| **② 怎么更新** | 不涉及运行时更新；ACC 是**离线把轨迹编译成训练数据**的"一次性变换"。 |
| **③ 怎么使用（核心）** | 训练模型**把散落工作记忆整合起来直接作答**的能力——即"怎么使用跨段证据"这件事被**显式做成监督目标**。这是"使用 working memory"能力的**训练向获取**，区别于推理向的 rendering/masking。 |

> 机制分析亮点：ACC 训练后的模型呈现 **task-adaptive attention restructuring（任务自适应的注意力重构）** 与 **expert specialization（专家特化）**——说明"被显式监督跨段整合"真的改变了模型内部的信息路由方式。

---

## 3. 关键实验结果

| 基准 | 任务性质 | ACC（Qwen3-30B-A3B） | 增益 |
| ---- | -------- | -------------------- | ---- |
| **MRCR** | 跨 turn coreference resolution | **68.3** | **+18.1** |
| **GraphWalks** | extended context 上的 graph traversal | **77.5** | **+7.6** |
| 通用能力（GPQA / MMLU-Pro / AIME / IFEval） | 保持 | 基本无损 | — |

- **关键对照**：用 ACC 训练的 **30B-A3B** 在上述长程依赖任务上**逼近 Qwen3-235B-A22B**——以远小的体量获得可比的长上下文推理能力。
- **机制证据**：注意力**任务自适应重构** + **专家特化**，佐证 ACC 改变了模型整合散落证据的内在机制。

> 这是本专题里少数给出**完整数值对照**的工作，结论扎实：散落在 agent 轨迹里的工作记忆，编译成显式监督后能显著提升模型"跨段整合"的硬实力。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **重新发现 agent 轨迹的价值**：轨迹不只是"训练 tool 选择"的素材——它本身是**高质量、天然的长上下文训练数据**，关键在于**别 mask 掉 tool 响应**。这扭转了标准 agent SFT 的一个默认做法。
2. **把"使用工作记忆"做成可训练目标**：以往"跨 turn 整合证据"被当作模型的隐性能力；ACC 把它**显式化为 QA 监督**，使之可直接优化——填补了 supervision blind spot。
3. **低成本、可叠加的长上下文数据来源**：无需昂贵长文档策划或启发式合成，**复用已有 agent 轨迹**即可规模化产出，且与任何 long-context 方法正交。

### ⚠ 局限

- 依赖**已有 agent 轨迹的质量与多样性**——轨迹覆盖的工具/领域决定了编译出的长上下文分布。
- 训练模型"**不调工具直接作答**"，与"agent 实际要调工具"的部署形态存在分布差异——ACC 练的是"整合能力"，落到 agentic 推理时的迁移程度需进一步验证（摘要称通用能力保持，但未直接测 agentic 任务）。
- 主要在 MRCR / GraphWalks 这类**合成长依赖基准**上验证，真实长程任务收益待观察。

### 🔮 揭示的趋势

1. **"轨迹即数据"成为长上下文训练范式**：agent 跑出来的海量交互，反过来喂养模型的长上下文能力，形成闭环。
2. **监督盲区思维**：审视训练流程里"被 mask / 被忽略"的信号，往往藏着免费的能力增益。
3. **工作记忆能力的"训练向"与"推理向"分工**：ACC（训练模型先天会整合）+ Harness-1/Masking（推理时管好上下文）——两条线将协同。

### 📊 同方向工作

- [[01-harness-1]]：推理时把工作记忆**渲染**回上下文；ACC 训练模型**整合**散落工作记忆——训练 vs 推理互补。
- [[06-qwenlong-l15]]：同样为长上下文造监督（atomic facts 合成多跳题），ACC 则**复用轨迹**——两种长上下文数据合成路线。
- [[02-masking]]：Masking 研究"推理时删多少观测"，ACC 研究"训练时如何让模型用好不删的观测"。
- daVinci-Agency（见 `00-summary` 02 月）：用 PR 序列造长程监督——与 ACC"用 agent 轨迹造长上下文监督"同族。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.21850
- **HF Papers**: https://huggingface.co/papers/2605.21850 （59↑）
- **机构**: USTC（中国科学技术大学，ustc-community）
- **开源**: HF 页面注明开源（具体仓库以页面为准）
- **核心基准**: MRCR（cross-turn coreference）、GraphWalks（graph traversal over extended context）
