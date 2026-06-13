# OpenSearch-VL — 用 fatal-aware GRPO 训练前沿多模态搜索 agent 的全开源配方

> **arXiv**：2605.05185（2026.05）｜**机构**：Tencent Hunyuan
> **HF 月榜**：2026-05 #44，102↑
> **关键词**：Multimodal Search Agent · Agentic RL · Source-anchor Grounding · Fatal-aware GRPO
> **GitHub**：https://github.com/shawn0728/OpenSearch-VL （215★）

---

## 1. 这篇论文为什么重要

**一句话**：OpenSearch-VL 是 **首个把「数据合成 + 工具环境 + RL 算法」三件套全部开源的前沿多模态深度搜索配方**——其中 **fatal-aware GRPO** 专门解决多轮搜索中的**级联工具失败**问题，**source-anchor 视觉锚定**则从数据侧根除「捷径 + 一步检索坍缩」。

为什么这对多模态搜索社区是关键补全：

- 顶级多模态搜索 agent 长期**难以复现**——缺开源高质量数据、缺透明轨迹合成、缺训练配方细节。OpenSearch-VL 把这三块全部公开。
- 它与 [[07-vision-deepresearch]] 是**同期两条多模态 DR 路线**：Vision-DeepResearch 走「cold-start 监督 + RL 内化多轮/多实体/多尺度搜索」；OpenSearch-VL 走「抗失败的 fatal-aware GRPO + 抗捷径的数据合成」。**对照点在于 RL 算法设计**——前者重「能力内化」，后者重「失败鲁棒」。
- 其**反捷径数据设计**与 [[09-harness-1]] 仓库里 FORT 的「shortcut-resistant 训练数据合成」是同一思想脉络——都在防止「搜索过程被一条更便宜的识别路径绕过」。

> **一句话定位**：如果说 [[07-vision-deepresearch]] 回答「多模态搜索范式该长什么样」，OpenSearch-VL 回答「怎么用全开源 RL 把它稳定训出来」。

---

## 2. 核心方法

OpenSearch-VL 是一套完整配方，三个支柱：**数据 → 工具环境 → 算法**。

### 2.1 数据流水线：从 Wikipedia 到抗捷径轨迹

三步构造高质量训练数据，**联合降低 shortcuts 与 one-step retrieval collapse**：

| 步骤                         | 作用                                                                 |
| ---------------------------- | -------------------------------------------------------------------- |
| Wikipedia path sampling      | 沿 Wikipedia 实体路径采样，生成多跳结构                               |
| Fuzzy entity rewriting       | 对实体做**模糊改写**，避免问题中直接暴露答案实体（防先验/文本捷径）    |
| Source-anchor visual grounding | 把证据**锚定到图像来源**，强制模型真正做视觉检索，**防一步检索坍缩** |

产出两份数据集：

| 数据集            | 用途 | 规模     |
| ----------------- | ---- | -------- |
| **SearchVL-SFT-36k** | SFT  | 36k 条   |
| **SearchVL-RL-8k**   | RL   | 8k 条    |

### 2.2 工具环境：感知 + 检索统一

设计了一个统一工具环境，把「主动感知」和「外部知识获取」结合：

| 类别       | 工具                                                  |
| ---------- | ----------------------------------------------------- |
| 外部检索   | text search、image search                             |
| 视觉感知   | OCR、cropping、sharpening、super-resolution、perspective correction |

这套工具集与 [[07-vision-deepresearch]] 的「multi-scale」裁剪思想、以及 GeoVista 的 image-zoom-in 工具同源——核心都是「让模型能放大、增强、矫正局部，再去搜」。

### 2.3 算法：multi-turn fatal-aware GRPO

多轮搜索中，**一次工具失败会级联污染后续所有 token**。fatal-aware GRPO 的处理：

| 机制                            | 含义                                                                 |
| ------------------------------- | -------------------------------------------------------------------- |
| Mask post-failure tokens        | 屏蔽**失败之后**产生的 token（它们被污染，不应回传梯度）              |
| Preserve pre-failure reasoning  | 保留**失败之前**的有用推理                                            |
| One-sided advantage clamping    | 用**单侧优势钳制**实现上述「只保留失败前」的不对称处理                |

直觉公式（概念示意）：

