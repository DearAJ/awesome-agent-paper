# Deep Research 专题 — HuggingFace 月榜精选（2025-11 ~ 2026-06）

> **专题定位**：从 HuggingFace Papers **月榜每月 top50** 中筛出 **Deep Research / Search Agent / Web Agent / Agentic Search** 主线论文，做深度阅读笔记。
> **数据范围**：2025-11、2025-12、2026-01、2026-02、2026-03、2026-04、2026-05、2026-06（共 8 个月）
> **与 `huggingface/` 周榜库的区别**：那里是每周 top10、覆盖全 agent 主题；这里是每月 **top50**、只聚焦 DR 一条线，因此挖得更深、更专。已被 `huggingface/` 收录的论文（VideoDR、Youtu-Agent、MiroThinker-1.7&H1）只交叉引用、不重复精读。
> **目录定位**：`Q:/awesome-agent-paper/deep-research/`

---

## 📚 13 篇精读笔记（按主题分类）

> **Scaling 新轴：		01-02 + 12**
>
> **训练范式 & 数据合成：	03-06 + 09**
>
> **多模态 DR：			07-08**
>
> **检索接口：			10**
>
> **记忆即研究：			11**
>
> **评测：				13**

---

### 🚀 Scaling 的新维度（交互 / 宽度 / 委派）

> 继 model size、context length 之后，2025 H2–2026 H1 集中提出三条互补的扩展轴——交互深度、并行宽度、委派结构。

| #            | 论文                                                       | 月/票       | 机构              | 核心贡献                                                                                          |
| ------------ | ---------------------------------------------------------- | ----------- | ----------------- | ------------------------------------------------------------------------------------------------- |
| **01** | **[MiroThinker v1.0](01-mirothinker-v1.md)**         | 11 / 196↑  | MiroMind          | **交互缩放（interaction scaling）= 第三性能维度**；256K ctx 单任务 600 tool call，72B 逼近 GPT-5-high |
| **02** | **[IterResearch](02-iterresearch.md)**               | 11 / 80↑   | —                | **Markovian 工作区重构**对抗 context suffocation；EAPO；交互扩到 2048 次（3.5%→42.5%）          |
| **12** | **[SearchSwarm & WideSeek-R1](12-searchswarm-and-wideseek.md)** | 06/02 / 49↑·100↑ | SearchSwarm·RLinf | **委派智能（SFT）vs 宽度扩展（MARL）**——横向多 subagent 的两条路线，4B 匹敌 671B            |

### 🏗 开源 DR 系统 & 训练范式

> "把开源 DR 从昂贵工业特权变成可复现、低成本的公共能力"——数据合成成为真正胜负手。

| #            | 论文                                             | 月/票       | 机构              | 核心贡献                                                                                                |
| ------------ | ------------------------------------------------ | ----------- | ----------------- | ------------------------------------------------------------------------------------------------------- |
| **03** | **[DR Tulu](03-dr-tulu.md)**                  | 11 / 63↑   | RL ReSearch       | **RLER（演化 rubric）**——首个直接为 long-form DR 训练的开源模型，配 MCP 基础设施                  |
| **04** | **[Step-DeepResearch](04-step-deepresearch.md)** | 12 / 88↑   | StepFun           | **Atomic capability 数据合成** + agentic mid-training→SFT→RL；32B 拿 61.4% Research Rubrics；ADR-Bench |
| **05** | **[OpenSeeker](05-openseeker.md)**            | 03 / 150↑  | 上交              | **逆向 web graph** 合成多跳 QA；仅 11.7k 样本单次 SFT 即 SOTA，BrowseComp-ZH 超 Tongyi DeepResearch  |
| **06** | **[OpenResearcher](06-openresearcher.md)**    | 03 / 100↑  | TIGER-Lab         | **离线 1500 万语料 + search/open/find 原语**；97K 轨迹（100+ tool call 长尾），+34.0pp on BrowseComp-Plus |
| **09** | **[Harness-1 + FORT + Masking](09-harness-1.md)** | 06 / 52↑   | Chroma 等         | **状态外化（state externalization）+ 训练踩坑**——把 bookkeeping 交给环境；反 shortcut；context 管理 regime-dependent |

### 🖼 多模态 Deep Research

