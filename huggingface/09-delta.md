# DelTA — 把 RLVR 重新看作"线性判别器"，做 Token 级 Credit Assignment

> **arXiv**：2605.21467（2026.05）｜**机构**：3 作者（Kaiyi Zhang、Wei Wu、Yankai Lin；机构未披露）
> **HF 周榜**：W21 / May 17-23，#4，204↑
> **关键词**：RLVR · Discriminative View · Token Credit Assignment · Self-normalized Surrogate

---

## 1. 这篇论文为什么重要

**一句话**：DelTA 提出了 **RLVR 训练的"判别器视角"**——policy gradient update 隐式地在 token 梯度向量上构造一个**线性判别器**，而标准 RLVR 下这个判别器被高频共享 token（如格式 token）主导，**稀疏但判别性强的方向被稀释**。DelTA 用 token 级系数重加权这个判别器，在 Qwen3-Base 上**+3 分**。

为什么这个 reframing 重要：
- RLVR（GRPO / PPO + verifiable reward）是 2026 H1 推理 LLM 训练的主流方法
- **"为什么 RLVR 有效"** 的解释一直停留在 "贝叶斯+RL" 抽象层
- DelTA 给了**几何/判别角度**的解释——把 update 写成"**正负质心**之差"
- 这种 reframing 让后续工作可以做**几何上更精确的修改**

类似工作链：[[sdar]] 用 sigmoid 门控、WMSS [[weak-driven]] 用 entropy dynamics、DelTA 用判别器视角——**三者都在做 RLVR 的 token 级精细化**。

---

## 2. 核心方法

### 2.1 关键洞察：RLVR Update = 线性判别器

**论文核心论点**：在标准序列级 RLVR 下，policy gradient 的更新方向可以写为：

$$
\nabla_\theta \mathcal{L}_{\text{RLVR}} \propto \underbrace{\bar{g}_+ - \bar{g}_-}_{\text{判别方向}}
$$

其中 $\bar{g}_+$ / $\bar{g}_-$ 是**优势加权平均**形成的"正侧/负侧质心"。

**核心问题**：
- 格式 token（如 `\n`、`Step 1:`、`</answer>`）**在正样本和负样本中都频繁出现**
- 它们贡献的梯度 $g_t$ **既出现在 $\bar{g}_+$ 也出现在 $\bar{g}_-$**
- → 两个质心之差被这些 token "**拉到中间**"，**真正稀疏判别**的 token（如关键推理步骤）的方向被稀释

类比：训分类器时，所有样本都有的特征对判别毫无贡献，但因为高频会主导梯度。

### 2.2 DelTA 解法：Token 系数重加权

**步骤**：

#### Step 1: 估计 token 系数
对每个 token 估计一个系数 $\alpha_t$：
- 若 token 在**特定侧（正或负）独有**：放大其系数
- 若 token 是**共享或弱判别方向**：降低其系数

#### Step 2: 重加权自归一化 RLVR 替代目标
$$
\mathcal{L}_{\text{DelTA}} = \sum_t \alpha_t \cdot \mathcal{L}_{\text{RLVR}}^{\text{self-norm}}(t)
$$

**为什么用 self-normalized surrogate**：避免简单乘 $\alpha_t$ 后总 loss scale 失衡。

### 2.3 几何效果

```
普通 RLVR:                       DelTA:
  ╲      格式 token 拉到中间       格式 token 系数 ↓
   ╲   ┌─ ─ ─ ─ ─ ─┐              ┌──────┐
    ╲  │  +质心    │              │ +质心 │
     ╲ │  ╲       │              │ ↑    │ 推理 token 系数 ↑
      ╲│ ─ ─ ── ─ │              │ ╲   │ → 判别方向更对比性
       │  -质心    │              │ -质心│
       └ ─ ─ ─ ─ ─┘              └──────┘
       judgement 模糊             清晰
```

**结果**：有效的侧向质心**更具对比性**，判别方向更精确。

---

## 3. 关键实验结果

### 3.1 主结果（数学基准）

| Backbone | 评测 | 平均提升 |
|----------|------|---------:|
| **Qwen3-8B-Base** | 7 个数学 benchmark | **+3.26 分** |
| **Qwen3-14B-Base** | 7 个数学 benchmark | **+2.62 分** |

> 7 个数学 benchmark 通常包括 AIME 24/25、AMC、MATH500、HMMT 等。

### 3.2 跨域泛化验证

| 泛化方向 | 验证情况 |
|---------|---------|
| **代码生成** | ✅ 验证有效 |
| **不同 backbone** | ✅ 验证（非 Qwen 系列） |
| **OOD 评测** | ✅ 验证（域外评测） |

→ **不是 Qwen-specific 的 trick**，是普适机制。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **RLVR 理论框架的新视角**
   - 之前 RLVR 多用 RL/Bayes 语言解释
   - DelTA 用**判别学习**语言重新解释
   - 这个 reframing 可以指导**更精细的 RLVR 改进**

2. **Token-Level Credit Assignment 的代表作之一**
   - 与 [[sdar]] 的 sigmoid 门控、[[weak-driven]] 的 entropy dynamics 并列
   - **2026 H1 token 级 RL 调优**的三个代表角度
   - 后续 token-level RLVR 工作必须挂这三篇

3. **"格式 token 稀释判别"的诊断**
   - 解释了为什么 RLVR 训出来的模型 chain-of-thought **格式 stable 但内容质量参差**
   - 启发了 future work：是否应该**显式分离 format token 和 content token**

### ⚠ 局限

- **+3 分**幅度中等——是否值得增加 RLVR pipeline 的复杂度需要权衡
- 摘要未披露 token 系数 $\alpha_t$ 的**具体估计方法**（是 learned classifier、heuristic、还是 information-theoretic）
- 在更大模型（30B+）上是否仍有效未验证
- 与 [[sdar]] / [[weak-driven]] 的横向对比缺失
- 3 作者小团队，验证规模相对有限

### 🔮 揭示的趋势

1. **RLVR 进入精细化解释阶段**
   - 不再是 "RLVR + reward 凑数 = SOTA"
   - 而是 "**为什么** RLVR 这样更新 = 什么 token 在贡献"

2. **判别学习思想回归 LLM 训练**
   - 之前判别 vs 生成是泾渭分明的分类
   - DelTA 表明 **RLVR 本质上是带生成约束的判别学习**

3. **Self-normalized 目标普及**
   - GRPO 已经用了 self-normalization
   - DelTA 进一步扩展自归一化的边界
   - 后续 RLVR 变种大概率都会带 self-normalization 项

### 📊 同方向工作

- [[sdar]]（W20，浙大+美团+清华）：OPSD + sigmoid 门控
- [[weak-driven]]（W07）：弱 checkpoint + entropy dynamics
- W21 Anti-Self-Distillation（rednote）：PMI 抑制错误偏置——与 DelTA 同周、同主题
- W16 KnowRL：minimal-sufficient knowledge 引导 RL
- [[grandcode]] 的 Agentic GRPO：解决 multi-stage rollout 漂移（trajectory 级修正）
- DelTA / SDAR / Anti-Self-Distillation **三篇并列代表 2026 H1 "RLVR 精细化"潮流**

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.21467
- **HF Papers**: https://huggingface.co/papers/2605.21467（204↑）
- **作者**: Kaiyi Zhang, Wei Wu, **Yankai Lin**（人民大学常见作者署名风格）
