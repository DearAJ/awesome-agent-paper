# DR Tulu — 让 rubric 随 policy 一起进化，首个直训长文深度研究的开源模型

> **arXiv**：2511.19399（2025.11）｜**机构**：RL ReSearch
> **HF 月榜**：2025-11 月榜 #47，63↑
> **关键词**：RLER · Evolving Rubrics · Long-form DR · MCP infrastructure
> **GitHub**：[rlresearch/dr-tulu](https://github.com/rlresearch/dr-tulu)（659★）

---

## 1. 这篇论文为什么重要

**一句话**：DR Tulu 指出绝大多数开源深度研究模型都是用**易验证的短答 QA**经 RLVR 训出来的，这**不能迁移到真实的长文研究任务**；它提出 **RLER（RL with Evolving Rubrics，带演进式评分准则的强化学习）**——让 rubric 与 policy **协同进化**，从而训出**首个直接面向开放式长文深度研究**的开源模型 DR Tulu-8B。

为什么这是深度研究训练的关键进展：

- **奖励信号的根本错配**：短答 QA 有标准答案、可机械验证（RLVR）；但长文研究的产出是**多步、带引用的长报告**，没有唯一正确答案，RLVR 无从下手。
- **RLER 的核心洞见**：rubric 不应是**静态预设**的——它应当**随训练过程演进**，吸收模型**新探索到的信息**，从而提供**有区分度的 on-policy 反馈**。
- DR Tulu-8B 因此成为**第一个直接为开放式长文 DR 而训练**的开源模型，并配套开源 **MCP-based agent 基础设施**。

"长文研究没有可验证 reward"这一痛点，与 [[13-dr-eval-and-error]] 揭示的"长文研究**评估**同样难做 rubric / 难定位错误"是**同一枚硬币的两面**——一边是训练时的奖励信号缺失，一边是评测时的评分准则缺失。RLER 的"演进 rubric"恰是对这一痛点在训练侧的回应。

---

## 2. 核心方法

### 2.1 问题：RLVR 撑不起长文研究

| 范式 | 任务形态 | 奖励来源 | 局限 |
| ---- | -------- | -------- | ---- |
| **RLVR**（主流开源 DR） | 易验证的**短答 QA** | 可机械验证的正确答案 | **不延伸到**真实长文任务 |
| **RLER**（本文） | **开放式长文** DR | **演进式 rubric** 的判别性反馈 | —— |

### 2.2 RLER：Reinforcement Learning with Evolving Rubrics

核心机制——**rubric 与 policy 协同进化**：

$$
\text{rubric}_{t}\;\xleftarrow{\text{co-evolve}}\;\pi_{\theta_t}\quad\Rightarrow\quad \text{discriminative, on-policy feedback}
$$

- **构建并持续维护 rubric**，使其在训练中**与 policy 一起演进**。
- rubric 能**吸收模型新探索到的信息**——即随着 policy 探索出新内容，评分准则随之更新，而非停留在训练初期的静态标准。
- 由此提供**有区分度的（discriminative）、on-policy 的反馈**——避免静态 rubric 对"模型实际产出"评分失真。

> 直觉：静态 rubric 像一把不会更新的尺子，量不准模型新长出的能力；演进 rubric 则随模型一起"长大"，始终对当前 policy 的真实表现敏感。

### 2.3 DR Tulu-8B + MCP 基础设施

- 用 RLER 训出 **DR Tulu-8B**——**首个直接为开放式长文深度研究而训练的开源模型**。
- 同时开源全部 **data / models / code**，含**面向深度研究系统的 MCP-based agent 基础设施**（标准化工具调用 / agent 编排）。

> 摘要未披露 RLER 中 rubric 的具体生成器、演进频率、由谁打分（LLM judge?）等实现细节，需读 PDF。

---

## 3. 关键实验结果

横跨 **science / healthcare / general** 三类领域的 **4 个长文深度研究 benchmark**：

| 对比对象 | DR Tulu-8B 表现 |
| -------- | --------------- |
| 既有**开源** DR 模型 | **大幅超越（substantially outperforms）** |
| **专有（proprietary）** DR 系统 | **匹配或超越（matches or exceeds）** |
| 模型规模 / 每 query 成本 | **显著更小、更便宜（significantly smaller and cheaper）** |

定性结论：以 8B 的小体量，在长文 DR 上**追平甚至反超闭源系统**，且单 query 成本更低——证明"演进 rubric"训练范式的有效性。

> 摘要未披露 4 个 benchmark 的具体名称与各自分数、对比的专有系统是哪些，需读 PDF。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **打破 RLVR 的天花板**：明确指出"短答 QA + RLVR"训不出长文研究能力，并给出 RLER 这一可行替代——把 RL 从"可验证奖励"推广到"演进式判别奖励"。
2. **首个直训长文 DR 的开源模型**：DR Tulu-8B 填补了"开源模型只会短答"的空白，且 8B 即可匹敌闭源。
3. **开源 MCP agent 基础设施**：为社区提供可复用的工具调用 / agent 编排底座，降低长文 DR 的研究门槛。

### ⚠ 局限

- 演进 rubric 可能**漂移或被 policy 利用（reward hacking）**——rubric 跟着 policy 走，是否会自我强化偏差？摘要未讨论。
- rubric 的打分主体（人 / LLM judge）与一致性未披露。
- 4 个 benchmark、对比的专有系统、绝对分数均未在摘要给出。
- "更便宜"缺具体成本数字。

### 🔮 揭示的趋势

1. **从"静态 reward"到"协同进化 reward"**：奖励信号本身成为可学习、可演进的对象，是长文 / 开放式任务 RL 的新方向。
2. **小模型 + 好奖励 ≈ 大模型**：8B 匹敌闭源，延续"训练范式比模型规模更关键"的趋势。
3. **MCP 成为 DR agent 标准底座**：工具调用基础设施开源化、标准化。

### 📊 同方向工作

- [[13-dr-eval-and-error]]：长文 DR 的**评估**侧同样苦于 rubric / 错误定位（TELBench + DRIFT），与 RLER 的训练侧痛点互为镜像。
- [[01-mirothinker-v1]] / [[04-step-deepresearch]]：同期开源 DR agent，但仍偏向可验证 benchmark（GAIA/BrowseComp）；DR Tulu 主攻**长文**这一难验证场景。
- [[02-iterresearch]]：长程研究的范式 / 上下文管理，与 DR Tulu 的奖励范式构成"范式 × 奖励"的互补。

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2511.19399
- **HF Papers**: https://huggingface.co/papers/2511.19399（63↑）
- **机构**: RL ReSearch
- **GitHub**: https://github.com/rlresearch/dr-tulu （659★）
- **开源内容**: 全部 data / models / code，含 MCP-based agent 基础设施；模型 DR Tulu-8B
