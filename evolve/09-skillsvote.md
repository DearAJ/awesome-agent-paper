# SkillsVote — Agent Skill 的全生命周期治理（采集→推荐→进化）

> **arXiv**：2605.18401（2026.05）｜**机构**：Memtensor Research Group（IAAR-Shanghai / 记忆张量）
> **HF 月榜**：2026-05 #30，126↑
> **代码**：https://github.com/MemTensor/skills-vote （277★）｜**站点**：https://skills.vote
> **关键词**：Agent Skills · Lifecycle Governance · Evidence-gated Updates · Agentic Library Search · Frozen Agent

---

## 1. 这篇论文为什么重要

**一句话**：SkillsVote 不研究"如何进化单个 skill"，而是研究**如何治理整个 skill 生态**——从 million 级开源语料里采集、按环境/质量/可验证性画像、执行前检索、执行后**只把验证过的成功发现纳入"证据门控更新"**，让 frozen agent 不更新权重也能变强。

它直面 skill 生态的脏乱差现实：
- 长程 agent 留下大量轨迹，本可成为可复用经验
- 但**原始轨迹噪声重、难治理**
- 开放 skill 生态里充斥**冗余、参差、环境敏感的 artifact**
- **不加区分地更新 skill 库会污染未来的 context**（坏 skill 进库 → 拖累后续所有任务）

**SkillsVote 的核心立场**：skill 进化的关键不是"产生更多 skill"，而是**控制曝光（exposure）、归因（credit）、保存（preservation）**——这是一个**治理（governance）**问题。

---

## 2. 核心方法

### 2.1 全生命周期治理框架

把 Agent Skill 定义为一种**经验 schema**——耦合"可执行脚本 + 非执行的流程指引"。

```
   ┌──────────────────────────────────────────────────────────────┐
   │  【采集 Collection】                                           │
   │   对 million-scale 开源语料做画像（profiling）：               │
   │     • 环境需求（environment requirements）                     │
   │     • 质量（quality）                                          │
   │     • 可验证性（verifiability）                                │
   │   为"可验证的 skill"合成对应任务                               │
   └────────────────────────────┬─────────────────────────────────┘
                                │
   ┌────────────────────────────┴─────────────────────────────────┐
   │  【推荐 Recommendation】（执行前）                              │
   │   Agentic Library Search：                                     │
   │     在结构化 skill 库上做 agentic 检索                         │
   │     → 暴露出"指导性的 skill context"给 agent                  │
   └────────────────────────────┬─────────────────────────────────┘
                                │
   ┌────────────────────────────┴─────────────────────────────────┐
   │  【进化 Evolution】（执行后）—— 核心                           │
   │   ① 把 trajectory 分解为 skill-linked 子任务                   │
   │   ② 归因（attribution）：把成败归因到                          │
   │        skill 使用 / agent 探索 / 环境 / 结果 信号             │
   │   ③ Evidence-gated Update（证据门控更新）：                    │
   │        只有【成功且可复用】的发现才被纳入 skill 库            │
   │        ⟹ 防止坏 skill 污染未来 context                        │
   └──────────────────────────────────────────────────────────────┘
```

### 2.2 三个治理关键

**① 采集：million 级语料 + 三维画像**
- 不是随便收 skill，而是按 **环境需求 / 质量 / 可验证性** 三个维度画像
- 重点筛出"可验证"的 skill——能合成任务自动验证的才有价值

**② 推荐：Agentic Library Search（执行前控制"曝光"）**
- 执行任务前，用 agentic 检索从结构化库里找出相关 skill
- 把"指导性 skill context"喂给 agent——**控制 exposure**（只暴露相关的，不污染 context）

**③ 进化：Evidence-gated Update（执行后控制"归因 + 保存"）**
- 这是 SkillsVote 最关键的设计——**证据门控**
- 先把轨迹拆成 skill-linked 子任务，再把成败**精细归因**到四个来源：skill 使用、agent 探索、环境、结果信号
- 只有"成功 + 可复用"的发现才允许进库——**坏 skill 进不来**
- 直接解决了"indiscriminate update 污染 context"的痛点

---

## 3. 关键实验结果

### 3.1 主结果（offline + online 双模式）

| 模式 | 基准 | 提升 |
|------|------|------|
| **离线进化（offline evolution）** | Terminal-Bench 2.0（GPT-5.2） | **最高 +7.9 pp** |
| **在线进化（online evolution）** | SWE-Bench Pro | **最高 +2.6 pp** |

### 3.2 核心论断

> **"受治理的外部 skill 库，能在不更新模型权重的前提下提升 frozen agent——只要系统控制好 exposure、credit、preservation 三件事。"**

- 这是对"frozen model + 外部 skill"范式的有力背书（与 HF 目录 SkillOpt 同立场）
- 关键不在"产生 skill"，而在**治理 skill**

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **把 skill 研究从"单 skill 进化"升级到"生态治理"**
   - 此前工作（[[05-skillrl]]、[[06-skill1]]、[[07-psn]]）聚焦"怎么进化一个 skill 库"
   - SkillsVote 问的是"million 级 skill 生态怎么治理、防污染"——更工程、更现实
   - 与 [[10-skillnet]] SkillNet（20 万 skill 基础设施）同属"skill 生态规模化"主题

2. **Evidence-gated Update —— 防污染的标准方案**
   - "只采纳验证过的成功发现"是对 [[01-agent0]]/[[05-skillrl]] 等自演化系统稳定性问题的直接回应
   - 精细归因（attribution）让 credit 不再笼统

3. **"frozen agent + 治理良好的外部 skill"的工业价值**
   - 闭源模型时代，不能动权重，只能动外部 skill——SkillsVote 给出治理蓝图
   - 在 Terminal-Bench 2.0 / SWE-Bench Pro 等硬基准上验证，工业可信度高

### ⚠ 局限

- million 级语料画像的算力/工程门槛高
- 归因（attribution）的准确性决定门控质量——归因错了会误杀好 skill 或放进坏 skill
- 治理框架本身的复杂度较高，落地需要完整基建

### 🔮 揭示的趋势

1. **Skill governance（治理）**成为独立子领域——采集/推荐/进化全生命周期
2. **Evidence-gated / validation-gated update**成为防污染标配（与 HF SkillsBench "self-generated skill 平均无收益" 的发现呼应——必须有 validation gate）
3. **"控制 exposure / credit / preservation"**是 frozen-agent 时代 skill 管理的三大支柱

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.18401
- **HF Papers**: https://huggingface.co/papers/2605.18401
- **GitHub**: https://github.com/MemTensor/skills-vote （277★）
- **站点**: https://skills.vote
- **机构**: Memtensor Research Group（记忆张量 / IAAR-Shanghai）
- **基准**: Terminal-Bench 2.0, SWE-Bench Pro