$$\tilde{A}_t = \begin{cases} A_t, & t < t_{\text{fail}} \ (\text{保留失败前推理}) \\ \text{clamp}_{\text{one-sided}}(A_t), & t \ge t_{\text{fail}} \ (\text{屏蔽失败后污染}) \end{cases}$$

**概念同构**：这与 agentic RL 中 OPSD 式的「门控/掩码」监督（见 [[09-harness-1]]、以及 HuggingFace 库里 SDAR 的 OPSD 思路）一脉相承——都在做「区分轨迹中可信段与不可信段，只对可信段学习」。区别在于 fatal-aware GRPO 专门针对**工具调用失败**这一触发点，且用 one-sided clamping 而非硬截断。

---

## 3. 关键实验结果

| 维度                  | 摘要披露的数字                                              |
| --------------------- | ----------------------------------------------------------- |
| **平均提升**          | 跨 **7 个 benchmark** 平均提升 **10+ 个点**                  |
| **对比闭源商业模型**  | 在**若干任务**上达到**与专有商业模型相当**的水平            |
| **SFT 数据**          | SearchVL-SFT-36k（36k 条）                                  |
| **RL 数据**           | SearchVL-RL-8k（8k 条）                                     |
| **各 benchmark 具体分数** | 摘要未披露，需读 PDF                                    |
| **基座模型尺寸**      | 摘要未披露，需读 PDF                                        |

> 摘要给出的是聚合指标（7 基准平均 +10 点）。逐基准分数、基座规模、与具体商业模型（GPT-5 / Gemini 等）的点对点对比，摘要未列出，需读 PDF。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **全开源配方降低多模态搜索 agent 复现门槛**
   - 数据（36k SFT + 8k RL）、代码、模型全开放
   - 与 [[07-vision-deepresearch]] 共同把多模态 DR 从「闭源 workflow」推向「可复现训练」

2. **fatal-aware GRPO：把「工具失败鲁棒性」做进 RL 目标**
   - 多轮 agentic RL 的核心痛点之一就是级联失败污染信用分配
   - one-sided advantage clamping 提供了一个轻量、可迁移的解法
   - 与 [[09-harness-1]] 的「把可恢复 bookkeeping 移出 policy」是互补思路——一个净化梯度，一个净化状态

3. **反捷径数据设计成为共识**
   - fuzzy entity rewriting + source-anchor grounding 与 FORT（[[09-harness-1]] 仓库内）的 shortcut-resistant 合成、VDR-Bench（[[07-vision-deepresearch]]）的抗 answer-leakage 评测，三者共同确立：**「让搜索过程无法被便宜路径绕过」是数据/评测设计的第一原则**

### ⚠ 局限

- 逐基准分数与基座模型规模摘要未披露，「+10 点」「comparable to commercial」缺乏可复现细节
- fatal-aware GRPO 相对标准 GRPO 的**消融贡献**摘要未单列
- 「在若干任务上 comparable」——具体是哪几个任务、与哪个商业模型对比，需读 PDF
- Wikipedia path sampling 的覆盖偏差（偏向 Wiki 可达实体）未讨论

### 🔮 揭示的趋势

1. **多模态搜索 RL 从「能跑」转向「抗失败」**：fatal-aware 一类机制会成为多轮 agentic RL 标配
2. **「感知工具 + 检索工具」统一环境**：OCR/裁剪/超分/透视校正与 search 并列，成为多模态 agent 的标准工具箱
3. **数据合成的核心 KPI 是「捷径率」而非「难度」**：与 FORT、VDR-Bench 同步收敛到这一判断

### 📊 同方向工作

- [[07-vision-deepresearch]]：同期多模态 DR 范式 + VDR-Bench，对照 cold-start+RL vs fatal-aware GRPO
- [[09-harness-1]] / FORT：反捷径数据合成与状态外置，与本文数据设计互补
- GeoVista（Tencent Hunyuan，2511.15705）：同机构，image-zoom-in + web search 的 cold-start+RL，工具思想同源
- [[videodr]]（HF 库 04）：多模态 DR 评测起点，本文工具环境延续其「视觉锚点 + web 检索」pipeline

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.05185
- **HF Papers**: https://huggingface.co/papers/2605.05185 （102↑）
- **GitHub**: https://github.com/shawn0728/OpenSearch-VL （215★）
- **机构**: Tencent Hunyuan
- **开源**: SearchVL-SFT-36k / SearchVL-RL-8k 数据 + 代码 + 模型
