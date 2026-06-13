# Vision-DeepResearch — 把「多轮 × 多实体 × 多尺度」搜索内化进 MLLM 的多模态深度研究

> **arXiv**：2601.22060（2026.01，方法）＋ 2602.02185（2026.02，配套基准 VDR-Bench）｜**机构**：Osilly 团队
> **HF 月榜**：方法 2026-02 #20，155↑｜基准 2026-02 #26，118↑
> **关键词**：Multimodal DR · Multi-turn/entity/scale Search · Cold-start+RL · VDR-Bench
> **GitHub**：https://github.com/Osilly/Vision-DeepResearch （643★，方法与基准共用）

---

## 1. 这篇论文为什么重要

**一句话**：Vision-DeepResearch 是 **首个把「多轮 × 多实体 × 多尺度」视觉-文本搜索内化进 MLLM 的端到端多模态深度研究范式**，配套的 **VDR-Bench 则系统揭穿了现有多模态搜索基准「答案泄漏 + 过度理想化」两大设计缺陷**——一篇给方法，一篇给评测，构成同一团队的「范式 + 标尺」配对。

为什么这是多模态 deep research 的关键一跳：

- [[videodr]] 是**视频**深度研究的开山基准（HuggingFace 库 04），它确立了「局部视觉锚点 + 开放网络可验证答案」的多模态 DR pipeline；**VDR-Bench 是它在「图像」维度上的同构延伸**——把视频换成静态图像，同样要求「先识别实体 → 再多跳搜索 → 再验证」。
- 之前的多模态搜索（如 reasoning-then-tool-call 系列）有一个**天真假设**：一张全图/实体级图像查询 + 几条文本查询就够了。Vision-DeepResearch 指出：**真实场景充满视觉噪声**，这个假设根本不成立。
- 它与 [[08-opensearch-vl]] 是**同期的两条多模态搜索配方**——一个走 cold-start+RL 内化范式，一个走 fatal-aware GRPO 抗工具失败，可对照阅读。

> **核心张力**：方法论解决「怎么搜得更深更广」，基准解决「怎么测得真实不泄漏」。后者的 **answer leakage（答案泄漏）** 诊断，是 2026 多模态评测设计最值得记住的一课。

---

## 2. 核心方法

### 2.1 方法侧（2601.22060）：从「天真搜索」到「深度研究」

现有 reasoning-then-tool-call 范式 vs Vision-DeepResearch 范式：

| 维度         | 现有多模态搜索（naive setting）          | Vision-DeepResearch                       |
| ------------ | ---------------------------------------- | ----------------------------------------- |
| 图像查询     | 单张 full-level **或** entity-level 图像 | **多实体（multi-entity）**：拆解出多个目标 |
| 文本查询     | 少量 text query                          | 多轮迭代 query                            |
| 尺度         | 单一尺度                                 | **多尺度（multi-scale）**：全图↔局部裁剪   |
| 交互轮次     | 少数几轮                                 | **多轮（multi-turn）**，数十步推理         |
| 搜索引擎交互 | 几次                                     | **数百次** engine interactions             |
| 噪声鲁棒性   | 假设无噪声                               | 在 **heavy noise** 下仍能命中真实引擎      |

三个「多」是范式核心：

$$\text{Search} = \underbrace{\text{multi-turn}}_{\text{推理深度}} \times \underbrace{\text{multi-entity}}_{\text{搜索广度}} \times \underbrace{\text{multi-scale}}_{\text{视觉粒度}}$$

### 2.2 训练：cold-start 监督 + RL

把深度研究能力**内化**进 MLLM 权重，而非靠外部 workflow 编排：

| 阶段              | 作用                                                        |
| ----------------- | ----------------------------------------------------------- |
| Cold-start 监督   | 学会「reasoning-then-tool-call」的基本搜索模式与工具调用先验 |
| RL 训练           | 强化在高噪声下的多轮搜索决策、推理深度与搜索广度             |
| → 端到端 MLLM     | 单模型直接完成数十步推理 + 数百次引擎交互                    |

这条「cold-start SFT → RL」路线与 [[videodr]] 中提到的 GeoVista、以及 [[08-opensearch-vl]] 的「SFT→RL」配方同源，是 2026 多模态 agentic 训练的主流模板。

### 2.3 基准侧（2602.02185）：VDR-Bench 诊断两大缺陷

VDR-Bench 共 **2,000 个 VQA 实例**，针对现有基准的两个病根做针对性设计：

| 缺陷                          | 具体表现                                                                                       | VDR-Bench 的修正                       |
| ----------------------------- | ---------------------------------------------------------------------------------------------- | -------------------------------------- |
| **① 非视觉-搜索为中心**       | 本应需要视觉搜索的答案，被文本问题里的 **cross-textual cues 泄漏**，或可被 MLLM **先验知识**推出 | 设计成必须真正做视觉搜索才能答对        |
| **② 过度理想化的评测场景**    | 图像搜索端只需对全图做 **near-exact matching**；文本搜索端 **过于直接**、挑战性不足              | 提高视觉检索难度，逼近真实噪声场景      |

