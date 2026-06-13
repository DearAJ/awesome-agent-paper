# DR 评测三件套 — 把"答对没有"升级为"过程是否可信、能否自动化、跨语言够不够"

> **论文 A**：DeepResearchEval｜arXiv 2601.09688（2026.01）｜**机构**：Infinity Lab｜**HF 月榜**：2026-01 #18，127↑｜**GitHub**：[Infinity-AILab/DeepResearchEval](https://github.com/Infinity-AILab/DeepResearchEval)（137★）
> **论文 B**：Where Do Deep-Research Agents Go Wrong?（DRIFT / TELBench）｜arXiv 2606.02060（2026.06）｜**机构**：NJU-LINK Lab｜**HF 月榜**：2026-06 #40，54↑｜**GitHub**：[NJU-LINK/DRIFT](https://github.com/NJU-LINK/DRIFT)（21★）
> **论文 C**：K-BrowseComp｜arXiv 2606.02404（2026.06）｜**机构**：Carnegie Mellon University（CMU）｜**HF 月榜**：2026-06 #38，56↑｜**GitHub**：[prometheus-eval/K-BrowseComp](https://github.com/prometheus-eval/K-BrowseComp)（12★）

---

## 1. 这篇论文为什么重要

**一句话**：深度研究（DR）的评测正从单一的"**final-answer 答对没有**"分裂为**三条新前沿**——(1) **过程级可信度**（DRIFT 在 trajectory 内做 span 级错误定位），(2) **抗饱和 + 全自动化**（DeepResearchEval 用 persona 造题 + 主动事实核查），(3) **多语种 / 真实世界覆盖**（K-BrowseComp 把 BrowseComp 搬到韩语语境）。三篇放在一起，正好补齐了"DR 该怎么被衡量"的三个缺口。

为什么 2026 这个时点 DR 评测本身成了热点：

- **训练侧的痛点先暴露**：[[03-dr-tulu]] 指出长文研究"**没有可验证 reward**"——只能靠演进式 rubric。评测侧是同一枚硬币的另一面：长文报告**没有唯一正确答案**，rubric 难定、错误难定位。DR Tulu 的"rubric 与 policy 协同进化"与 DeepResearchEval 的"per-task 动态导出维度"在解决**同一个长文判分难题**。
- **答对 ≠ 过程对**：[[videodr]] 已实证"**process 指标比 accuracy 更能区分模型**"（Agentic 模式因 Goal Drift 反而退化）。DRIFT 把这一观察工程化——既然终答正确也可能建立在错误证据链上，就必须**审计轨迹本身**。
- **公开榜趋于饱和**：[[agents-last-exam]] 用"经济价值任务"证明主流 harness 通过率仅 2.6%，说明**旧 benchmark 已被刷穿**。DeepResearchEval 的 persona 造题 + K-BrowseComp 的对抗合成，都是**抗饱和**的造题思路。

一句话定位：**DR 评测正在从"判结果"走向"判过程、判覆盖、判自动化"。**

---

## 2. 核心方法

### DeepResearchEval —— persona 造题 + agentic 评测（全自动闭环）

针对现有 benchmark 的三宗罪：**标注成本高**、**评估维度静态**、**缺引用时无法核实事实**。框架分两段全自动流水线：

| 阶段 | 组件 | 机制 |
| --- | --- | --- |
| **造题** | Persona-Driven Pipeline | 锚定**多样化用户画像**，生成**真实、复杂**的研究任务 |
| **造题·过滤** | 两阶段筛选 | ① **Task Qualification**（任务合格性）② **Search Necessity**（检索必要性）——只保留**必须多源证据整合 + 外部检索**才能完成的任务 |
| **评测** | Adaptive Point-wise Quality Evaluation | **逐任务动态导出** task-specific 的评估维度 / 标准 / 权重（而非全局静态 rubric） |
| **评测** | Active Fact-Checking | **自主抽取并核实**报告中的陈述（经 web search），**即使原文没有引用也能验证** |

