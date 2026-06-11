# SkillOpt — Agent 技能的 Text-Space 优化器

> **arXiv**：2605.23904（2026.05）｜**机构**：Microsoft + 上交 + 同济 + 复旦
> **HF 周榜**：W22 #2，224↑
> **关键词**：Skill Optimization · Text-Space Learning · Frozen Model Adaptation · Domain Adaptation

---

## 1. 这篇论文为什么重要

**一句话**：SkillOpt 提出了一个**完全不更新模型权重**的 agent 适配范式——只优化一份外部 Markdown 技能文档，就在 **52 个 (模型, 基准, harness) 实验单元上全部夺冠**。

这篇论文回答了 2026 一个核心问题：**当 frontier 模型是 GPT-5.5、Claude 4.5 这种 API-only 闭源模型时，怎么做领域适配？**

传统做法都失败：
- **手工 prompt 工程**：劳动密集、不可保证改进
- **SFT / RL**：闭源模型根本拿不到权重
- **TextGrad / GEPA**：只优化静态 prompt，对 agent 长流程不适用

**SkillOpt 的方案**：把 agent 的"程序性知识"（procedure、tool 用法、领域 heuristics）外化为一份 ~300-2000 token 的 Markdown 技能文档，用一个 frontier 模型当**优化器**反复 propose 编辑，**验证集筛选**后接受。

---

## 2. 核心方法

### 2.1 核心比喻：把深度学习概念映射到文本空间

| 深度学习概念 | SkillOpt 对应物 |
|-------------|----------------|
| 外部状态 | Skill 文档（Markdown 文本）|
| 优化器 | 额外的 frontier model |
| Batch size | Rollout / reflection batch size |
| Learning rate | 文本编辑预算 $L_t$ |
| Validation | 留出 selection split |
| Momentum | Epoch-wise slow/meta update |

**这是一个很优雅的 reframing**——把整个 SGD 训练 loop 搬到文本编辑空间。

### 2.2 优化流程（一个 epoch）

```
┌──────────────────────────────────────────────┐
│ ① Forward Pass (Rollout)                     │
│   frozen target model + current skill        │
│   → 在训练集 D_tr 上跑出 trajectories         │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│ ② Backward Pass (Reflection)                 │
│   optimizer model 分析成功 / 失败 minibatch   │
│   → propose 结构化的 add/delete/replace 编辑 │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│ ③ Bounded Update                             │
│   编辑预算 L_t 限制单步改动量                  │
│   schedule: constant / linear / cosine       │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│ ④ Validation Gate                            │
│   候选 skill 必须在 D_sel 上严格优于当前       │
│   失败 → 编辑进入 "rejected buffer"          │
│   （作为下一轮的 negative feedback）         │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│ ⑤ Epoch-wise Slow/Meta Update                │
│   长程规律写入受保护的 slow-update field      │
│   实现"momentum"语义                          │
└──────────────────────────────────────────────┘
```

### 2.3 核心数学定义

每个 trajectory 的得分：
$$(\tau(s), r(s)) = h(M, x, s), \quad r(s) \in [0,1]$$

skill 选择标准（每轮）：
$$s^{\star}_{sel} = \arg\max_{s \in \mathcal{C}(D_{tr})} \frac{1}{|D_{sel}|}\sum_{x \in D_{sel}} r(s)$$

### 2.4 Harness-agnostic 部署

通过 adapter 接口，同一个 `best_skill.md` 可以无修改用于：
- Direct chat
- Codex CLI
- Claude Code CLI

---

## 3. 关键实验结果

### 3.1 主结果：52/52 全胜

在 **6 个基准 × 7 个目标模型 × 3 个 harness** 的 52 个实验单元中，SkillOpt 是 best 或 tied-best 的次数 = **52/52**，**击败 8 个基线**（no-skill、human-skill、one-shot LLM-skill、TextGrad、GEPA、Trace2Skill、EvoSkill 等）。

**平均每模型改进**：约 **+17.6 分**

### 3.2 GPT-5.5 主结果（Direct Chat 模式）

| 基准 | No Skill | SkillOpt | Δ |
|------|---------:|---------:|---:|
| SearchQA | 77.7 | 87.3 | +9.6 |
| **SpreadsheetBench** | 41.8 | 80.7 | **+38.9** |
| **OfficeQA** | 33.1 | 72.1 | **+39.0** |
| DocVQA | 78.8 | 91.2 | +12.4 |
| LiveMath | 37.6 | 66.9 | +29.3 |
| ALFWorld | 83.6 | 95.5 | +11.9 |

**惊人的是 OfficeQA 和 SpreadsheetBench 都接近翻倍**——说明 skill 文档对"领域专属程序性知识"价值极大。

### 3.3 三种迁移性测试

| 迁移方向 | 结果 |
|---------|------|
| **跨模型**：GPT-5.4 训的 skill → 用在小一些的 GPT 变体 | 全部正增益 |
| **跨 harness**：Codex 训的 SpreadsheetBench skill → 用在 Claude Code | **+59.7 分** |
| **跨基准**：OlympiadBench skill → Omni-MATH | 三个 model scale 都有正增益 |

### 3.4 Skill 文档特征

- **大小**：379 – 1995 tokens（中位 ~920）
- **编辑次数**：每基准只需 **1-4 次** accepted edits
- **成本**：0.6M – 46.4M 训练 tokens（一次成本，部署免费）

### 3.5 基线对比（GPT-5.5 平均分）

```
No skill           65.0
Human skill        68.2  (+3.2)
One-shot LLM-skill 71.4  (+6.4)
Trace2Skill        72.1  (+7.1)
TextGrad           73.5  (+8.5)
GEPA               74.8  (+9.8)
EvoSkill           75.0  (+10.0)
SkillOpt           82.6  (+17.6) 🥇
```

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **「Frozen model + external state」成为闭源时代的新主流范式**
   - 与 prompt optimization、in-context learning、retrieval 同属"非参数化适配"
   - 但**更结构化、更可解释**（skill 文档可读、可手工修订）

2. **挑战了「fine-tuning 是适配必经之路」的旧观念**
   - 在闭源模型时代，权重根本不可得，文本空间优化是唯一可行路径

3. **方法可移植到任何 agentic 系统**
   - 同篇论文跨 Codex CLI、Claude Code CLI、Direct chat 三种 harness 都成功

### ⚠ 局限

- 需要一个**强的 optimizer model**（论文用 GPT-5.5）做 reflection，成本不低
- skill 文档对"程序性知识"有效，对"事实性知识"（如新文献）作用有限
- 没有理论收敛保证（毕竟是文本编辑空间，无 gradient）

### 🔮 揭示的趋势

1. **「Text-Space 优化」成为 2026 的新范式**——SkillOpt 是这条路线的代表作
2. **Skill 库 + 检索** 正在取代 fine-tuning 成为新的「能力封装」方式
3. **「编辑预算 + validation gate」的 SGD-like 框架**可以推广到其他文本资产（prompt、tool description、reward model）

### 📊 同方向并行工作（2026）
- W23 COLLEAGUE.SKILL（专家知识蒸馏生成 skill）
- W19 Skill1（skill-augmented agent RL）
- W17 AgentSPEX（agent 规约与执行语言）
- W24 LatentSkill（in-context skill → in-weight latent skill）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.23904
- **HF Papers**: https://huggingface.co/papers/2605.23904
- **机构**: Microsoft（主导）+ 上交 + 同济 + 复旦
- **作者**: Yifan Yang, Ziyang Gong, Weiquan Huang, Qihao Yang, ..., Chong Luo
