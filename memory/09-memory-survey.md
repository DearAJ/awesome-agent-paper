# Memory in the Age of AI Agents — forms × functions × dynamics 三透镜（本专题的概念地图）

> **arXiv**：2512.13564（2025.12）｜**机构**：（社区综述，维护方 Shichun-Liu 等）
> **HF 月榜**：2025-12 月榜 #4，157↑
> **关键词**：agent memory survey · forms (token/parametric/latent) · functions (factual/experiential/working) · dynamics (formed/evolved/retrieved) · vs RAG / context engineering
> **GitHub**：[Shichun-Liu/Agent-Memory-Paper-List](https://github.com/Shichun-Liu/Agent-Memory-Paper-List)（2122★）

---

## 1. 这篇论文为什么重要

**一句话**：agent memory 研究**爆发式增长却日益碎片化**——动机、实现、评测各不相同，松散的术语进一步模糊了概念；传统的 long/short-term 二分**已不足以**刻画当代 agent 记忆系统的多样性。这篇综述用 **forms（形式）× functions（功能）× dynamics（动态）** 三个透镜，给整个领域画了一张**最新、统一的地图**。

它是**本 memory 专题的概念锚点与阅读地图**，因为它提供的三透镜**恰好对应用户关心的三个维度**，并精确定位了"working memory"在整个版图中的位置：

- **划清边界**：明确把 **agent memory** 与 **LLM memory / RAG / context engineering** 区分开——这对避免"什么都叫记忆"的混乱至关重要。
- **functions 维度直接给出"working memory"的定义坐标**：在 **factual（事实）/ experiential（经验）/ working（工作）** 的细分里，**working memory 是独立一类**——这正是本专题聚焦的对象。
- **dynamics 维度 = 用户的"表示/更新/使用"**：**formed（如何形成）→ evolved（如何演化）→ retrieved（如何取回）** 几乎一一对应"怎么更新（formed/evolved）"与"怎么使用（retrieved）"，而 **forms** 对应"怎么表示"。

它为本专题所有精读提供统一坐标：[[01-harness-1]]/[[03-fs-researcher]] 是 **token-level form** 的 working memory，[[07-delta-mem]] 是 **latent form**，[[06-qwenlong-l15]] 触及 **parametric**；[[02-masking]]/[[05-swe-pruner]] 属 dynamics 的 **retrieved/使用** 环节；[[08-evoarena-evomem]]/[[11-memskill-memento]] 属 **evolved/更新** 环节。

---

## 2. 核心方法（综述的组织框架）

### 2.1 先划边界：agent memory ≠ 这些相邻概念

| 概念 | 与 agent memory 的区别（综述强调要分清） |
| ---- | ---------------------------------------- |
| **LLM memory** | 偏模型参数内的知识/记忆 |
| **RAG** | 偏"检索外部知识库增强生成" |
| **context engineering** | 偏"如何排布上下文"这一工程实践 |
| **agent memory** | 更广——涵盖 agent 跨时间的状态形成、演化、取回，含 working memory 这类运行时状态 |

> 综述指出：传统 **long/short-term 二分不足**以覆盖当代多样性，需要更细的三维框架。

### 2.2 三透镜框架

| 透镜 | 子类 | 含义 | 对应"用户三维" |
| ---- | ---- | ---- | -------------- |
| **Forms（形式）** | **token-level / parametric / latent** | 记忆**以什么形式承载**——上下文 token / 模型参数 / 潜状态 | **① 怎么表示** |
| **Functions（功能）** | **factual / experiential / working** | 记忆**起什么作用**——存事实 / 存经验 / 当工作记忆 | （界定研究对象） |
| **Dynamics（动态）** | **formed / evolved / retrieved** | 记忆**随时间如何流转**——如何形成、演化、取回 | **② 更新（formed/evolved）+ ③ 使用（retrieved）** |

$$
\text{Agent Memory} = \underbrace{\text{Forms}}_{\text{token / param / latent}} \times \underbrace{\text{Functions}}_{\text{factual / experiential / } \textbf{working}} \times \underbrace{\text{Dynamics}}_{\text{formed → evolved → retrieved}}
$$

### 2.3 working memory 在框架中的定位（本专题视角）

把本专题的"working memory"放进三透镜：

- **Functions**：明确属于 **working** 一类（区别于 factual 知识库、experiential 经验库）。
- **Forms**：**主要是 token-level**（上下文里的结构化状态，如 [[01-harness-1]] 的 candidate pool），但也有 **latent**（[[07-delta-mem]] 的状态矩阵）与少量 **parametric**（[[06-qwenlong-l15]]）探索。
- **Dynamics**：working memory 是**动态最活跃**的一支——formed/evolved/retrieved 在一个长任务内就要反复发生（对应表示/更新/使用的高频循环）。

---

## 3. 关键内容（综述贡献）

| 贡献 | 内容 |
| ---- | ---- |
| **范围界定** | 清晰划定 agent memory 边界，与 LLM memory / RAG / context engineering 区分 |
| **三透镜分类** | forms × functions × dynamics，取代不足的 long/short-term 二分 |
| **functions 细分** | factual / experiential / **working**——给 working memory 独立定位 |
| **forms 三分** | token-level / parametric / latent——覆盖从上下文到参数到潜状态 |
| **dynamics 三段** | formed / evolved / retrieved——记忆的时间流转 |
| **实践支持** | （摘要末尾被截断，原文含面向落地的资源/分类整理） + 维护 2122★ 论文清单 |

> 这是一篇**框架性综述**，不含单一新方法的实验数字；其价值在于**统一语言 + 分类体系**，可作为读懂本专题所有精读的"坐标系"。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **给碎片化的 agent memory 提供统一语言**：forms × functions × dynamics 三透镜成为讨论记忆系统的共同坐标，缓解"术语松散、各说各话"的乱象。
2. **把 working memory 正式列为一类功能**：在 factual/experiential 之外独立出 working——为本专题这类"运行时状态管理"研究提供了正当的概念位置。
3. **维护活跃的社区论文清单（2122★）**：作为领域的 living survey，持续追踪进展。

### ⚠ 局限

- **综述性质**：提供框架而非解法；具体"怎么做好 working memory"仍需看各 primary work（如本专题精读）。
- 三透镜虽统一，但**边界处仍有交叉**（如 KV-cache 记忆介于 token-level 与 latent 之间；GAM 的 page-store 介于 factual 与 working 之间）——分类并非完全互斥。
- 截至 2025-12，**2026 H1 的 state-externalization 浪潮**（Harness-1 等）尚未纳入——需结合本专题 00-summary 的趋势更新。

### 🔮 揭示的趋势

1. **记忆研究将围绕三透镜组织**：新工作会更自觉地标注自己在 forms/functions/dynamics 中的位置。
2. **working memory 与 long-term memory 的巩固桥接**是空白区（dynamics 的 formed↔evolved 之间）——[[#sleep]] 的 Sleep 范式正填此处。
3. **评测需按 functions 分层**：factual / experiential / working 各需不同评测协议。

### 📊 同方向工作

- [[10-externalization-efficiency]]：另一套互补框架——**externalization（外化）**，从"weights → context → harness"的视角看记忆/skill/protocol；与本综述的 forms 维度（尤其 token-level 外化）呼应。
- 本专题**所有精读**都可用本综述坐标定位：Harness-1/FS-Researcher（token-level working）、δ-mem（latent working）、QwenLong-L1.5（parametric 触及）、Masking/SWE-Pruner（retrieved 动态）、EvoMem/MemSkill（evolved 动态）。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2512.13564
- **HF Papers**: https://huggingface.co/papers/2512.13564 （157↑）
- **GitHub**: https://github.com/Shichun-Liu/Agent-Memory-Paper-List （2122★，living paper list）
- **定位**: 本 memory 专题的概念地图——forms × functions × dynamics 三透镜