> 直觉：把"出题—筛题—判分—查事实"全交给 agent 自动跑通；其中 Adaptive Point-wise 正是 [[03-dr-tulu]] "动态 rubric"思想在**评测侧**的镜像。

### Span-Level Error（DRIFT / TELBench）—— claim-centric 轨迹审计

核心问题：**final-answer 评估只告诉你 agent 成没成功，不告诉你轨迹的哪一段让答案不可靠**。DR agent 的轨迹是 search → tool use → 证据检视 → 答案合成的长链，错误可能藏在任意一段。

数据与基准构建：

| 环节 | 规模 / 做法 |
| --- | --- |
| 真实轨迹采集 | **2,790 条**（**2 个 agent 框架 × 3 个 backbone × 3 个 benchmark**） |
| 日志 → 语义 span | 把原始 log 切成 semantic spans |
| 标注 | **LLM 辅助专家复核**，标出 harmful error spans |
| **TELBench** | **1,000 instance** 基准——要在 ①正常探索 ②失败检索 ③试探性假设 ④无害噪声 之间，**精确识别出错误 span** |

**DRIFT** = claim-centric auditing（以"声明"为中心的审计）：

$$
\text{claim}_i \;\xrightarrow{\text{check support}}\; \text{trajectory evidence} \;\Rightarrow\; \text{mark span if unsupported / conflicting}
$$

- **追踪** agent 提出的每一条 claim；
- **核对**该 claim 在轨迹证据中是否被支撑；
- 在出现**无支撑 / 相互冲突**且**影响了答案路径（answer path）**的地方**标记 span**。

> 这是把 [[videodr]] 的 Goal Drift 观察落成可操作工具：不再只看终答，而是定位"漂移**从哪一段开始**"。与 [[01-mirothinker-v1]] H1 的 global verification（审计整条 trace 证据链一致性）属同构思路，区别在 DRIFT 是**外部第三方审计基准**而非 agent 内置步骤。

### K-BrowseComp —— 韩语网页浏览 agent 基准（抗饱和 + 多语种）

动机：前沿模型评测正从基础能力（指令遵循、推理）转向**组合式、agentic 能力**，但**韩语 agentic 基准极度稀缺**。共 **400 题**，分两个互补 split：

| Split | 规模 | 构建方式 | 定位 |
| --- | --- | --- | --- |
| **K-BrowseComp-Verified** | **300 题** | **母语者手工构建并校验** | 主评测集 |
| **合成诊断 split** | **100 题** | **hard few-shot 范例 + 失败模式定向生成**（利用"解题 vs 造题"的不对称性）；再**对抗式过滤** | 单独报告的定向压力测试 |

> 关键设计：BrowseComp 风格的题"造比解容易"，因此用强模型当前的**失败模式**反向定向造题，再对抗过滤，得到一个连最强模型都打不动的诊断集。

---

## 3. 关键实验结果

### DeepResearchEval

> 摘要为方法框架描述，**未披露**具体打分 / 一致性数字，需读 PDF。

### DRIFT / TELBench

| 指标 | 结果 |
| --- | --- |
| 真实轨迹 | 2,790 条 |
| TELBench 规模 | 1,000 instances |
| **DRIFT 增益** | span 级错误定位 + **first-error accuracy 最高提升 30 个百分点** |

### K-BrowseComp（300 题 Verified 子集）

| 模型类别 | 代表模型 | 准确率 |
| --- | --- | --- |
| 前沿 LLM | GPT-5.5 / DeepSeek-V4-Pro / GLM-5.1 | **30.00 – 45.67%**（相比 BrowseComp **大幅下滑**） |
| 韩国 PAF 项目 LLM | Korea Proprietary AI Foundation 模型 | **0.00 – 10.33%** |
| 合成对抗 split | 最强模型 | **仅 26.00%** |

**三篇的共同信号**：公开 DR 基准一旦"换语境（韩语）/ 换审计粒度（span）/ 换造题方式（persona+对抗）"，模型表现**立刻塌方或被看穿**——印证 [[agents-last-exam]] "经济价值任务远未饱和"的判断。

