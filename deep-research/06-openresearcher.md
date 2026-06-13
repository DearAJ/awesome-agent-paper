# OpenResearcher — 完全离线、可复现的长程 DR 轨迹合成流水线

> **arXiv**：2603.20278（2026.03）｜**机构**：TIGER-Lab
> **HF 月榜**：2026-03 月榜 #36，100↑
> **关键词**：Offline Corpus · Trajectory Synthesis · search/open/find primitives · Reproducible Pipeline
> **GitHub**：[TIGER-AI-Lab/OpenResearcher](https://github.com/TIGER-AI-Lab/OpenResearcher)（777★）

---

## 1. 这篇论文为什么重要

**一句话**：OpenResearcher 把 **DR 轨迹合成从「依赖专有 web API」彻底解耦为「一次性语料引导 + 完全离线的多轮轨迹合成」**——用 search/open/find 三个浏览器原语在 15M 文档语料上跑搜索-浏览循环，让长程 DR 数据生产变得 **可复现、低成本、可控可分析**。

为什么这是 deep research 的关键进展：

- 现有轨迹采集流水线普遍 **依赖专有 web API**，导致大规模合成 **昂贵、不稳定、难复现**——这是阻碍学界训练 DR agent 的现实瓶颈。
- OpenResearcher 的解法是 **完全离线（entirely offline）**：把「联网搜索」换成「在固定语料上跑全程可观测的搜索循环」，与 [[05-openseeker]] 同属 **全开源数据流水线**，但二者「数据从哪来」路线不同——OpenSeeker **逆向工程真实 web graph** 合成可控任务，OpenResearcher 则 **构建离线语料环境**再蒸馏轨迹。
- 它与 [[10-dci-and-grepseek]] 一脉相承——都主张 **agent 直接与离线语料交互**（DCI / 离线浏览器原语），把检索接口从「黑盒 web API」变成「可控、可复现、可分析」的环境。

---

## 2. 核心方法

### 2.1 解耦：一次性语料引导 vs 多轮轨迹合成

```mermaid
flowchart LR
    A[一次性 corpus bootstrapping<br/>构建 15M 文档离线语料] --> B[多轮 trajectory synthesis<br/>search/open/find 循环]
    B --> C[GPT-OSS-120B teacher<br/>合成 97K+ 轨迹]
    C --> D[SFT 30B-A3B backbone]
```

把「只需做一次的语料引导」与「需反复跑的轨迹合成」**解耦**——语料只构建一次，之后所有轨迹合成都在离线环境内完成，**零 web API 调用**。

### 2.2 三个显式浏览器原语

| 原语 | 语义 |
| --- | --- |
| **search** | 在 15M 文档语料中检索 |
| **open** | 打开/读取某文档 |
| **find** | 在文档内定位关键信息 |

→ 完全模拟真实浏览器的搜索-浏览行为，但全部作用于离线语料。

### 2.3 教师蒸馏与长程尾部

- **Teacher**：**GPT-OSS-120B**，合成 **97K+ 轨迹**。
- **长程尾部（long-horizon tail）**：包含 **100+ 次 tool call** 的高难度长轨迹——这是训练长程 DR 能力的关键数据形态。
- **Student**：**30B-A3B** backbone，纯 SFT。

### 2.4 离线环境带来的「可控分析」红利

因环境离线且 **全程可观测（fully instrumented）**，作者得以做受控分析，得到三类工程洞见：

1. **数据过滤策略**（哪些轨迹该留）。
2. **agent 配置选择**（如何配工具/上下文）。
3. **retrieval success 与 final answer accuracy 的关系**——检索成功是否必然转化为答对。

---

## 3. 关键实验结果

| 基准 | OpenResearcher (30B-A3B, SFT) | 对比 |
| --- | --- | --- |
| **BrowseComp-Plus** | **54.8%** | 比 base 模型 **+34.0pp** |
| **BrowseComp** | 有竞争力（competitive） | 摘要未披露具体分值，需读 PDF |
| **GAIA** | 有竞争力 | 同上 |
| **xbench-DeepSearch** | 有竞争力 | 同上 |

核心数字：**BrowseComp-Plus 54.8%，较 base +34.0pp**——证明纯离线合成的轨迹足以撑起强 DR 能力，且在 BrowseComp / GAIA / xbench 上保持竞争力。

> 注：除 BrowseComp-Plus 的 54.8% / +34.0pp 外，其余基准摘要仅称 "competitive"，未给具体分值。

---

## 4. 对领域的影响 / 后续方向

### 学界影响

1. **把 DR 数据生产从「联网黑盒」变成「离线白盒」**
   - 完全离线 + 全程可观测 = **可复现**，直接解决「专有 API 昂贵/不稳定/难复现」的痛点。
2. **离线环境解锁受控科学分析**
   - 数据过滤、agent 配置、retrieval→accuracy 关系等都能做对照实验——这是联网环境给不了的。
3. **长程尾部（100+ tool call）的显式构造**
   - 为长程 DR 能力提供了可控的数据来源。

### 局限

- 除 BrowseComp-Plus 外的基准只给定性 "competitive"，缺具体数值。
- 15M 文档语料的覆盖面 vs 真实开放 web 的差距，可能限制对最新事实的研究能力。
- 仅 SFT，未探索 RL 叠加。

### 揭示的趋势

1. **离线语料环境（offline corpus）成为 DR 训练/分析基础设施**——与 [[10-dci-and-grepseek]] 的直接语料交互（DCI）同向。
2. **「一次性引导 + 可复现合成」**——把昂贵的环境构建摊销为一次性成本。
3. **可观测性驱动的 DR 工程学**——把 agent 行为变成可测量、可消融的对象。

### 同方向工作

- [[05-openseeker]]：同为全开源数据流水线，但走「逆向真实 web graph」路线，与本文「离线语料」形成对照。
- [[10-dci-and-grepseek]]：直接语料交互（grep/shell），同属「agent 直接操作离线语料」的范式。
- [[04-step-deepresearch]]：数据中心化全开源 DR 的另一代表。

---

## 5. 资源

- **arXiv**：https://arxiv.org/abs/2603.20278
- **HF Papers**：https://huggingface.co/papers/2603.20278（100↑）
- **GitHub**：https://github.com/TIGER-AI-Lab/OpenResearcher（777★，含 pipeline + 轨迹 + checkpoint + 离线搜索环境）
- **机构**：TIGER-Lab
- **Teacher / Student**：GPT-OSS-120B（teacher）→ 30B-A3B（student, SFT）
