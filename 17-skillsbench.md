# SkillsBench — Agent Skill 的事实评测标准

> **arXiv**：2602.12670（2026.02，HF 周榜 W08 / Feb 15-21）｜**机构**：BenchFlow（提交者 xdotli）
> **HF 票数**：1,330↑（半年榜第二高，仅次于 GLM-5）
> **关键词**：Agent Skills · Curated vs Self-generated · Deterministic Verifier · 11×86 Task Matrix
> **作者**：41 人，第一作者 Xiangyi Li，参与者包括 Jiankai Sun、Xuandong Zhao、Han-chung Lee 等

---

## 1. 这篇论文为什么重要

**一句话**：SkillsBench 是 **agent skill 时代的"GLUE benchmark"**——首次用 7,308 条轨迹 × 11 个领域 × 86 个任务，**实证回答了"skill 到底有没有用、怎么用最好"**，得出多个反直觉的关键结论。

为什么半年票数榜能排到第二（仅次于 GLM-5）：
1. **时机刚好**——2026 H1 整个 skill 主题（SkillOpt、SkillClaw、Skill1、COLLEAGUE.SKILL、LatentSkill）爆发，但**没人系统评测过 skill 到底带来多少**
2. **结论硬核**——"**self-generated skill 平均无收益**" 这条单挑战了 SkillOpt 之外的几乎所有自演化 skill 工作
3. **可复现**——deterministic verifier 让所有 baseline 都能复现，避免主观判分

这是 [[skillopt]] / [[skillclaw]] / [[colleague-skill]] 等所有 skill 类工作**必须挂的评测标杆**。

---

## 2. 评测设计

### 2.1 任务矩阵

```
11 领域 × 86 任务
├── 软件工程     ├── 医疗
├── 数据分析     ├── 金融
├── Web 研究     ├── 法律
├── 创意写作     ├── 教育
├── 数学         ├── 科学
└── 通用助手
```

每个任务都配备：
- **人工策划的 curated skill**（专家撰写、~300-2000 token Markdown）
- **deterministic verifier**（基于规则/正则/单测，不靠 LLM judge，避免循环依赖）

### 2.2 三种对照条件

| 条件 | 含义 | 评测目标 |
|------|------|---------|
| **No Skill** | 裸 agent，只看 task 指令 | baseline |
| **Curated Skill** | 注入专家写的 skill | skill 上限 |
| **Self-generated Skill** | 让模型自己写 skill，再注入 | skill 能否自举 |

### 2.3 7 个模型配置 × 全 matrix → 7,308 trajectories

覆盖闭源 + 开源、强 + 弱、多个模型族——保证结论不是某个特定模型的偶然。

### 2.4 多 skill 模块化测试

除了"是否给 skill"，还测**给几个 skill 最优**——结果：**2-3 个最佳**，更多反而稀释。

---

## 3. 关键实验发现（**全部反直觉**）

### 3.1 主结果：Curated skill 显著有效，但**领域差异巨大**

| 领域 | Curated Skill 提升 |
|------|------------------:|
| **医疗** | **+51.9 pp** 🥇 |
| 数据分析 | +XX pp |
| Web 研究 | +XX pp |
| ... | ... |
| **软件工程** | **+4.5 pp** ⚠ |
| **平均** | **+16.2 pp** |

> **观察 1**：医疗领域提升 +51.9pp，软件工程仅 +4.5pp。差距 11×。
> **解读**：当模型在某领域**预训练数据稀疏**时，skill 的相对增益巨大；反之 base 已经强的领域 skill 收益边际化。

### 3.2 16/84 任务出现**负向效果**

curated skill **让模型表现变差**的任务有 16 个——说明 skill 不是无脑加就好，**有些 skill 反而干扰了 base 模型的强项**。

### 3.3 Self-generated skill 的**坏消息**

> **观察 2**："Self-generated Skills provide **no benefit on average**" —— 模型无法可靠地自行编写其所受益的程序性知识

这条**直接挑战**了一批"自演化 skill"工作的核心假设（如 [[skillclaw]] 的 Agentic Evolver、Voyager 风格的 skill library 自构建）。

**SkillOpt 为什么例外**：SkillOpt 用了 **validation gate + 强 optimizer model**（GPT-5.5）做 reflection——光让 base model 自己写不够，必须有 stronger evaluator 把关。SkillsBench 验证了这条 setup 的必要性。

### 3.4 模块化的反直觉发现

最优 skill 数 = **2-3 个**，而不是越多越好。
> **解读**：skill 是"局部专家"，太多 skill 进入 context 反而让 attention 分散；少而精胜过多而泛。

### 3.5 "Skill 让小模型追上大模型"

> 配备 skill 的小模型可媲美无 skill 的大模型

——这条本质上是给 [[skillopt]] 的 "Frozen large + external skill" 范式提供了**实证背书**：在 frontier 闭源时代，skill 真的可以替代部分模型规模。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **Skill 类工作的事实评测基准**
   - 之前 SkillOpt、SkillClaw 都用各自构造的小规模测试集
   - SkillsBench 后，所有 skill 类工作要"对标 SkillsBench"才有可比性

2. **Self-generated skill 的"祛魅"**
   - 揭穿了"让 agent 自己写 skill 自我提升"的简单浪漫
   - 把研究焦点逼向 "**怎样的 setup（validator/optimizer/loop）让 self-generation 有用**"

3. **领域敏感性的提示**
   - skill 在专业领域（医疗、法律、金融）收益远大于在通用领域
   - 给工业应用一个清晰信号：**先找 base 模型最弱的领域上 skill**

### ⚠ 局限

- **41 作者、众包式贡献**——curated skill 质量难免参差，不同领域不同人写，难免风格差异
- **86 个任务**对 11 个领域不算密集（平均每领域 ~8 个）
- **deterministic verifier** 限制了任务类型——开放创作类很难纳入
- 未公开论文级别细节——只看摘要难以判断 self-generated skill 失败的细分原因

### 🔮 揭示的趋势

1. **Skill 不是 silver bullet**——领域敏感、模块数敏感、生成方法敏感
2. **"Skill quality > skill quantity"**——2-3 个高质量 skill > 10 个一般 skill
3. **小模型 + skill ≈ 大模型** 让"frozen 适配"路线获得验证（呼应 [[skillopt]] 的核心论点）

### 📊 影响的论文链

- **SkillOpt** [[skillopt]]：SkillsBench 实质上**验证了 SkillOpt 论点**——validation gate + strong optimizer 是 self-generated skill 有效的前提
- **SkillClaw** [[skillclaw]]：被 SkillsBench 部分挑战——"Agentic Evolver 让 skill 自演化"在 base model 自写情形下不 work，需要外部强 evaluator
- **LatentSkill**（W24）：从"in-context 文本 skill"走向"in-weight latent skill"，部分受 SkillsBench 启发——既然 in-context skill 有上限，把它内化是出路

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2602.12670
- **HF Papers**: https://huggingface.co/papers/2602.12670（1,330↑）
- **机构**: BenchFlow（提交者 xdotli），41 作者
- **第一作者**: Xiangyi Li
