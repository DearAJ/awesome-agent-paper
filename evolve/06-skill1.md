# Skill1 — 单 Policy 统一协同进化（选择 / 使用 / 蒸馏）

> **arXiv**：2605.06130（2026.05）｜**机构**：（论文未在 HF 标注组织）
> **HF 月榜**：2026-05 #37，112↑
> **关键词**：Skill Selection · Skill Utilization · Skill Distillation · Single-Policy Co-Evolution · Task-Outcome Reward

---

## 1. 这篇论文为什么重要

**一句话**：Skill1 用**单一 policy + 单一 task-outcome 奖励信号**，同时协同进化"skill 选择、skill 使用、skill 蒸馏"三种能力——解决了既有方法"三种能力各自优化导致进化方向冲突"的核心痛点。

它把"持久 skill 库"的维护拆解为**三个紧耦合的能力**：
1. **Select**：选出相关 skill
2. **Utilize**：执行时用好它
3. **Distill**：从经验里蒸馏出新 skill

**现有方法的根本问题**：把这三种能力**孤立优化**，或用**不同 reward 源**——结果导致**partial and conflicting evolution（局部且互相冲突的进化）**。比如：优化"选择"的 reward 和优化"蒸馏"的 reward 互相打架，整个 skill 系统进化得别别扭扭。

**Skill1 的洞察**：三种能力应该朝**同一个 task-outcome 目标**协同进化，用**一个信号**统一驱动。

---

## 2. 核心方法

### 2.1 单 Policy 的完整闭环

```
   ┌─────────────────────────────────────────────────────────┐
   │            单一 Policy（统一训练）                         │
   │                                                           │
   │  ① 生成 query 检索 skill 库                                │
   │            ↓                                              │
   │  ② re-rank 候选，选出一个 skill        ← Selection（选择） │
   │            ↓                                              │
   │  ③ 条件于该 skill 解决任务            ← Utilization（使用）│
   │            ↓                                              │
   │  ④ 从 trajectory 蒸馏出新 skill        ← Distillation（蒸馏）│
   └────────────────────────────┬────────────────────────────┘
                                │
                                ▼
   ┌─────────────────────────────────────────────────────────┐
   │     单一 Task-Outcome 信号驱动全部学习                     │
   │                                                           │
   │   • 低频趋势（low-frequency trend）   → credit【选择】      │
   │   • 高频变化（high-frequency variation）→ credit【蒸馏】    │
   └─────────────────────────────────────────────────────────┘
```

### 2.2 核心创新：从单一信号里"分离"出对不同能力的 credit

这是 Skill1 最精巧的地方。它**不引入额外 reward 源**，而是从同一个 task-outcome 信号里，按频率分离出对不同能力的归因：

| 信号成分 | 归因到 | 直觉 |
|---------|--------|------|
| **低频趋势**（long-term trend） | **Skill 选择** | 选对 skill 是"慢变量"——影响一段时间内的整体表现趋势 |
| **高频变化**（short-term variation） | **Skill 蒸馏** | 蒸馏出的新 skill 是"快变量"——带来即时的、波动性的改进 |

通过这种**频率分解**，单一奖励信号就能同时给三种能力提供训练 credit，避免了多 reward 源互相冲突。

### 2.3 为什么"统一"很重要？

- **既有方法**：select / utilize / distill 各训各的，目标不一致 → 进化方向打架
- **Skill1**：一个 policy、一个目标——三种能力**朝同一方向协同进化**
- 训练动态实验证实了三种能力确实在 **co-evolve**，消融任一 credit 信号都会破坏进化

---

## 3. 关键实验结果

### 3.1 主结果

| 基准 | 结果 |
|------|------|
| **ALFWorld + WebShop** | 超过既有 skill-based 与 RL 基线 |

### 3.2 关键验证（训练动态 + 消融）

1. **训练动态确认三种能力 co-evolve**——选择、使用、蒸馏三条能力曲线同步上升
2. **消融实验**：移除任一 credit 信号（低频或高频），进化都会退化
   - 移除低频 → 选择能力进化受损
   - 移除高频 → 蒸馏能力进化受损
   - 证明"频率分解归因"是有效且必要的

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **解决了 skill 学习的"多目标冲突"难题**
   - 此前 skill agent 把 select/utilize/distill 拆开训，Skill1 首次统一
   - "单 policy + 单信号 + 频率分解 credit"是优雅的方法论贡献

2. **与 [[05-skillrl]] SkillRL 形成"同主题双子星"**
   - SkillRL：experience-based distillation + recursive co-evolution（强调 skill 库与 policy 共进化）
   - Skill1：单 policy 内统一三种 skill 能力的协同进化（强调 credit 分配）
   - 两者都在 ALFWorld/WebShop 验证，是 2026 H1 skill RL 的代表作

3. **"频率分解做 credit assignment"是可迁移的技巧**
   - 这一思想可推广到其他"多耦合能力共同优化"的场景

### ⚠ 局限

- 频率分解的阈值/方法需要调参，泛化性待验证
- 实验环境结构化（ALFWorld/WebShop），开放域 skill 复杂度更高
- 单 policy 承担四步（query/select/solve/distill）可能有容量瓶颈

### 🔮 揭示的趋势

1. **"统一目标 + 单信号"协同进化**——取代多 reward 源的拼接式优化
2. **Credit assignment 精细化**——从 token 级（[[huggingface 的 SDAR/DelTA]]）延伸到"能力级"
3. **Skill selection/utilization/distillation 三位一体**正在成为 skill agent 的标准能力刻画

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.06130
- **HF Papers**: https://huggingface.co/papers/2605.06130
- **基准**: ALFWorld, WebShop
- **关联工作**: [[05-skillrl]] SkillRL（skill 库与 policy 共进化的姊妹工作）