> 文本 DR 已趋成熟，2026 H1 明显向图像/视频扩张；共同痛点是真实视觉噪声下的检索与答案泄露。

| #            | 论文                                                       | 月/票       | 机构        | 核心贡献                                                                                              |
| ------------ | ---------------------------------------------------------- | ----------- | ----------- | ----------------------------------------------------------------------------------------------------- |
| **07** | **[Vision-DeepResearch (+VDR-Bench)](07-vision-deepresearch.md)** | 02 / 155↑·118↑ | —          | **multi-turn × multi-entity × multi-scale** 多模态搜索；配套基准专门构造"不可被文本/先验泄露"样本 |
| **08** | **[OpenSearch-VL](08-opensearch-vl.md)**                | 05 / 102↑  | 腾讯混元    | **fatal-aware GRPO**（mask 失败后 token + 单侧 advantage clamp）；统一 OCR/crop/super-res 工具环境  |

### 🔌 检索接口本身成为研究对象

| #            | 论文                                             | 月/票       | 机构              | 核心贡献                                                                                          |
| ------------ | ------------------------------------------------ | ----------- | ----------------- | ------------------------------------------------------------------------------------------------- |
| **10** | **[DCI & GrepSeek](10-dci-and-grepseek.md)** | 05·06 / 123↑·106↑ | TIGER-Lab·UMass | **Direct Corpus Interaction**——抛弃 embedding 检索器，让 agent 直接 grep 原始语料；"检索质量 = 接口分辨率" |

### 🧠 记忆即研究

| #            | 论文                                                       | 月/票       | 机构  | 核心贡献                                                                                |
| ------------ | ---------------------------------------------------------- | ----------- | ----- | --------------------------------------------------------------------------------------- |
| **11** | **[General Agentic Memory](11-general-agentic-memory.md)** | 11 / 171↑  | BAAI  | **JIT 编译式记忆**——Memorizer + Researcher 双设计，把"记忆检索"重构为一个 deep research 任务 |

### 📊 DR 评测的三条新前线

> 评测从"最终答案对错"走向过程可信度 + 抗饱和 + 多语言真实环境。

| #            | 论文                                             | 月/票       | 机构                | 核心贡献                                                                                          |
| ------------ | ------------------------------------------------ | ----------- | ------------------- | ------------------------------------------------------------------------------------------------- |
| **13** | **[DR 评测三件套](13-dr-eval-and-error.md)** | 01·06 / 127↑·54↑·56↑ | Infinity·NJU·CMU | **DeepResearchEval**（persona+主动核查）·**DRIFT/TELBench**（span 级错误定位 +30pp）·**K-BrowseComp**（韩语，韩语 LLM 仅 0-10%） |

---

## 目录结构

```
deep-research/
├── README.md                          # 本文件 — 总索引（按主题组织）
├── 00-summary-2025.11-2026.06.md      # 8 个月 top50 命中 DR 论文汇总（按月）+ 研究现状与趋势
│
│ ── Scaling 新轴 ──
├── 01-mirothinker-v1.md               # 交互缩放第三维度
├── 02-iterresearch.md                 # Markovian 工作区重构
├── 12-searchswarm-and-wideseek.md     # 委派智能 vs 宽度扩展
│
│ ── 开源 DR 系统 & 训练范式 ──
├── 03-dr-tulu.md                      # RLER 演化 rubric
├── 04-step-deepresearch.md            # Atomic capability 数据合成
├── 05-openseeker.md                   # 逆向 web graph，全开源数据
├── 06-openresearcher.md               # 离线语料 + 轨迹合成
├── 09-harness-1.md                    # 状态外化 + 训练踩坑（含 FORT、Masking）
│
│ ── 多模态 DR ──
├── 07-vision-deepresearch.md          # 多模态 DR + 配套基准
├── 08-opensearch-vl.md                # fatal-aware GRPO 多模态搜索
│
│ ── 检索接口 ──
├── 10-dci-and-grepseek.md             # 直接语料交互（grep/shell）
│
│ ── 记忆 ──
├── 11-general-agentic-memory.md       # JIT 记忆 = deep research
│
│ ── 评测 ──
└── 13-dr-eval-and-error.md            # DeepResearchEval + Span-Error + K-BrowseComp
```

---

## 阅读顺序建议

