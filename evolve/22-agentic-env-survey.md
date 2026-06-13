# Agentic Environment Engineering 综述 — Agent-环境协同进化的生命周期视角

> **arXiv**：2606.12191（2026.06）｜**机构**：Chinese Academy of Sciences（中科院）
> **HF 月榜**：2026-06 #34，58↑
> **关键词**：Environment Engineering · Agent-Environment Co-Evolution · Symbolic/Neural Synthesis · Four Evolution Pathways · Survey

---

## 1. 这篇论文为什么重要

**一句话**：这是 2026-06 最新的综述，**首次从"环境工程生命周期"视角系统梳理 agentic environment**，并提炼出 agent 在动态环境中进化的**四条路径**与环境演化的**三种范式**——是理解"agent-环境协同进化"的最佳地图。

为什么对"agent evolve"主题至关重要：本库多篇论文（[[03-agent-world]] Agent-World、[[02-evocua]] EvoCUA、[[18-evoarena]] EvoArena）都强调"环境/任务也要进化"。这篇综述把这一思想**系统化、分类化**，明确指出：**环境是驱动模型能力持续进化的关键。**

它填补的空白：环境对 agent 至关重要，但既有工作缺乏**系统分类与深度分析**。综述从环境工程生命周期（modeling → synthesis → evaluation → application）四阶段切入。

---

## 2. 核心方法（综述框架）

### 2.1 环境工程生命周期（四阶段）

```
   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
   │ ① Modeling  │──▶│ ② Synthesis │──▶│ ③ Evaluation│──▶│④ Application│
   │ 环境建模     │   │ 环境合成     │   │ 环境评估     │   │ 环境应用     │
   │             │   │             │   │             │   │             │
   │ 8 个属性     │   │ 符号合成     │   │ 各范式下的    │   │ agent-环境   │
   │ × 8 个领域   │   │ + 神经合成   │   │ 评估方法     │   │ 协同进化     │
   └─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
```

**① Modeling（建模）**：从 **8 个属性 × 8 个领域**刻画代表性环境，分析其发展路径与核心能力

**② Synthesis（合成）**：两种自动环境合成范式
- **Symbolic Synthesis（符号合成）**——基于规则/模板
- **Neural Synthesis（神经合成）**——基于神经网络生成

**③ Evaluation（评估）**：各合成范式下的环境评估方法

**④ Application（应用）**：从 **agent-environment co-evolution** 视角讨论环境应用——这是 evolve 主题的核心

### 2.2 Agent 进化的四条路径（核心贡献 ★）

综述把"agent 在动态环境中进化"刻画为**四个互补视角**——这是本库各论文的精确定位框架：

```
   ┌──────────────────────────────────────────────────────────────┐
   │  Agent Evolution in Dynamic Environments —— 四条路径            │
   │                                                                │
   │  ① Memory-centric Experience Evolution（记忆中心·经验进化）     │
   │     → 本库代表：[[18-evoarena]] EvoMem（记忆进化）             │
   │                                                                │
   │  ② Orchestration-centric Workflow Evolution（编排中心·工作流进化）│
   │     → 本库代表：[[19-onemancompany]] 组织级进化                │
   │                                                                │
   │  ③ Trajectory-centric Offline Evolution（轨迹中心·离线进化）    │
   │     → 本库代表：[[05-skillrl]]/[[09-skillsvote]] 轨迹蒸馏 skill │
   │                                                                │
   │  ④ Exploration-centric Online Evolution（探索中心·在线进化）    │
   │     → 本库代表：[[01-agent0]]/[[03-agent-world]] 在线 RL 演化   │
   └──────────────────────────────────────────────────────────────┘
```

这四条路径几乎**完美覆盖了本库的所有论文**——是理解 evolve 全景的最佳分类法。

### 2.3 环境演化的三种范式

综述还识别出**环境本身**如何演化的三种范式：

| 范式 | 含义 | 本库呼应 |
|------|------|---------|
| **Neural-driven（神经驱动）** | 用神经网络驱动环境演化 | [[04-dreamgym]] reasoning-based experience model |
| **Difficulty-driven（难度驱动）** | 按难度梯度驱动环境演化 | [[01-agent0]] curriculum、[[03-agent-world]] 难度可控合成 |
| **Scaling-driven（规模驱动）** | 靠规模扩展驱动环境演化 | [[03-agent-world]] 环境多样性 scaling |

### 2.4 未来方向

- **Environment-as-a-Service**（环境即服务）
- **Multi-agent Environments**（多 agent 环境）
- **Neural-Symbolic Environments**（神经-符号环境）

---

## 3. 关键内容亮点

### 3.1 把"环境"提升为一等公民

长期以来 agent 研究聚焦"模型/算法"，环境只是背景。这篇综述明确主张：**环境在驱动模型能力的持续进化中扮演关键角色**——把环境工程提升为独立研究对象。

### 3.2 四条进化路径 = 本库的分类底座

综述的"memory / orchestration / trajectory / exploration"四路径，与本库的分类高度吻合：
- 本库"记忆进化" ≈ memory-centric
- 本库"组织级自改进" ≈ orchestration-centric
- 本库"skill 进化（轨迹蒸馏）" ≈ trajectory-centric
- 本库"自演化训练" ≈ exploration-centric

读这篇综述能帮你把本库所有论文**串成一张完整地图**。

### 3.3 与 [[21-agentic-reasoning-survey]] 互补

- [[21-agentic-reasoning-survey]]（UIUC，1月）：从 **agent 能力分层**（foundational/self-evolving/collective）看
- 本篇（中科院，6月）：从 **环境工程生命周期 + 协同进化**看
- 两篇综述一个"以 agent 为中心"、一个"以环境为中心"，**互补构成 evolve 主题的完整框架**

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **确立"环境工程"为独立子领域**
   - modeling → synthesis → evaluation → application 的生命周期视角
   - 把散落的环境合成工作（[[03-agent-world]]、[[02-evocua]] 等）统一组织

2. **四条进化路径成为 agent evolve 的标准分类**
   - memory / orchestration / trajectory / exploration 四视角
   - 为后续研究提供精确定位坐标

3. **"Environment-as-a-Service"前瞻**
   - 预示环境将像云服务一样可按需合成、调用
   - [[03-agent-world]] 用 MCP 接入真实环境已是雏形

### ⚠ 局限（作为综述）

- 2026-06 最新，但综述天然滞后于最前沿工作
- 四条路径的边界有重叠（如 trajectory 与 exploration 常交织）
- 对"环境质量如何度量"的标准化讨论仍开放

### 🔮 揭示的趋势（综述明确提出）

1. **Agent-环境协同进化**是通用智能的关键路径——不能只进化 agent
2. **环境合成（符号 + 神经）**成为可扩展训练的基础设施
3. **Environment-as-a-Service / Neural-Symbolic 环境**是下一步方向

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2606.12191
- **HF Papers**: https://huggingface.co/papers/2606.12191
- **机构**: Chinese Academy of Sciences（中科院）
- **定位**: 本库"环境视角"的框架读物，与 [[21-agentic-reasoning-survey]]（agent 视角）互补
