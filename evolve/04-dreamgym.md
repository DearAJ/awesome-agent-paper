# DreamGym — 经验合成驱动的可扩展 Agent 自改进

> **arXiv**：2511.03773（2025.11）｜**机构**：Meta Research（+ 合作高校）
> **HF 月榜**：2025-11 #33，83↑
> **关键词**：Experience Synthesis · Reasoning-based Experience Model · Experience Replay · Adaptive Curriculum · Sim-to-Real

---

## 1. 这篇论文为什么重要

**一句话**：DreamGym 是**首个统一的"经验合成"框架**——它不再靠昂贵的真实环境 rollout，而是把环境动态**蒸馏成一个"基于推理的经验模型"**，在合成经验里训练 agent，在 WebArena 上**超过所有基线 30%+**，并能用纯合成经验给真实 RL 做"热启动"。

它直击 agent RL 落地的四大顽疾：
- **rollout 昂贵**（真实环境交互成本高）
- **任务多样性有限**
- **reward 信号不可靠**
- **基础设施复杂**

这些都阻碍了"可扩展经验数据"的采集——而经验数据正是 agent self-improvement 的燃料。

**DreamGym 的核心洞察**：与其在真实环境里反复试错，不如让一个**会推理的"梦境环境模型"**来生成一致的状态转移和反馈信号——agent 在"梦里"练，成本几乎为零。

---

## 2. 核心方法

### 2.1 三大组件

```
┌────────────────────────────────────────────────────────────────┐
│  ① Reasoning-based Experience Model（基于推理的经验模型）         │
│                                                                  │
│   不做昂贵的真实 rollout，而是把环境动态【蒸馏】进一个经验模型：   │
│     • 通过 step-by-step reasoning 推导【一致的状态转移】          │
│     • 推导【反馈信号 feedback signals】                          │
│   ⟹ 可扩展地采集 agent rollout 用于 RL                          │
└──────────────────────────────┬─────────────────────────────────┘
                               │
┌──────────────────────────────┴─────────────────────────────────┐
│  ② Experience Replay Buffer（经验回放缓冲）                       │
│                                                                  │
│   • 用【离线真实数据】初始化（保证 transition 质量/稳定性）       │
│   • 持续用新的交互【充实】buffer                                  │
│   ⟹ 主动支撑 agent 训练，提升 transition 的稳定性与质量          │
└──────────────────────────────┬─────────────────────────────────┘
                               │
┌──────────────────────────────┴─────────────────────────────────┐
│  ③ Adaptive Task Generation（自适应任务生成）                     │
│                                                                  │
│   自适应生成【挑战当前 policy】的新任务                          │
│   ⟹ 实现更有效的 online curriculum learning（在线课程学习）     │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2 三个关键设计的动机

**① Reasoning-based Experience Model —— "用推理代替试错"**
- 传统 world model 用神经网络预测下一状态，但在复杂 agent 任务上不准
- DreamGym 让经验模型**通过 step-by-step reasoning** 推导状态转移——更一致、更可控
- 这让"合成经验"的质量足够高，能直接拿来训 RL

**② Experience Replay Buffer —— "真实数据打底 + 合成数据扩充"**
- 纯合成会偏离真实分布，纯真实又太贵
- 折中：用**离线真实数据初始化** buffer，保证基础质量；再用合成交互**持续扩充**
- 兼顾稳定性（真实锚点）与规模（合成扩充）

**③ Adaptive Task Generation —— "在线课程"**
- 自适应生成"恰好挑战当前 policy"的任务
- 与 [[01-agent0]] 的 curriculum agent 异曲同工——都在做**自动课程生成**
- 但 DreamGym 把它嵌进经验合成框架，服务于 online RL

---

## 3. 关键实验结果

### 3.1 三类场景的结果

| 场景 | 结果 |
|------|------|
| **non-RL-ready 任务**（如 WebArena，本来难做 RL） | **超过所有基线 30%+** 🔥 |
| **RL-ready 但昂贵的场景** | 仅用**合成交互**就**追平 GRPO / PPO** |
| **Sim-to-Real 迁移** | 纯合成经验训的 policy 迁到真实 RL，**额外显著增益** + 大幅减少真实交互需求 |

### 3.2 三个核心论断

1. **WebArena +30%**——在"本来难以做 RL"的任务上，合成经验让 RL 变得可行
2. **追平真实 RL**——在标准 RL 任务上，纯合成经验能达到 GRPO/PPO 用真实 rollout 的水平，**成本却低得多**
3. **可扩展的 warm-start 策略**——合成经验训好的 policy 作为"热启动"迁到真实 RL，**用更少真实交互拿更高性能**

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **「Experience Synthesis」成为 self-improvement 的关键使能技术**
   - DreamGym（Meta，2025-11）是这条线最早的统一框架之一
   - 与 [[02-evocua]] EvoCUA（合成 CUA 经验）、[[03-agent-world]] Agent-World（合成环境）共同构成"经验/环境合成"浪潮

2. **降低 agent RL 的门槛**
   - rollout 昂贵是 agent RL 普及的最大障碍，DreamGym 用"会推理的经验模型"把成本砸下来
   - "合成经验 warm-start + 少量真实 RL"成为务实的工业方案

3. **把 self-improvement 与 world model 思想结合**
   - reasoning-based experience model 本质是一种"轻量 world model"
   - 呼应了 2026 H1 的 world model 研究热潮

### ⚠ 局限

- 合成经验的保真度受经验模型推理能力限制——复杂环境可能仍有偏差
- 需要离线真实数据初始化 buffer，纯冷启动场景受限
- "推理生成状态转移"在超长程任务上的一致性待验证

### 🔮 揭示的趋势

1. **「在虚拟经验里练，在真实环境里用」**——sim-to-real warm-start 成为标准范式
2. **Reasoning > 神经网络预测**——用 LLM 推理生成经验，比传统 world model 更可控
3. **经验合成 + 自适应课程**——两者结合是可扩展自改进的核心配方（[[01-agent0]] 同源）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2511.03773
- **HF Papers**: https://huggingface.co/papers/2511.03773
- **机构**: Meta Research
- **基准**: WebArena 等；对比 GRPO / PPO
- **框架名**: DreamGym
