# GrandCode — Agentic RL 拿下 Codeforces Grandmaster

> **arXiv**：2604.02721（2026.04）｜**机构**：Deep Reinforce｜**HF 周榜**：W15 #1，632↑
> **关键词**：Agentic GRPO · Multi-agent RL · Competitive Programming · Test-time RL

---

## 1. 这篇论文为什么重要

**一句话**：GrandCode 是 **首个在 Codeforces 直播赛中持续击败所有人类选手的 AI 系统**——包括 legendary grandmaster 级别的顶尖人类。

这是一个**里程碑事件**：

- AlphaCode (2022) 只到第 54%
- AlphaCode2 (2023) 升到 85th percentile
- OpenAI o3 (2025) 全球第 175 名
- Gemini 3 Deep Think (2026) 第 8 名（**且不在直播条件下**）
- **GrandCode (2026.3)** = 在 Round 1087/1088/1089 三场直播赛**全部第一**，**全部最先完成所有题**

这意味着 AI 在「人类智力最难抵抗的领域之一」也已超过最强人类。学界关注度极高（632 票，半年第二高，仅次于 GLM-5）。

---

## 2. 核心技术贡献

### 2.1 多 Agent 架构（4 个组件分工）

| 组件                          | 模型                    | 职责                                                                                             |
| ----------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------ |
| **π_main**             | Qwen-3.5-397B           | 主求解器：reasoning + code generation                                                            |
| **π_hypothesis**       | Qwen-3.5-27B            | 提出结构性猜想（如$k_{\max}=\max_v(\text{last}(v)-\text{first}(v))$），小样例验证后注入 prompt |
| **π_summary**          | Qwen-3.5-27B            | 压缩超长 reasoning trace（>100K tokens），让后续 RL 可行                                         |
| **Test Case Generator** | Fine-tuned Qwen-3.5-27B | 生成 adversarial / solution-attack / large-size 测试用例                                         |

**设计哲学**：用小模型做辅助、大模型做主求解，**算力 + 训练稳定性双赢**。这是当前 agentic RL 工业落地的主流路线。

### 2.2 Agentic GRPO（最核心的算法创新）

**问题**：agentic RL 中一次完整 rollout 可能超过 1 分钟（要编译、执行、再编译），传统 GRPO 等到 $r_N$ 才更新，**严重 off-policy**。

**解法**：把 trajectory 切成多个阶段 $s_1, r_1, s_2, r_2, \ldots, s_N, r_N$，每个阶段做**两阶段更新**：

**① Immediate Reward**（立即更新）：

$$
A_t^{(i)}=\frac{r_t^{(i)}-\mu_t}{\sigma_t}
$$

**② Delayed Correction**（延迟修正）：当 $r_N$ 可得时，对每个早期阶段做修正：

$$
\delta_t^{(i)} = r_N^{(i)} - r_t^{(i)}
$$

然后用更紧的 clip $\epsilon_2 \leq \epsilon$ 再次更新。

**为什么是 2026 H1 最重要的 agentic RL 算法进展之一**：

- 第一次系统地解决「multi-stage rollout 的延迟奖励 + off-policy 漂移」
- 可与 asynchronous training（pipeline-RL）正交叠加
- W21 DelTA、W20 SDAR 等多个后续工作都引用了这个思路

### 2.3 Hypothesis-Guided Generation

在求解前，π_hypothesis 先提出**结构化猜想**（compact mathematical characterization）：

1. 生成 hypothesis
2. brute-force 在小样例上验证
3. 通过 → 注入 π_main 的 prompt；失败 → 反馈 counter example 修正
4. 还可查 **OEIS**（在线整数序列百科）找数列模式

这是一个把**符号推理 + 神经生成**结合的优雅设计。

### 2.4 Adversarial Test Case Generation

| 策略                        | 做法                                                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Difference-driven** | 让多个 LLM (Claude/GPT/DeepSeek/Kimi) 生成测试用例，跑在不同 candidate solutions 上，**保留能区分输出的用例** |
| **Solution Attack**   | 训练时有 gold solution，让 LLM 对比 gold vs candidate，定位 bug 并生成攻击性边界用例                                |

### 2.5 Reward 三阶段评分

1. **Executability** — 不能编译/执行 = 0 分
2. **Correctness** — 不通过测试 = 0 分
3. **Efficiency** — 相对 brute-force 的加速比

外加**难度自适应长度惩罚**：5 级难度分类器，难题允许更长 thinking budget。

---

## 3. 关键实验结果

### 3.1 Codeforces 直播赛（最有说服力的数字）

| Round          | 日期      | S(joint) | 完成时间 | 排名  |
| -------------- | --------- | -------- | -------- | ----- |
| 1087 (Div.2)   | 2026.3.21 | 8334     | 00:51:11 | 🥇 #1 |
| 1088 (Div.1+2) | 2026.3.28 | 15008    | 01:40:35 | 🥇 #1 |
| 1089 (Div.2)   | 2026.3.29 | 9506     | 00:56:43 | 🥇 #1 |

> 注：Codeforces 禁 AI，所以他们等人类几乎完成后再提交「full version」以避免账号被封。

### 3.2 100 题 benchmark 上 vs 顶尖闭源模型

| 模型                 | 通过率 | Level 5 | 加权分 (0-100)  |
| -------------------- | ------ | ------- | --------------- |
| **GrandCode**  | —     | —      | 🥇 击败所有人类 |
| Gemini 3.1 Pro       | 75%    | 7/20    | 64.3            |
| Claude Opus 4.6      | 73%    | 8/20    | 63.7            |
| GPT-5.4              | 72%    | 7/20    | 63.0            |
| Kimi K2.5            | 65%    | 5/20    | 53.3            |
| Qwen 3.5-397B (base) | 64%    | 4/20    | 52.3            |

### 3.3 训练阶段消融

| 阶段                 | 通过率             | Level 5 | 加权分 |
| -------------------- | ------------------ | ------- | ------ |
| Qwen 3.5-397B base   | 64%                | 4/20    | 52.2   |
| + Continued training | 71%                | 6/20    | 61.0   |
| + SFT                | 73%                | 7/20    | 62.5   |
| + Multi-component RL | (最终结果见直播赛) | —      | —     |

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

- **证伪了「AI 永远做不过人类竞赛选手」的悲观判断**，将话题从「能不能赢」转向「赢多少 / 什么场景下赢」
- **Agentic GRPO 成为新基线**：W20 SDAR、W21 DelTA 等明确对标
- **多组件 RL 编排范式** = 取代「单一 LLM 端到端」成为新标准

### ⚠ 局限

- **不开源**（Deep Reinforce 公司不公开代码/权重）
- 用了大量闭源模型（Claude、Gemini、GPT-5.4、Kimi K2.5）做数据生成 → 复现门槛高
- 实际编程能力是否泛化到工业场景（如 SWE-bench）未验证

### 🔮 论文揭示的趋势

1. **Agentic ≠ 一个大模型自言自语**，而是**多个专长模型 + 显式编排**
2. **数据合成 + 验证循环** 是 SFT 阶段的关键（continued pretraining 用了 Claude/Gemini 生成的扩展数据）
3. **Test-time RL** 已经从研究概念变成实战武器（live contest 用在线 RL loop）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2604.02721
- **HF Papers**: https://huggingface.co/papers/2604.02721
- **GitHub**: 未公开
- **联系**: {xiaoya_li, xiaofei_sun, songqiao_su, chris_shum, jiwei_li}@deep-reinforce.com