### 🎯 最短路径（4 篇抓住 2026 H1 DR 主线）

1. **[00-summary 第五节](00-summary-2025.11-2026.06.md)** — 先读"研究现状与趋势"建立全局
2. **[MiroThinker v1.0](01-mirothinker-v1.md)** — 看 scaling 新维度（交互深度）
3. **[OpenSeeker](05-openseeker.md)** — 看开源如何用数据合成追平闭源
4. **[Harness-1 + FORT + Masking](09-harness-1.md)** — 看长程 agent 的状态管理与训练踩坑

### 📚 按主题串读

**Scaling 三条轴**（推荐串读）：

- [MiroThinker v1.0](01-mirothinker-v1.md)：交互深度
- [IterResearch](02-iterresearch.md)：交互深度 + 工作区重构（对抗 context 窒息）
- [SearchSwarm & WideSeek-R1](12-searchswarm-and-wideseek.md)：宽度（MARL）vs 委派（SFT）

**开源 DR 数据合成对比**：[Step-DeepResearch](04-step-deepresearch.md)（atomic capability）→ [OpenSeeker](05-openseeker.md)（逆向 web graph）→ [OpenResearcher](06-openresearcher.md)（离线语料）→ [FORT（在 09）](09-harness-1.md)（反 shortcut）

**多模态 DR 双件套**：[Vision-DeepResearch](07-vision-deepresearch.md)（范式 + 基准）↔ [OpenSearch-VL](08-opensearch-vl.md)（RL 算法）

**检索范式之争**：[DCI & GrepSeek](10-dci-and-grepseek.md)（抛弃检索器）↔ [OpenSeeker](05-openseeker.md)/[OpenSearch-VL](08-opensearch-vl.md)（仍基于检索器）

**评测三前线**：[DR 评测三件套](13-dr-eval-and-error.md)——过程级（DRIFT）+ 抗饱和（DeepResearchEval）+ 多语言（K-BrowseComp）

---

## 2025-11 → 2026-06 关键趋势速览

> 完整论述见 [00-summary 第五节](00-summary-2025.11-2026.06.md)。

1. **Scaling 三新轴**：交互深度（MiroThinker/IterResearch）、并行宽度（WideSeek-R1）、委派结构（SearchSwarm）三者并存、按任务选型。
2. **数据合成成胜负手且开始"反 shortcut"**：FORT 证明"结构复杂 ≠ 真难"，难度必须用 trajectory signature 实测。
3. **State Externalization**：Harness-1 把 bookkeeping 交给环境，让 RL 只学语义决策——held-out transfer 增益尤强。
4. **Context 管理是 regime-dependent**：Masking 发现 mask 增益呈非对称倒 U 形，没有银弹。
5. **检索接口成研究对象**：DCI 让 agent 直接 grep 原始语料，"检索质量 = 接口分辨率"。
6. **多模态 DR 是新蓝海**：VideoDR → Vision-DR → OpenSearch-VL，痛点是真实视觉噪声 + 答案泄露。
7. **评测走向过程可信度**：DRIFT span 级错误定位、DeepResearchEval 主动核查、K-BrowseComp 多语言——"最终答案对"已不够。

**仍待解决**：① 长程一致性的根因消除；② 多模态视觉感知瓶颈；③ long-form 评测的 LLM-judge 循环依赖；④ DCI 的 lexical 局限；⑤ 真实经济价值的可靠落地。

---

## 报告结构说明

每篇精读笔记统一 5 章节（与 `huggingface/` 库一致）：

1. **这篇论文为什么重要** — 一句话核心 + 学界影响背景
2. **核心方法** — 关键技术贡献，配表/公式/流程图
3. **关键实验结果** — 重要数字、ablation（摘要未披露处明确标注"需读 PDF"）
4. **对领域的影响 / 后续方向** — 学界冲击、局限、趋势、同方向工作
5. **资源** — arXiv / HF Papers / GitHub / 机构

> **数据来源说明**：本专题论文元数据来自 HuggingFace Papers 月榜（`huggingface.co/papers/month/YYYY-MM`），笔记内容基于 arXiv 摘要 + 部分全文撰写。涉及的模型/数据规模数字均以论文摘要为准，摘要未披露的细节统一标注待查 PDF。