---

## 4. 对领域的影响 / 后续方向

### 学界影响

1. **评测维度从"结果"扩到"过程 + 覆盖 + 自动化"**
   - DRIFT 把 [[videodr]] "process > accuracy" 的论断变成**可复现的 span 级基准**（TELBench）；
   - DeepResearchEval 把"造题—判分—查证"做成**无人值守闭环**；
   - K-BrowseComp 把评测推向**非英语真实语境**。
2. **训练 rubric 与评测 rubric 是同一个问题**
   - [[03-dr-tulu]] 的 Evolving Rubrics（训练侧）与 DeepResearchEval 的 Adaptive Point-wise（评测侧）共享"**长文没有标准答案、判分维度必须随任务/策略动态生成**"这一根因。
3. **"声明—证据"审计成为通用范式**
   - DRIFT 的 claim-centric auditing 与 [[01-mirothinker-v1]] H1 的 local/global verification、[[aris]] 的三阶段证据-声明审计同向——区别在 DRIFT 是**事后第三方基准**，后两者是 agent **内置 / 跨模型**机制。

### 局限

- **LLM-judge 的循环依赖**：DeepResearchEval 的 Active Fact-Checking 和 DRIFT 的 LLM 辅助标注，本质都是**用 LLM（+ web search）评判 LLM 生成的报告**。当被评 agent 与裁判 agent 同源（或共享同一检索栈、同一偏见）时，存在**自我确认 / 漏判**风险——"裁判"可能复现"选手"的错误证据链。这与 [[03-dr-tulu]] 演进 rubric 的"裁判由谁监督"是同一隐忧。
- **DeepResearchEval 摘要无量化结果**，框架有效性缺乏可复现数字（需读 PDF）。
- **DRIFT 的 30pp 增益**是相对基线的提升，绝对 first-error accuracy 上限、跨更多框架/语言的泛化未在摘要披露。
- **K-BrowseComp 合成 split** 依赖"失败模式定向生成"，可能**过拟合当前强模型的弱点**，对下一代模型的区分度待验证。

### 揭示的趋势

1. **process-level 可信度成为一等公民**——只报终答准确率的 benchmark 将被认为"信息不足"。
2. **造题自动化 + 抗饱和**——persona 驱动、失败模式定向、对抗过滤，取代昂贵的人工标注。
3. **多语种 / 真实职业语境**——与 [[agents-last-exam]] 的 O*NET 职业任务一道，把 DR 评测拉出"英语 + 知识竞赛题"的舒适区。

### 同方向工作

- [[03-dr-tulu]]：训练侧的演进 rubric，与本三件套的评测 rubric 互为镜像。
- [[videodr]]：最早实证 process 指标 > accuracy，DRIFT 是其工程化落地。
- [[01-mirothinker-v1]]：把 verification 内置进推理（local/global），DRIFT 则是外部 span 审计基准。
- [[agents-last-exam]]：经济价值任务的抗饱和 benchmark，与 K-BrowseComp 的多语种抗饱和同向。
- [[aris]]：跨 agent 的证据-声明审计，与 DRIFT 的 claim-centric 思路同构。

---

## 5. 资源

- **DeepResearchEval**：arXiv https://arxiv.org/abs/2601.09688 ｜ HF https://huggingface.co/papers/2601.09688（127↑）｜ GitHub https://github.com/Infinity-AILab/DeepResearchEval （137★）｜ 机构 Infinity Lab
- **DRIFT / TELBench**：arXiv https://arxiv.org/abs/2606.02060 ｜ HF https://huggingface.co/papers/2606.02060（54↑）｜ GitHub https://github.com/NJU-LINK/DRIFT （21★）｜ 机构 NJU-LINK Lab
- **K-BrowseComp**：arXiv https://arxiv.org/abs/2606.02404 ｜ HF https://huggingface.co/papers/2606.02404（56↑）｜ GitHub https://github.com/prometheus-eval/K-BrowseComp （12★）｜ 机构 Carnegie Mellon University
