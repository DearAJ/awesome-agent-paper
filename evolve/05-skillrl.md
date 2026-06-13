# SkillRL — 递归 Skill 增强的强化学习（Skill 库与 Policy 共进化）

> **arXiv**：2602.08234（2026.02）｜**机构**：University of North Carolina at Chapel Hill (UNC)
> **HF 月榜**：2026-02 #42，76↑
> **代码**：https://github.com/aiming-lab/SkillRL （833★）
> **关键词**：Skill Discovery · Recursive Evolution · SkillBank · Experience-based Distillation · Policy Co-Evolution

---

## 1. 这篇论文为什么重要

**一句话**：SkillRL 让 agent 在 RL 训练过程中**自动从经验里发现 skill、并让 skill 库与 policy 递归地共进化（co-evolve）**——在 ALFWorld / WebShop / 七个 search 任务上比强基线高 **15.3%**，同时**大幅压缩 token 占用**。

它瞄准 LLM agent 的一个根本缺陷：**agent 常常"孤立作战"，无法从过去经验中学习。**

现有 memory-based 方法的问题：
- 主要存**原始轨迹（raw trajectories）**——冗余、噪声重
- agent 无法从中抽取**高层、可复用的行为模式**
- 而这种高层模式恰恰是泛化的关键

**SkillRL 的方案**：在"原始经验"与"policy 改进"之间架一座桥——通过自动 skill discovery + recursive evolution，把噪声轨迹蒸馏成结构化 skill，并让 skill 库随 RL 一起进化。这是 [[01-agent0]] 同组（aiming-lab）把自演化从"推理层"推进到"skill 层"的工作。

---

## 2. 核心方法

### 2.1 三大创新组件

```
┌──────────────────────────────────────────────────────────────┐
│  ① Experience-based Distillation（经验蒸馏）                    │
│     原始轨迹（噪声重） ──蒸馏──▶ 层级化 skill 库【SkillBank】    │
│     抽取高层、可复用的行为模式（而非存原始轨迹）                 │
└──────────────────────────────┬─────────────────────────────────┘
                               │
┌──────────────────────────────┴─────────────────────────────────┐
│  ② Adaptive Retrieval Strategy（自适应检索策略）                 │
│     区分【通用 heuristics】与【任务专属 heuristics】             │
│     执行时按需检索合适的 skill                                   │
└──────────────────────────────┬─────────────────────────────────┘
                               │
┌──────────────────────────────┴─────────────────────────────────┐
│  ③ Recursive Evolution（递归进化机制）—— 核心                    │
│     让 SkillBank 在 RL 过程中【与 agent policy 共进化】          │
│       • policy 变强 → 产出更好轨迹 → 蒸馏出更好 skill            │
│       • 更好 skill → 反过来提升 policy                          │
│     ⟹ skill 库与 policy 互相驱动、递归上升                      │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 三个组件的逻辑

**① Experience-based Distillation —— "存模式，不存轨迹"**
- 既有 memory agent 存 raw trajectory，又长又脏
- SkillRL 把轨迹**蒸馏**成层级化的 SkillBank——存的是"可复用行为模式"
- 直接好处：**token footprint 大幅下降**（不用每次塞一堆原始轨迹进 context）

**② Adaptive Retrieval —— "通用 + 专属"双层检索**
- skill 分两类：通用启发式（general heuristics）vs 任务专属启发式（task-specific）
- 执行时自适应检索——既能用通用经验，也能调专属技巧

**③ Recursive Evolution —— "共进化"的精髓**
- 关键词是 **co-evolve**：skill 库不是训练前固定的，而是**随 RL 一起进化**
- policy 改进 → 轨迹质量提升 → 蒸馏出更优 skill → skill 又提升 policy
- 形成**递归上升**的正反馈——这正是"evolving agents"标题的由来

---

## 3. 关键实验结果

### 3.1 主结果

| 基准 | 结果 |
|------|------|
| **ALFWorld + WebShop + 7 个 search-augmented 任务** | 全面 SOTA，超强基线 **+15.3%** |
| **鲁棒性** | 随任务复杂度上升保持稳健 |
| **Token footprint** | 显著降低（蒸馏 skill 替代原始轨迹） |

**两个亮点**：
1. **+15.3%** 的平均增益，且任务越复杂优势越明显
2. **效率与效果双赢**——既提升推理能力，又压缩了 token 消耗

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **把"自演化"从推理层推进到 skill 层（与 [[01-agent0]] 同组延续）**
   - Agent0：双 agent 共进化（推理能力）
   - SkillRL：skill 库与 policy 共进化（可复用技能）
   - 同一 lab（aiming-lab）的递进式探索

2. **"co-evolve skill 库 + policy"成为 2026 H1 skill 研究主线**
   - 与 [[06-skill1]] Skill1（单 policy 统一协同 selection/utilization/distillation）几乎同期、同思想
   - 与 [[07-psn]] PSN（programmatic skill 网络进化）形成"神经 RL 派 vs 程序化派"对照

3. **"存模式而非存轨迹"——agent memory 的范式升级**
   - 直接呼应 memory evolution 主题（见 [[18-evoarena]]）

### ⚠ 局限

- skill 蒸馏质量依赖底层模型的抽象能力
- recursive evolution 的稳定性（避免 skill 库退化/污染）需要细致控制——[[09-skillsvote]] 专门研究"如何防止 skill 库被污染"
- 实验环境（ALFWorld/WebShop）相对结构化，开放域待验证

### 🔮 揭示的趋势

1. **Skill 库与 policy 共进化**——取代"先学 skill 再用 skill"的两阶段范式
2. **经验蒸馏 = token 效率的关键**——存模式比存轨迹省得多
3. **自动 skill discovery**正在成为 agent RL 的标配模块

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2602.08234
- **HF Papers**: https://huggingface.co/papers/2602.08234
- **GitHub**: https://github.com/aiming-lab/SkillRL （833★）
- **机构**: University of North Carolina at Chapel Hill（aiming-lab）
- **同组工作**: [[01-agent0]] Agent0（同 lab，自演化推理）