构造与质控：**多阶段 curation pipeline + 严格专家复审**。

### 2.4 multi-round cropped-search workflow

针对当前 MLLM 视觉检索能力不足，VDR-Bench 配套提出一个**简单的多轮裁剪-搜索工作流**：反复裁剪图像局部区域 → 分别检索 → 聚合证据，被证明能有效提升真实视觉检索场景下的表现。这与方法侧的「multi-scale」思想呼应——本质都是「不要只拿全图去搜」。

---

## 3. 关键实验结果

> 两篇摘要均以**定性声明**为主，未披露具体数字表。精确分数需读 PDF。

| 维度                  | 摘要披露的信息                                                                  |
| --------------------- | ------------------------------------------------------------------------------- |
| **方法性能宣称**      | 大幅超越现有多模态 DR MLLMs                                                       |
| **对比闭源基础模型**  | 超越构建在 **GPT-5 / Gemini-2.5-pro / Claude-4-Sonnet** 上的 workflow            |
| **推理深度**          | 支持 **数十步** reasoning steps                                                  |
| **搜索强度**          | 支持 **数百次** engine interactions                                              |
| **VDR-Bench 规模**    | **2,000** VQA 实例                                                               |
| **cropped-search 增益** | 「effectively improve」——具体百分点摘要未披露，需读 PDF                          |
| **模型尺寸 / 各基准分数** | 摘要未披露，需读 PDF                                                          |

**与 [[videodr]] 框架的对照预期**：VideoDR 实证「弱模型在长搜索链中视觉 anchor 会漂移（Agentic 反而 -9 分）」。Vision-DeepResearch 的多尺度 + cold-start+RL 设计，**正是针对「锚点漂移」做的能力内化**——把「保持初始视觉锚点」从外部 workflow 的运气，变成训练目标。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **「三个多」成为多模态搜索的事实范式**
   - multi-turn × multi-entity × multi-scale 把「单图单查」的天真假设彻底替换
   - 与 [[08-opensearch-vl]] 的工具环境（裁剪/锐化/超分/透视校正）在「多尺度感知」上殊途同归

2. **answer leakage 被确立为基准设计的头号陷阱**
   - VDR-Bench 指出：很多「视觉搜索」基准其实测的是文本线索或先验知识
   - 这与 [[videodr]] 的「Web-only / Video-only 双重消融门」是同一思想——**只有真正依赖该模态才保留样本**
   - 给后续多模态评测立了一条硬规矩：**先做泄漏审计，再谈难度**

3. **VDR = VideoDR 的图像同构**
   - [[videodr]] 开了视频 DR，VDR-Bench 补齐图像 DR
   - 「视觉锚点 → query 生成 → web 检索 → 多跳验证」的 pipeline 在两种模态间统一

### ⚠ 局限

- 两篇摘要**均无具体数字**，「substantially outperform / effectively improve」缺乏可复现验证
- cold-start 与 RL 的**训练数据规模、算力、模型尺寸**摘要未披露
- VDR-Bench 的 2,000 实例**领域/难度分布**未在摘要给出
- multi-round cropped-search 是否会显著增加 token / 延迟成本，摘要未讨论

### 🔮 揭示的趋势

1. **多模态 DR 从「视频」扩展到「图像」全模态**：文本 DR 已卷，视觉 DR 是新蓝海（同 [[videodr]] 判断）
2. **「能力内化」压过「workflow 编排」**：端到端训练的 MLLM 开始超越基于 GPT-5/Gemini 的外部 workflow
3. **评测设计「去泄漏化」**：answer leakage 审计成为多模态基准的标配前置步骤

### 📊 同方向工作

- [[videodr]]（HF 库 04）：视频 DR 开山基准，VDR-Bench 是其图像同构
- [[08-opensearch-vl]]：同期多模态搜索配方，对照其 fatal-aware GRPO 与工具环境设计
- GeoVista（Tencent Hunyuan，2511.15705）：图像缩放 + web 搜索的 agentic 视觉推理，cold-start+RL 同源
- [[09-harness-1]]：其抗 shortcut 的状态外置思想，与 VDR-Bench 抗泄漏的评测设计在「防捷径」上同构

---

## 5. 资源

- **方法 arXiv**: https://arxiv.org/abs/2601.22060 （155↑）
- **基准 arXiv**: https://arxiv.org/abs/2602.02185 （VDR-Bench，118↑）
- **HF Papers**: https://huggingface.co/papers/2601.22060 ／ https://huggingface.co/papers/2602.02185
- **GitHub**: https://github.com/Osilly/Vision-DeepResearch （643★，方法与基准共用）
- **机构**: Osilly 团队（17 位作者）
- **备注**: 方法 v1 提交 2026-01-29，v3 修订 2026-03-23
