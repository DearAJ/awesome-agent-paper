# OpenSeeker — 仅 11.7k 样本 + 单次 SFT，全开源逼平工业级 search agent

> **arXiv**：2603.15594（2026.03）｜**机构**：上海交通大学（SJTU）
> **HF 月榜**：2026-03 月榜 #20，150↑
> **关键词**：Fully Open Data · Web-graph Reverse-engineering · Entity Obfuscation · Denoised Trajectory
> **GitHub**：[rui-ye/OpenSeeker](https://github.com/rui-ye/OpenSeeker)（746★）

---

## 1. 这篇论文为什么重要

**一句话**：OpenSeeker 是 **首个「模型 + 数据」全开源、且达到前沿水平的 search agent**——靠两项数据合成创新，**仅用 11.7k 合成样本 + 单次 SFT** 就在多个基准刷到 SOTA，把「高性能 search agent 只能由工业巨头造」的格局彻底打破。

为什么这是 deep research 的关键进展：

- 高性能 search agent 长期被工业巨头垄断，根因是 **缺乏透明、高质量的训练数据**——数据稀缺从根本上卡住了社区创新。
- OpenSeeker 的核心主张是 **democratization（民主化）**：把完整训练数据集 + 模型权重全部开源，对照 [[04-step-deepresearch]]、[[06-openresearcher]] 同属 **「以数据为中心、全开源」的 DR 路线**。
- 它与 [[06-openresearcher]] 形成有趣对照：OpenSeeker **逆向工程真实 web graph** 来合成可控多跳任务；OpenResearcher 则在 **离线 15M 文档语料**上跑 search-and-browse。二者代表「数据从哪来」的两条不同技术路线。
- 其 **抗捷径**（防止任务被更便宜的识别路径绕过）的关切，与 [[09-harness-1]] 文件组里 FORT-Searcher 的 shortcut-resistant 合成思路相互呼应。

---

## 2. 核心方法

### 2.1 两项核心创新

| 创新 | 机制 | 解决的问题 |
| --- | --- | --- |
| **① Fact-grounded 可控 QA 合成** | 逆向工程 web graph：**topological expansion（拓扑扩展）+ entity obfuscation（实体混淆）** | 生成覆盖度/复杂度**可控**的多跳推理任务 |
| **② Denoised trajectory 合成** | **retrospective summarization（回溯式摘要）** 对轨迹去噪 | 促使 teacher LLM 产出高质量动作 |

### 2.2 创新①：逆向 web graph 合成可控多跳任务

不同于「先有问题再找答案」，OpenSeeker 反过来——**从真实 web graph 倒推问题**：

```mermaid
flowchart LR
    A[采样种子节点] --> B[Graph Expansion<br/>沿出边收集 k 个连接节点<br/>构成依赖子图]
    B --> C[Entity Extraction<br/>蒸馏出精简 Entity Subgraph]
    C --> D[Question Gen + Entity Obfuscation<br/>具体实体→模糊指代]
    D --> E[Dual-Criteria 校验]
```

- **Entity Obfuscation（实体混淆）**：把具体实体替换为模糊指代，**强制多跳推理**而非一步查表。
- **Dual-Criteria 验证**：① **Difficulty**——base 模型闭卷必须答错；② **Solvability**——给定 Entity Subgraph（oracle）时 base 模型必须答对。这保证任务「难但可解」。

### 2.3 创新②：回溯式摘要去噪轨迹

- **Retrospective summarization**：把每条历史 tool response 压缩为 "Summarized Response" 替换原始噪声响应。
- **非对称训练**：teacher 在「干净/摘要后」的上下文里生成轨迹，而 student 在「原始完整响应」上训练——逼 student 学会 **看穿噪声（see through the noise）** 的内在去噪能力。
- **上下文协议**："Summarized History + Raw Recent"（摘要历史 + 原始近期）。

### 2.4 训练设置

| 项 | 值 |
| --- | --- |
| Backbone | Qwen3-30B-A3B-Thinking-2507（30B 总参 / 3B 激活） |
| 训练样本 | **11.7k**（10.3k 英文 + 1.4k 中文） |
| 训练方式 | **仅 SFT，单次训练** |
| 上下文 / 最大 tool call | 256k / 200 |

---

## 3. 关键实验结果

| 基准 | OpenSeeker (30B, SFT) | DeepDive-32B (SFT+RL) | Tongyi DeepResearch (CPT+SFT+RL) |
| --- | --- | --- | --- |
| **BrowseComp** | **29.5%** | 15.3% | 43.4% |
| **BrowseComp-ZH** | **48.4%** | 29.7% | 46.7% |
| **xbench-DeepSearch** | 74.0% | 51.8% | 75.0% |
| **WideSearch（item F1）** | 59.4% | – | – |

关键对比：

- **vs 次优全开源 agent DeepDive**：BrowseComp **29.5% v.s. 15.3%**（近 2×），且 DeepDive 用了 SFT+RL，OpenSeeker 只用 SFT。
- **vs 工业级 Tongyi DeepResearch**：在 **BrowseComp-ZH 上 48.4% > 46.7%**——而 Tongyi 用了 continual pre-training + SFT + RL 的重型流水线，OpenSeeker 仅 11.7k 样本单次 SFT。（注：BrowseComp 与 xbench 上 Tongyi 仍略高，OpenSeeker 的超越特指 BrowseComp-ZH。）

> 数据量对比：同期 OpenResearcher 用 96k 样本得 BrowseComp 26.3 / xbench 65，OpenSeeker 用 11.7k 即得 29.5 / 74.0——样本效率优势明显。

---

## 4. 对领域的影响 / 后续方向

### 学界影响

1. **「全开源（模型+数据）前沿 search agent」的里程碑**
   - 首次把完整训练数据集与权重一并开源，直接回应「数据稀缺垄断创新」的结构性问题。
2. **逆向 web graph 成为可控任务合成的范式**
   - topological expansion + entity obfuscation 让任务的覆盖度/复杂度变成**可调旋钮**，对照 [[06-openresearcher]] 的离线语料路线。
3. **样本效率成为新的竞争指标**
   - 11.7k + 单次 SFT 即 SOTA，说明**数据质量 >> 数据数量**与训练阶段数量。

### 局限

- 摘要/HTML 均未明确点名 teacher 模型具体型号（仅描述 teacher + summarizer 双 LLM）。
- 仅 SFT，未验证叠加 RL 是否能进一步提升（DeepDive/Tongyi 的 RL 上限是否被低估）。
- BrowseComp（英文）与 xbench 上仍略逊于 Tongyi，全面超越尚未达成。

### 揭示的趋势

1. **数据中心化（data-centric）压倒训练阶段堆叠**——单次 SFT 即可逼平多阶段重型流水线。
2. **「反向工程任务源」**——从答案/web graph 倒推可控问题，比正向找答案更可扩展、更可控。
3. **抗捷径意识普及**——entity obfuscation 与 [[09-harness-1]] 组内 FORT-Searcher 的 shortcut-resistant 合成同向。

### 同方向工作

- [[04-step-deepresearch]]：同属数据中心化全开源 DR，原子能力定向合成。
- [[06-openresearcher]]：离线语料 + 轨迹合成，与 OpenSeeker 的逆向 web graph 形成「数据从哪来」的两条路线对照。
- [[09-harness-1]]：FORT-Searcher 的抗捷径任务合成与 entity obfuscation 同向。

---

## 5. 资源

- **arXiv**：https://arxiv.org/abs/2603.15594 ｜ HTML：https://arxiv.org/html/2603.15594
- **HF Papers**：https://huggingface.co/papers/2603.15594（150↑）
- **GitHub**：https://github.com/rui-ye/OpenSeeker（746★，含完整训练数据集 + 模型权重）
- **机构**：上海交通大学（SJTU）
- **Backbone**：Qwen3-30B-A3B-Thinking-2507
