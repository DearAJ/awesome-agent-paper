# FS-Researcher — 把文件系统当作"持久工作记忆 + 跨 agent 协调介质"

> **arXiv**：2602.01566（2026.02）｜**机构**：muset.ai（muset-ai）
> **HF 月榜**：2026-02 月榜 #32，52↑
> **关键词**：file system as external memory · persistent workspace · dual-agent · Context Builder / Report Writer · test-time scaling
> **GitHub**：[Ignoramus0817/FS-Researcher](https://github.com/Ignoramus0817/FS-Researcher)（30★）

---

## 1. 这篇论文为什么重要

**一句话**：deep research 这类长程任务的轨迹**经常超出模型上下文上限**，把 token 预算同时挤压给"证据收集"和"报告写作"，导致 test-time scaling 失效；FS-Researcher 用**文件系统当持久工作区**，让工作记忆**远超 context length**，并以双 agent 分工绕开窗口限制。

它在用户关心的"**怎么表示** working memory"维度上给出一个极具工程价值的答案——**用文件系统这个最朴素、最可扩展的载体充当外部记忆**：

- **表示的可扩展性**：分层知识库（hierarchical knowledge base）可以**生长到远超任何上下文窗口**——这是 transcript-based 表示根本做不到的。
- **文件系统的双重身份**：它既是 **durable external memory（持久外部记忆）**，又是 **跨 agent / 跨 session 的 shared coordination medium（共享协调介质）**——记忆与协作合一。
- **可验证的 test-time scaling**：报告质量与"分配给 Context Builder 的算力"**正相关**——证明在文件系统范式下，**多投入 = 更好**，长程任务终于能稳定 scale。

它与 [[01-harness-1]] 同属 **state externalization** 主线，但载体不同：Harness-1 把工作记忆做成**结构化 harness 对象**，FS-Researcher 做成**文件系统里的笔记与归档**。与 IterResearch（演进报告，`deep-research/02`）、GAM（page-store，`deep-research/11`）一起，构成"working memory 外化的不同载体谱系"。这一思路也正是 2026 H1 "Code/FS as Agent Harness"潮流（见 `huggingface/` 的 Code-as-Harness）在记忆侧的体现。

---

## 2. 核心方法

### 2.1 诊断：长轨迹挤压 token 预算

deep research 的痛点链条：

$$
\text{长程轨迹} \to \text{超出 context 上限} \to \underbrace{\text{证据收集 与 报告写作 争抢 token}}_{\text{预算被双向挤压}} \to \text{test-time scaling 失效}
$$

单一上下文窗口里，"多搜证据"和"多写报告"是**零和**的——这正是 mono-contextual 范式在 long-form 任务上的死结。

### 2.2 双 agent + 文件系统：解耦收集与写作

FS-Researcher 用**文件系统**把两件事解耦到两个 agent：

| Agent | 角色比喻 | 职责 | 与文件系统的关系 |
| ----- | -------- | ---- | ---------------- |
| **Context Builder** | **图书管理员（librarian）** | 浏览互联网、写**结构化笔记**、把**原始源归档**进分层知识库 | **写入方**：构建可远超 context length 的知识库 |
| **Report Writer** | 撰稿人 | **逐节（section by section）**撰写最终报告，把知识库当**事实来源** | **读取方**：以知识库为 ground truth 取证 |

**文件系统的双重角色**：

- **durable external memory（持久外部记忆）**：知识库落盘，可跨 session 存续、可生长到任意规模。
- **shared coordination medium（共享协调介质）**：两个 agent（乃至跨 session）通过同一文件系统**交接状态**，实现**超越上下文窗口的迭代精炼（iterative refinement）**。

### 2.3 用户关心的三维度拆解：表示 / 更新 / 使用

| 维度 | FS-Researcher 的做法 |
| ---- | -------------------- |
| **① 怎么表示（核心）** | working memory = **文件系统中的分层知识库**：结构化笔记 + 归档的原始源。表示的关键优势是**容量无界**（远超 context length）、**持久**（落盘）、**可共享**（跨 agent/session）。 |
| **② 怎么更新** | 由 **Context Builder** 像图书管理员一样**增量写入**：边浏览边写笔记、边归档原始源。更新是**追加 + 结构化组织**式的（hierarchical），而非覆盖式。 |
| **③ 怎么使用** | **Report Writer 逐节读取**知识库作为事实来源；**按报告章节的需要**从知识库取相关条目——使用是**分段、按需、以知识库为 source of facts**，避免一次性把所有证据塞进上下文。 |

> 摘要未披露：分层知识库的目录/索引结构、笔记的具体 schema、Report Writer 从知识库检索的机制（关键词？向量？文件路径约定？）、两 agent 的具体协议。——**需读 PDF / 源码**。

---

## 3. 关键实验结果

| 评测设置 | 结果 | 说明 |
| -------- | ---- | ---- |
| **DeepResearch Bench** + **DeepConsult**（两个开放式基准） | **跨不同 backbone 均达 SOTA report quality** | 文件系统范式与具体 backbone 解耦 |
| **test-time scaling 验证** | 报告质量与**分配给 Context Builder 的算力正相关** | 证明"多投入收集 = 更好报告"，scaling 有效 |

> **核心信号**：质量随 Context Builder 算力**单调上升**——这正是长程任务梦寐以求的"可预测 test-time scaling"，而文件系统外部记忆是其前提（不再受窗口封顶）。
> 摘要未披露：具体分数、与哪些基线对比、Context Builder 算力的量化轴（步数？token？）——**需读 PDF**。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **文件系统 = 长程 agent 的"无界工作记忆"**：把最普通的文件系统确立为可生长、持久、可共享的外部记忆载体，工程上极易落地，且天然解决"窗口封顶"。
2. **"收集/写作"解耦解开 token 零和**：双 agent + 文件系统把互相挤压的两阶段拆开，各自独立 scale——这是 long-form deep research 质量提升的结构性原因。
3. **把 test-time scaling 落到"记忆载体"上**：证明 scaling 的瓶颈不在推理链长度，而在**工作记忆能否突破上下文窗口**——为长程任务的 scaling 提供了新抓手。

### ⚠ 局限

- **检索机制是质量上限的隐形天花板**：Report Writer 从庞大知识库取证的**召回质量**未量化——若取不到关键笔记，知识库再大也白搭（与 [[02-masking]] 揭示的"retriever recall 决定一切"呼应）。
- 仅在**开放式 long-form 报告**任务验证；对需要精确多跳事实核查的短答案任务是否同样有效，未知。
- 双 agent + 文件系统的**协调开销与延迟**未讨论。
- 知识库**无界生长后的检索可扩展性**（百万级笔记时）未评估。

### 🔮 揭示的趋势

1. **"Code/FS as Harness"延伸到记忆侧**：继"代码当 harness"之后，"文件系统当工作记忆"成为长程 agent 的标准件。
2. **记忆与协作合一**：外部记忆同时充当多 agent 协调介质——记忆不再是单 agent 的私有状态，而是**共享黑板**。
3. **持久工作区 → 跨 session 累积**：落盘的知识库让 agent 能跨任务/跨 session 复用，逼近"持续学习"。

### 📊 同方向工作

- [[01-harness-1]]：同为 state externalization——Harness-1 用**结构化 harness 对象**，FS-Researcher 用**文件系统**；前者偏"环境自动维护 + 预算渲染"，后者偏"图书管理员式归档 + 分段取用"。
- IterResearch（`deep-research/02`）：演进报告作记忆——比文件系统更"压缩"（一份 report），FS-Researcher 更"无损保管"（归档原始源）。
- GAM（`deep-research/11`）：page-store 无损保管 + 运行时检索——与 FS-Researcher 的"知识库 + Report Writer 取用"高度同构。
- [[10-externalization-efficiency]]：把文件系统记忆纳入 externalization 框架的"memory 外化状态跨时间"一支。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2602.01566
- **HF Papers**: https://huggingface.co/papers/2602.01566 （52↑）
- **机构**: muset.ai
- **GitHub**: https://github.com/Ignoramus0817/FS-Researcher （30★）
- **开源内容**: FS-Researcher 双 agent 框架 + 数据
