# Agent0 — 从零数据自演化 Agent（Tool-Integrated 双 Agent 共进化）

> **arXiv**：2511.16043（2025.11）｜**机构**：University of North Carolina at Chapel Hill (UNC)
> **HF 月榜**：2025-11 #17，110↑
> **代码**：https://github.com/aiming-lab/Agent0 （1.2k★）
> **关键词**：Self-Evolution · Zero Data · Multi-step Co-Evolution · Tool-Integrated Reasoning · Curriculum-Executor

---

## 1. 这篇论文为什么重要

**一句话**：Agent0 是 2026 H1「自演化 Agent」主题的**奠基性工作之一**——它证明了一个 LLM agent 可以**完全不依赖任何外部人工标注数据**，仅靠两个从同一 base model 初始化的 agent 互相博弈（curriculum agent 出题、executor agent 解题），就把推理能力大幅提升。

这篇论文回答了自演化研究的核心追问：**当人类高质量数据快要耗尽时，agent 还能不能继续变强？**

传统 RL agent 的根本瓶颈：
- **依赖 human-curated data**——数据规模 = 能力天花板，AI 被"锚定"在人类知识水平
- **已有 self-evolution 框架受限**：① 受 base model 固有能力限制；② 多为**单轮交互**，无法构造涉及 tool use / 动态推理的复杂课程

**Agent0 的方案**：建立两个 agent 之间的**共生竞争（symbiotic competition）**——curriculum agent 不断逼近能力边界出题，executor agent 用工具去解，二者在一个**自强化循环**里 multi-step co-evolution。

---

## 2. 核心方法

### 2.1 双 Agent 共生竞争架构

```
        ┌─────────────────────────────────────────────────┐
        │            同一 base LLM（Qwen3-8B-Base）         │
        └───────────────┬─────────────────┬───────────────┘
                        │ 初始化           │ 初始化
                        ▼                 ▼
            ┌───────────────────┐   ┌───────────────────┐
            │  Curriculum Agent  │   │   Executor Agent   │
            │  （出题者）         │   │   （解题者）        │
            │                    │   │                    │
            │ 提出"前沿难度"任务 │──▶│ 调用工具求解任务    │
            │ （frontier tasks） │   │ （tool-integrated） │
            │                    │◀──│                    │
            └───────────────────┘   └───────────────────┘
                   ▲                          │
                   │  executor 变强了 → 施压   │ 解题成功率
                   │  curriculum 出更难/更      │ 作为双方
                   └──── tool-aware 的题 ───────┘ 的 reward 信号
```

**核心机制**：
1. **Curriculum agent**：学习提出"恰好在 executor 能力边界上"的任务——太简单没信息量、太难无法学习
2. **Executor agent**：集成外部工具（如 code interpreter），学习解出这些任务
3. **共生压力（symbiotic pressure）**：executor 因为有了工具变强 → 反过来逼迫 curriculum agent 构造**更复杂、更 tool-aware** 的任务 → 形成自强化循环

### 2.2 为什么要 "Tool-Integrated"？

这是 Agent0 区别于早期 self-play（如 SPIN、Self-Rewarding）的关键：

- 早期 self-evolution 局限在**模型自身参数知识**里打转——出题和解题都不超出 base model 已知范围，很快饱和
- Agent0 引入**工具**作为"外部能力杠杆"——executor 用工具能解出"纯靠参数知识解不出"的题，于是 curriculum 也被迫出"必须用工具才能解"的题
- **工具打破了 base model 的能力天花板**，让 co-evolution 不会过早收敛

### 2.3 Multi-step（多轮）的意义

- 既有框架多是 **single-round**：出一道题、解一道题、更新一次
- Agent0 是 **multi-step co-evolution**：课程是逐步演进的序列，executor 的每一次能力跃迁都会改变 curriculum 的出题分布
- 这让系统能自动生成**渐进式课程（progressive curriculum）**，而非一次性的静态题库

---

## 3. 关键实验结果

### 3.1 主结果（Qwen3-8B-Base）

| 能力维度 | 相对提升 |
|---------|---------:|
| **数学推理**（mathematical reasoning） | **+18%** |
| **通用推理**（general reasoning benchmarks） | **+24%** |

**关键意义**：这些提升是**在零外部数据**下取得的——没有用任何人工标注的数学题或推理题，全部由 curriculum agent 自动生成。

### 3.2 自演化的核心证据

- 随着 co-evolution 轮数增加，curriculum agent 生成任务的**难度持续上升**（不会卡在某个水平）
- executor 的解题能力曲线**持续爬升**，没有出现早期 self-play 常见的"早熟饱和"
- 工具的引入是关键——消融掉 tool integration 后，co-evolution 很快停滞

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **「Zero-Data Self-Evolution」成为 2026 H1 的显学**
   - Agent0 是这条线最早、引用最广的代表作之一（1.2k★）
   - 与同组后续工作 [[05-skillrl]] SkillRL 一脉相承（同为 aiming-lab）

2. **双 Agent 共生竞争 = "GAN 思路"在 agent 训练的成功落地**
   - curriculum agent ≈ 生成器（出题）、executor agent ≈ 判别器/解题器
   - 与 [[08-ctx2skill]] Ctx2Skill 的 Challenger-Reasoner-Judge 三方自博弈是同源思想

3. **指明了"数据耗尽"时代的可行路径**
   - 不再"喂数据"，而是让 agent 自己"造课程"
   - 与 [[03-agent-world]] Agent-World、[[02-evocua]] EvoCUA 的"环境/任务自合成"形成呼应

### ⚠ 局限

- co-evolution 的稳定性依赖精细的难度调控——curriculum 若出题太激进会导致 executor 学崩（[[08-ctx2skill]] 用 Cross-time Replay 专门解决这种 adversarial collapse）
- 提升幅度仍受 base model 上限影响（工具只能部分突破，不是无限）
- 实验主要在数学/通用推理域，向开放式 agent 任务的迁移性待验证

### 🔮 揭示的趋势

1. **「Self-evolving layer」被正式纳入 agent 能力分层**——见综述 [[21-agentic-reasoning-survey]] 的三层框架（foundational / self-evolving / collective）
2. **Tool-integration 是自演化突破天花板的关键杠杆**
3. **Curriculum 自生成**正在取代人工设计的训练课程

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2511.16043
- **HF Papers**: https://huggingface.co/papers/2511.16043
- **GitHub**: https://github.com/aiming-lab/Agent0 （1.2k★）
- **机构**: University of North Carolina at Chapel Hill（aiming-lab）
- **同组工作**: [[05-skillrl]] SkillRL（同 lab，把 self-evolution 推到 skill 层）
