# IterResearch — 用 Markovian 状态重构破解长程研究的"上下文窒息"

> **arXiv**：2511.07327（2025.11）｜**机构**：摘要未披露，需读 PDF
> **HF 月榜**：2025-11 月榜 #36，80↑
> **关键词**：Markovian State · Context Suffocation · EAPO · Long-horizon

---

## 1. 这篇论文为什么重要

**一句话**：IterResearch 诊断出主流深度研究 agent 的"**单上下文范式（mono-contextual paradigm）**"会把所有信息堆进一个不断膨胀的 context 窗口，导致 **上下文窒息（context suffocation）+ 噪声污染（noise contamination）**；它把长程研究**重构为 MDP**，靠周期性的"工作区重构 + 演进式报告"在任意探索深度下维持稳定推理能力。

为什么这是长程研究的关键进展：

- 既有 agent 的隐含假设是"上下文越长越好"——但信息无限累积时，**有效信息被噪声淹没**，模型实际推理能力随深度下降。
- IterResearch 的反直觉主张：**不要囤积一切**，而是**周期性地重建工作区**，把已探索的洞见**合成进一份演进中的报告**当作记忆。
- 三重身份：它既是**可训练的 agent**，又是能直接套在 frontier 模型上的**prompting 范式**，还展示了惊人的 **interaction scaling**（扩展到 2048 次交互）。

这与 [[01-mirothinker-v1]] 把交互次数当第三维度的主张**互为印证**——两者都证明"交互能 scale"，但 IterResearch 的解法是**主动管理上下文**而非被动扩窗。它把"演进报告当记忆"的设计也与 [[09-harness-1]] **外化状态**、[[11-general-agentic-memory]] 的 memory 框架同属一脉：**把状态从 policy 的 transcript 里搬出来**。

---

## 2. 核心方法

### 2.1 诊断：mono-contextual paradigm 的两大病灶

| 病灶 | 成因 | 后果 |
| ---- | ---- | ---- |
| **Context suffocation**（上下文窒息） | 所有信息累积进单一膨胀窗口 | 有效推理容量被占满 |
| **Noise contamination**（噪声污染） | 无关 / 过期信息混入 | 干扰当前决策 |

### 2.2 IterResearch 范式：长程研究 = MDP

把长程研究**重构为 Markov Decision Process**，核心是 **strategic workspace reconstruction（策略性工作区重构）**：

- **演进式报告（evolving report）当记忆**：不保留原始全量上下文，而维护一份不断更新的 report 作为压缩记忆。
- **周期性综合洞见（periodically synthesizing insights）**：每隔若干步把探索结果蒸馏进 report，重建一个"干净"的工作区。
- 结果：在**任意探索深度**下都保持**一致的推理容量**（consistent reasoning capacity）——因为每一轮面对的都是被重构过的、低噪声的工作区，而非无限膨胀的历史。

> Markov 性质的关键：当前决策只依赖"重构后的工作区 + 演进报告"，而非完整历史轨迹——这正是它能 scale 到 2048 次交互而不窒息的原因。

### 2.3 EAPO：Efficiency-Aware Policy Optimization

配套的 RL 框架，专为"高效探索"设计：

| 机制 | 作用 |
| ---- | ---- |
| **几何奖励折扣（geometric reward discounting）** | 激励**高效探索**——越晚才拿到的收益被几何级折扣，迫使 agent 早出结果、不无效兜圈 |
| **自适应下采样（adaptive downsampling）** | 支撑**稳定的分布式训练** |

直觉：geometric discounting 让 reward 对"探索效率"敏感，从优化目标层面对抗长程任务的拖沓。

---

## 3. 关键实验结果

| 评测设置 | 结果 | 说明 |
| -------- | ---- | ---- |
| 6 个 benchmark 平均（vs. 开源 agent） | **+14.5pp** | 大幅超越既有开源 agent，并缩小与 frontier 闭源系统的差距 |
| Interaction scaling（扩展到 **2048 次交互**） | **3.5% → 42.5%** | 随交互次数增长性能戏剧性提升，展现"前所未有的交互缩放" |
| 作为 prompting 策略（vs. ReAct，frontier 模型上） | **+19.2pp** | 长程任务上，直接当 prompting 范式即可大幅提升 frontier 模型 |

定性结论：IterResearch 既是高效的**可训练 agent**，又是即插即用的**prompting 范式**——两种身份都成立。

> 摘要未披露具体六个 benchmark 的名称、各自的绝对分数、模型尺寸与 backbone，需读 PDF。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **正面否定"上下文越长越好"**：把"context suffocation"作为可命名、可诊断的失败模式提出，挑战了无脑扩窗的主流做法。
2. **MDP 重构 + 演进报告**：为长程 agent 提供了一个"**主动遗忘 / 周期重构**"的范式模板，区别于一味累积的 ReAct 式轨迹。
3. **prompting + training 双形态**：证明同一范式既能训练进权重，也能当 prompt 直接提升 frontier 模型——降低了落地门槛。

### ⚠ 局限

- 周期性重构会不会**丢失关键早期证据**？摘要未讨论 report 综合时的信息损失风险。
- "几何奖励折扣"的折扣因子如何选取、对不同任务的敏感性未披露。
- 机构、backbone、6 个 benchmark 明细均未在摘要给出。
- 2048 次交互的**绝对成本**（token / 延迟）未量化。

### 🔮 揭示的趋势

1. **从"累积式"到"重构式"上下文管理**：与 observation masking、记忆压缩等同属"如何让长轨迹不被噪声淹没"这条主线。
2. **interaction scaling 成为共识维度**：与 [[01-mirothinker-v1]] 一同把交互次数推上"性能轴"的位置。
3. **范式即 prompt**：好的 agent 范式应当能脱离训练、直接以 prompting 形式迁移到更强模型。

### 📊 同方向工作

- [[01-mirothinker-v1]]：同样主打 interaction scaling（600 tool call），但用扩窗 + RL，而非工作区重构。
- [[09-harness-1]]：把 working memory / 证据 / 验证记录外化到环境侧，与"演进报告当记忆"的状态外化思想同源。
- [[11-general-agentic-memory]]：JIT 编译式记忆（memorizer + researcher），是"重构记忆"的另一种实现。
- [[13-dr-eval-and-error]]：长程轨迹的可靠性 / 错误定位——IterResearch 减少噪声，DRIFT 类工作定位残余错误。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2511.07327
- **HF Papers**: https://huggingface.co/papers/2511.07327（80↑）
- **机构**: 摘要未披露，需读 PDF
- **GitHub**: 摘要未提供仓库链接，需读 PDF
- **开源内容**: 摘要未明确，需读 PDF
