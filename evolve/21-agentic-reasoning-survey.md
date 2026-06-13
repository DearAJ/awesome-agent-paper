# Agentic Reasoning 综述 — "Self-Evolving Layer" 的立论文

> **arXiv**：2601.12538（2026.01）｜**机构**：University of Illinois at Urbana-Champaign (UIUC)
> **HF 月榜**：2026-01 #4，206↑（本库最高票综述）
> **代码**：https://github.com/weitianxin/Awesome-Agentic-Reasoning （1.3k★）
> **关键词**：Agentic Reasoning · Self-Evolving Layer · Three-Layer Framework · In-context vs Post-training · Survey

---

## 1. 这篇论文为什么重要

**一句话**：这是 2026 H1 **agentic reasoning 领域最权威的体系化综述**，它提出的**三层框架**把"self-evolving agentic reasoning"正式确立为一个独立研究层——本库收录的几乎所有论文，都可以在这个框架里找到位置。

为什么对"agent evolve"主题至关重要：综述明确地把 agent 能力分成三层，**中间那层就是"自演化"**。这给了整个 evolve 主题一个**官方的学术坐标系**。

它的立论基础：LLM 在封闭世界推理很强，但在**开放、动态环境**里挣扎。Agentic reasoning 是范式转变——把 LLM 重构为能**规划、行动、并通过持续交互学习**的自主 agent。

---

## 2. 核心方法（综述框架）

### 2.1 维度一：环境动态的三层（核心贡献）

```
   ┌──────────────────────────────────────────────────────────────┐
   │  Layer 3: Collective Multi-Agent Reasoning（集体多 agent 推理）│
   │    扩展到协作场景：coordination、knowledge sharing、shared goals│
   └──────────────────────────────────────────────────────────────┘
                              ▲ 扩展
   ┌──────────────────────────────────────────────────────────────┐
   │  Layer 2: Self-Evolving Agentic Reasoning（自演化推理）★本库焦点│
   │    研究 agent 如何通过以下方式【精炼自身能力】：               │
   │      • Feedback（反馈）                                        │
   │      • Memory（记忆）                                          │
   │      • Adaptation（适应）                                      │
   └──────────────────────────────────────────────────────────────┘
                              ▲ 精炼
   ┌──────────────────────────────────────────────────────────────┐
   │  Layer 1: Foundational Agentic Reasoning（基础推理）            │
   │    稳定环境下的核心单 agent 能力：                             │
   │      • Planning（规划）• Tool Use（工具使用）• Search（搜索）  │
   └──────────────────────────────────────────────────────────────┘
```

**Layer 2（Self-Evolving）正是本库的主题**——综述把"agent 通过 feedback / memory / adaptation 精炼自身能力"明确定义为一个独立层。本库收录的论文几乎都落在这一层：
- **Feedback 驱动**：[[01-agent0]] 双 agent 反馈、[[08-ctx2skill]] Judge 反馈
- **Memory 驱动**：[[18-evoarena]] EvoMem、[[05-skillrl]] SkillBank
- **Adaptation 驱动**：[[02-evocua]] capability-boundary 适应、[[03-agent-world]] co-evolution

### 2.2 维度二：In-context vs Post-training

综述的第二个分类维度，区分两条实现路线：

| 路线 | 含义 | 成本-效果 | 本库代表 |
|------|------|----------|---------|
| **In-context Reasoning** | 结构化编排扩展 test-time 交互，**无需训练** | 低成本、快迭代 | [[08-ctx2skill]]、[[09-skillsvote]]、HF SkillOpt |
| **Post-training Reasoning** | 通过 RL + SFT **内化**行为，需要训练 | 高成本、能力深 | [[01-agent0]]、[[02-evocua]]、[[05-skillrl]]、[[06-skill1]] |

这条维度精准刻画了 evolve 领域的两大流派——"不训练、改外部状态" vs "训练、改权重"。

### 2.3 跨领域应用与挑战

- **应用领域**：科学、机器人、医疗、自主科研、数学
- **未来挑战**：personalization、long-horizon interaction、world modeling、scalable multi-agent training、governance

---

## 3. 关键内容亮点

### 3.1 把"自演化"从零散工作整合为体系

在这篇综述之前，self-evolution 相关工作（self-refine、reflexion、self-rewarding、skill evolution 等）是**零散的**。综述把它们统一到 **Layer 2** 下，按 feedback / memory / adaptation 三个机制组织——给了整个领域一个清晰的认知地图。

### 3.2 "Thought and Action" 的统一路线图

综述把 agentic reasoning 综合成一条**连接思考与行动**的统一路线图——这正是 agent evolve 的本质：不只"想得更好"，更要"通过行动学得更好"。

### 3.3 治理（Governance）作为未来方向

综述前瞻性地把 **governance（治理）** 列为关键未来方向——这一点在几个月后被 [[09-skillsvote]] SkillsVote（skill 生态治理）等工作直接验证。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **为"agent evolve"提供官方学术坐标系**
   - 三层框架（foundational / self-evolving / collective）成为领域共识
   - 本库的分类（自演化训练 / skill 进化 / 环境协同 / 记忆进化）本质是 Layer 2 的细化

2. **In-context vs Post-training 的二分**澄清了流派之争
   - 帮助研究者定位自己的工作属于哪条路线
   - 两条路线对应不同成本-效果曲线，无绝对优劣

3. **1.3k★ 的配套 Awesome 列表**成为社区入口
   - 持续更新的论文集，是进入该领域的标准起点

### ⚠ 局限（作为综述）

- 综述截止于 2026-01，后续大量 evolve 工作（本库 2-6 月的论文）未及收录
- 三层框架偏"能力分层"，对"机制如何实现"的技术细节着墨有限
- self-evolving layer 内部的子分类可进一步细化

### 🔮 揭示的趋势（综述明确提出）

1. **Self-evolving 是连接 foundational 与 collective 的关键桥梁**
2. **World modeling + long-horizon interaction**是自演化的深水区
3. **Governance**——随着 agent 自主性提升，治理成为必答题（[[09-skillsvote]] 已响应）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2601.12538
- **HF Papers**: https://huggingface.co/papers/2601.12538
- **GitHub（Awesome 列表）**: https://github.com/weitianxin/Awesome-Agentic-Reasoning （1.3k★）
- **机构**: University of Illinois at Urbana-Champaign (UIUC)
- **定位**: 本库的"框架奠基"读物，建议先读本篇再读各专题
