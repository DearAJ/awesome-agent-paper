# EvoCUA — 演化式计算机使用 Agent（合成经验自循环）

> **arXiv**：2601.15876（2026.01）｜**机构**：美团（Meituan）
> **HF 月榜**：2026-01 #35，92↑
> **代码**：https://github.com/meituan/EvoCUA （324★）
> **关键词**：Computer-Use Agent (CUA) · Evolutionary Cycle · Verifiable Synthesis · Sandbox Rollouts · Self-Correction

---

## 1. 这篇论文为什么重要

**一句话**：EvoCUA 把 computer-use agent（CUA）的**数据生成与策略优化合并成一个自持的演化循环**，在 OSWorld 上拿到 **56.7%** 的成功率，**刷新开源 SOTA**，并首次超过 UI-TARS-2（53.1%）这种闭源强模型。

它回答了 CUA 领域的核心瓶颈：**长程电脑操作任务的高质量轨迹数据极度稀缺，靠静态数据集模仿学习撞墙了怎么办？**

传统 CUA 训练范式的问题：
- **被动模仿静态数据集（passive imitation）**——无法捕捉长程电脑任务里复杂的**因果动态**（causal dynamics）
- 静态数据规模有限，能力随数据耗尽而饱和
- 失败轨迹被简单丢弃，浪费了最有价值的"反面教材"

**EvoCUA 的方案**：不靠人工数据，而是建立一个 **data generation ⇄ policy optimization 的自持演化循环**——agent 边自己造任务、边解任务、边从成败中学习。

---

## 2. 核心方法

### 2.1 自持演化循环（Self-Sustaining Evolutionary Cycle）

```
   ┌────────────────────────────────────────────────────────┐
   │  ① Verifiable Synthesis Engine（可验证合成引擎）          │
   │     自主生成多样任务 + 配套的【可执行验证器】              │
   │     （executable validators —— 自动判定成败，无需人工）   │
   └───────────────────────────┬────────────────────────────┘
                               │ 任务 + 验证器
                               ▼
   ┌────────────────────────────────────────────────────────┐
   │  ② Scalable Sandbox Infrastructure（可扩展沙盒基建）      │
   │     编排【数万个异步 sandbox rollouts】                   │
   │     大规模采集 trajectory                                 │
   └───────────────────────────┬────────────────────────────┘
                               │ 海量轨迹（成功 + 失败）
                               ▼
   ┌────────────────────────────────────────────────────────┐
   │  ③ Iterative Evolving Learning（迭代演化学习）            │
   │     按【能力边界 capability boundaries】动态调控更新：     │
   │       • 成功轨迹 → 强化成功套路                            │
   │       • 失败轨迹 → error analysis + self-correction       │
   │                    转化为"富监督信号"                      │
   └───────────────────────────┬────────────────────────────┘
                               │ 更强的 policy
                               └──▶ 回到 ① 出更难的题（演化下一轮）
```

### 2.2 三个关键设计

**① Verifiable Synthesis Engine（可验证合成引擎）**
- 自主生成**多样化任务**，每个任务都配一个**可执行验证器**
- 验证器能自动判定 agent 是否完成任务——这是"无人工标注还能学"的前提
- 解决了 CUA 最大痛点：长程任务的 reward 信号难获取

**② 数万异步 Sandbox Rollouts**
- 工程基建层面编排 **tens of thousands of asynchronous sandbox rollouts**
- 让"经验获取"能 scale 到大规模，匹配演化循环的数据吞吐需求

**③ Iterative Evolving Learning（核心算法创新）**
- **按 capability boundary 动态调控 policy 更新**——识别 agent 当前能力的边界
  - 边界内（已掌握）：强化成功 routine
  - 边界外（失败）：不是简单惩罚，而是做 **error analysis + self-correction**，把失败轨迹转成"富监督信号"
- 这是 EvoCUA 把"失败"变"养料"的关键——失败轨迹不再被丢弃

---

## 3. 关键实验结果

### 3.1 OSWorld 主结果（刷新开源 SOTA）

| 模型 | 类型 | OSWorld 成功率 |
|------|------|---------------:|
| **EvoCUA** | 开源 | **56.7%** 🥇 |
| UI-TARS-2 | 闭源 | 53.1% |
| OpenCUA-72B | 开源（前 SOTA） | 45.0% |

**两个突破**：
1. **+11.7pp** 超过前开源 SOTA（OpenCUA-72B）
2. **+3.6pp** 反超闭源强模型 UI-TARS-2——开源 CUA 首次在 OSWorld 上压过同期闭源

### 3.2 跨模型规模的一致性

- "learning from experience 驱动的演化范式"在**不同规模的 foundation model 上都有一致增益**
- 说明 EvoCUA 不是某个特定模型的偶然，而是一条**可扩展的通用路径**

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **把"自演化"从纯文本推理域推进到 GUI / 电脑操作域**
   - Agent0（数学推理）→ EvoCUA（computer-use）——证明演化范式跨模态有效
   - 与同期 [[03-agent-world]] Agent-World（通用 tool 环境）形成"演化训练三角"

2. **「失败轨迹 = 富监督信号」的工程化落地**
   - 区别于只用成功轨迹做 SFT 的做法，error analysis + self-correction 让失败也能学
   - 与 [[09-skillsvote]] SkillsVote 的"evidence-gated update"（只采纳验证过的更新）形成对照——一个用失败、一个严控成功

3. **工业级 CUA 的开源标杆**
   - 美团把数万 sandbox rollout 的工程基建开源，对 CUA 社区是重要基础设施贡献

### ⚠ 局限

- 数万异步 sandbox 的算力/工程门槛很高，小团队难复现
- 合成任务的多样性受 synthesis engine 设计上限约束
- OSWorld 之外的真实生产环境泛化性待验证

### 🔮 揭示的趋势

1. **「Data generation ⇄ Policy optimization 自循环」成为自演化 agent 的标准架构**——见 [[03-agent-world]]、[[01-agent0]] 同构
2. **Verifiable synthesis（可验证合成）是无人工数据训练的前提**——验证器 = 自动 reward
3. **Capability-boundary-aware 更新**——按能力边界精细调控，是演化稳定性的关键

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2601.15876
- **HF Papers**: https://huggingface.co/papers/2601.15876
- **GitHub**: https://github.com/meituan/EvoCUA （324★）
- **机构**: 美团（Meituan）
- **基准**: OSWorld（computer-use agent 标准基准）
