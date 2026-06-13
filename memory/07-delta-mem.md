# δ-mem — 用 8×8 在线状态矩阵把历史压进注意力（latent 工作记忆）

> **arXiv**：2605.12357（2026.05）｜**机构**：declare-lab（mindlab-research / SUTD）
> **HF 月榜**：2026-05 月榜 #7，128↑
> **关键词**：online memory · frozen backbone · associative memory · delta-rule · low-rank attention correction · fixed-size state · MemoryAgentBench
> **GitHub**：[declare-lab/delta-Mem](https://github.com/declare-lab/delta-Mem)（224★）

---

## 1. 这篇论文为什么重要

**一句话**：长期助手与 agent 系统越来越需要**累积并复用历史信息**，但**单纯扩上下文窗口又贵又常常用不好**；δ-mem 给一个**冻结的 full-attention backbone** 外挂一个**紧凑的在线关联记忆**——用 **delta-rule** 把历史压进一个**固定大小（8×8）的状态矩阵**，其 readout 在生成时对注意力做**低秩修正**。

它在用户关心的三个维度上都给出了与主流"token-level 工作记忆"截然不同的答案——一个 **latent（潜）形式** 的工作记忆样本：

- **【表示】不在上下文里堆 token**：工作记忆是一个**8×8 的状态矩阵**，而非上下文里的自然语言——极致紧凑、与序列长度解耦。
- **【更新】delta-rule 在线更新**：历史信息以 **delta 规则**（关联记忆的经典在线学习律）持续写入固定状态，**不动 backbone 权重、不扩上下文**。
- **【使用】低秩修正注意力**：记忆的 readout 直接对 backbone 的注意力计算做 **low-rank correction**——记忆"注入"推理的方式是改注意力，而非拼进 prompt。
- **效果惊人地高效**：仅 **8×8** 的在线状态，平均分就达冻结 backbone 的 **1.10×**、最强非-δ-mem 记忆基线的 **1.15×**；在 **MemoryAgentBench 上 1.31×**、LoCoMo 上 1.20×，且基本保住通用能力——**无需 full fine-tuning、不换 backbone、不显式扩上下文**。

它代表了 working memory 的另一条技术路线：与 [[01-harness-1]]/[[03-fs-researcher]] 的"**外化到环境/文件系统的 token-level 显式状态**"相对，δ-mem 走"**内化到注意力的 latent 隐式状态**"。在 [[09-memory-survey]] 的 forms 分类里，前者是 token-level，δ-mem 是 **latent**——两条路线回答同一个问题（上下文增长怎么办）的方式根本不同。

---

## 2. 核心方法

### 2.1 诊断：扩窗口不是答案

| 路线 | 问题 |
| ---- | ---- |
| **单纯扩上下文窗口** | **成本高**（注意力随长度二次增长），且**常常无法保证有效利用上下文**（长上下文里信息利用率低） |
| **full fine-tuning / 换 backbone** | 代价大、破坏通用能力 |
| **δ-mem** | 在**冻结** backbone 上挂**固定大小在线状态**，低成本、不伤通用能力 |

### 2.2 方法：固定状态矩阵 + delta 更新 + 低秩注入

三个关键设计：

| 组件 | 做法 |
| ---- | ---- |
| **紧凑在线状态（表示）** | 一个**固定大小的状态矩阵**（实验用 **8×8**）作为 associative memory，把过去信息**压缩**进去——大小**与序列长度无关** |
| **delta-rule 更新（更新）** | 用 **delta 规则**在线更新该状态矩阵——关联记忆的标准在线学习律，随生成持续吸收新信息 |
| **低秩注意力修正（使用）** | 用状态矩阵的 **readout** 对 backbone 的**注意力计算**生成**低秩修正（low-rank corrections）**——记忆通过"改注意力"参与生成 |

$$
\underbrace{\text{冻结 backbone}}_{\text{full-attention，权重不动}}\;+\;\underbrace{\text{8×8 状态矩阵}}_{\substack{\text{delta-rule 在线更新}\\\text{压缩历史}}}\;\xrightarrow{\text{readout}}\;\underbrace{\text{对 attention 的低秩修正}}_{\text{记忆注入生成}}
$$

**精髓**：把"长期记忆"实现为**一个直接耦合进注意力计算的紧凑在线状态**——**不需要** full fine-tuning、**不需要**替换 backbone、**不需要**显式上下文扩展。

### 2.3 用户关心的三维度拆解：表示 / 更新 / 使用

| 维度 | δ-mem 的做法 | 与 token-level 路线的对比 |
| ---- | ------------ | ------------------------- |
| **① 怎么表示** | **latent**：8×8 固定状态矩阵（associative memory），与序列长度解耦 | vs Harness-1/FS-Researcher 的**显式 token 状态**（candidate pool / 文件） |
| **② 怎么更新** | **delta-rule 在线更新**固定状态，backbone 冻结 | vs 环境自动 bookkeeping（Harness-1）/ 图书管理员归档（FS-Researcher） |
| **③ 怎么使用** | readout 对**注意力做低秩修正**——记忆"内化"进计算 | vs 把状态**渲染/拼回上下文**（Harness-1 的 budget rendering） |

> δ-mem 的独特之处：**表示、更新、使用全部发生在"模型内部 + 注意力层"，几乎不占上下文 token**——这是应对"上下文增长"的一条与"外化"完全相反的思路：**根本不让历史进入上下文，而是压进一个固定状态。**
> 摘要未披露：delta-rule 的具体形式、readout→低秩修正的数学细节、状态矩阵大小（8×8 vs 更大）的消融全貌——**需读 PDF / 源码**。

---

## 3. 关键实验结果

| 评测设置 | δ-mem 相对表现 | 说明 |
| -------- | -------------- | ---- |
| **平均分** vs 冻结 backbone | **1.10×** | 仅用 8×8 在线状态 |
| **平均分** vs 最强非-δ-mem 记忆基线 | **1.15×** | 超过现有记忆方法 |
| **MemoryAgentBench** | **1.31×** | memory-heavy 基准上增益最大 |
| **LoCoMo** | **1.20×** | 长对话记忆基准 |
| 通用能力 | **largely preserved** | 冻结 backbone + 外挂状态，不伤通用性 |

> **核心信号**：一个 **8×8**（64 个元素！）的在线状态矩阵，就能在 memory-heavy 基准上把冻结 backbone 提升 **31%**——证明**"有效记忆可以极其紧凑"**，只要它**直接耦合进注意力计算**。这与"必须扩大上下文窗口才能记更多"的直觉形成强烈反差。
> 摘要未披露：各基准绝对分数、不同状态矩阵尺寸的曲线、与具体记忆基线（哪些方法）的逐项对照——**需读 PDF**。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **证明"工作记忆可以极致紧凑且 latent"**：8×8 状态矩阵 + 注意力低秩修正，挑战了"记更多必须上下文更长"的默认假设——为 working memory 提供了一条**与序列长度解耦**的路线。
2. **"冻结 backbone + 外挂在线状态"的低成本范式**：无需 full fine-tuning / 换 backbone / 扩上下文，工程代价极小却在 memory-heavy 任务大幅提升——对部署友好。
3. **复活 delta-rule / 关联记忆**：把经典在线学习律用在现代 LLM 的注意力修正上，连接了 fast-weight / 关联记忆传统与当代 agent 记忆需求。

### ⚠ 局限

- **固定 8×8 状态的容量上限**：极紧凑也意味着**容量有界**——超长、超多样的历史是否会"挤爆"这个小状态，摘要未在极端长度下压力测试。
- **记忆是 latent 的 → 不可读、不可审计**：与 token-level 显式工作记忆（Harness-1 的 evidence links / verification records 可读可查）相反，δ-mem 的状态矩阵**无法被人/agent 检视或纠错**。
- 在 **frozen full-attention backbone** 上验证；对已经用稀疏/线性注意力的现代长上下文模型是否同样有效，未知。
- 对**需要精确逐字召回**（如引用原文）的任务，压缩进 8×8 是否丢失细节，待验证。

### 🔮 揭示的趋势

1. **token-level 与 latent 工作记忆将分工共存**：需要**可审计、可外化、可协调**时用 token-level（Harness-1/FS-Researcher）；需要**极致紧凑、低延迟、与长度解耦**时用 latent（δ-mem）。
2. **"记忆即注意力修正"**：把记忆直接作用于注意力计算，而非拼进上下文——可能成为长上下文的高效替代。
3. **固定状态 + 在线更新**：RNN/SSM 式的"固定状态"思想以"注意力外挂"的新形态回归记忆研究。

### 📊 同方向工作

- [[01-harness-1]] / [[03-fs-researcher]]：**token-level 外化**路线——可读可审计但占上下文；δ-mem 是其 **latent 对立面**——紧凑不可读。
- [[06-qwenlong-l15]]：同样承认"扩窗口不够"，但走"单遍 ⇄ 记忆 agent 融合"（仍偏 token-level 迭代处理），δ-mem 走"注意力内化"。
- [[09-memory-survey]]：δ-mem 是 forms 维度 **latent memory** 的代表实例。
- HERMES / MSA（见 `00-summary`）：把 **KV-cache** 当记忆——介于 token-level 与 latent 之间的另一条"架构级"记忆路线。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.12357
- **HF Papers**: https://huggingface.co/papers/2605.12357 （128↑）
- **机构**: declare-lab（SUTD，mindlab-research）
- **GitHub**: https://github.com/declare-lab/delta-Mem （224★）
- **核心基准**: MemoryAgentBench、LoCoMo
