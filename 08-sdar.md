# SDAR — 自蒸馏式 Agentic 强化学习

> **论文标题**：Self-Distilled Agentic Reinforcement Learning
> **arXiv**：2605.15155（2026.05）｜**机构**：浙江大学 + 美团 + 清华
> **HF 周榜**：W20 #8，111↑
> **关键词**：On-Policy Self-Distillation · Token-Level Gating · Asymmetric Trust · Multi-turn Agent RL
> **GitHub**：https://github.com/ZJU-REAL/SDAR

---

## 1. 这篇论文为什么重要

**一句话**：SDAR 用一个**简单优雅的 sigmoid 门控**，把 On-Policy Self-Distillation (OPSD) 安全地接入了多轮 agentic RL，在 ALFWorld、WebShop、Search-QA 上**击败 GRPO + 9.4% / 7.0% / 10.2%**，**避免了 naive GRPO+OPSD 的灾难性不稳定**。

这是 2026 Agentic RL 算法层最重要的"踏实"进步之一——不是 hype 性能数字，而是**系统揭示并修复了一类失败模式**：

> "Multi-turn agents 上，RL 应该是主目标，OPSD 应该是被严格控制的辅助。"

这种"诊断 + 修复"风格的论文，往往比堆 SOTA 的论文更有学术生命力。

---

## 2. 论文要解决的两个核心观察

### Observation 1: Multi-turn OPSD 不稳定

**OPSD（On-Policy Self-Distillation）原本想做什么**：
- RL 只提供 **trajectory 级稀疏奖励**
- OPSD 让 teacher 分支（同模型 + privileged context，如检索的 skill）提供 **token 级密集监督**
- 听起来很美

**实际发生了什么**：
- 一旦 student agent 偏离 teacher 支持的 trajectory，token 级监督**越来越不可靠**
- 复合错误 → 每轮 KL divergence 暴涨 → 任务性能灾难性退化
- TCOD 等先前工作试图用 curriculum learning 解决，但依赖**僵硬的时间/深度阈值**

### Observation 2: Privileged Guidance 的非对称性

**OPSD 的 teacher 不是更强的模型，而是同一个 policy 加了 privileged context**（如检索的 skill）。这让 teacher 的 token 级信号**天然非对称**：

| Gap 方向 | 含义 | 应该如何对待 |
|---------|------|-------------|
| **Positive gap**（teacher prob > student） | 检索 skill 提供 endorsement signal —— student 能生成但还没内化 | **强化 distillation** |
| **Negative gap**（teacher prob < student） | **可能**应该抑制，**也可能**是 privileged context 的不稳定造成（skill 质量差、skill 利用差、multi-turn drift） | **谨慎抑制** |

**作者初步实验显示**：在 Qwen2.5-3B 上，**negative-gap tokens 超过 50%**——这种问题非常普遍。

这两个观察组合得出一个核心洞察：
> **对 multi-turn agents，RL 应做主轴，OPSD 应被严控为辅助；强信任 positive endorsement，软处理 negative rejection。**

---

## 3. 核心方法

### 3.1 目标函数（保持 RL 不变 + 加门控辅助）

$$\mathcal{L}(\theta) = \mathcal{L}_{\text{GRPO}}(\theta) + \lambda_{\text{SDAR}} \cdot \mathcal{L}_{\text{SDAR}}(\theta)$$

**关键设计哲学**：verifier-driven RL policy loss **完全不动**，严格保留 RL advantage 的语义和无偏性。OPSD 完全作为辅助 objective。

### 3.2 Token 级 Gating（三种策略）

**Teacher-Student log-probability gap**（detached）：
$$\Delta_t = \text{sg}\left(\log \pi_{\theta^+}(y_t \mid s_t^+) - \log \pi_\theta(y_t \mid s_t)\right)$$

**Student entropy**：
$$h_t = -\sum_{v \in \mathcal{V}} \pi_\theta(v \mid s_t) \log \pi_\theta(v \mid s_t)$$

**三种 sigmoid 门控**：

1. **Entropy gating**：$g_t = \sigma(\beta h_t)$ —— 针对 student 最不确定的高熵位置
2. **Gap gating**：$g_t = \sigma(\beta \Delta_t)$ —— **正 gap token 加权，负 gap token 软抑制**（这是最核心的不对称设计）
3. **Soft-OR gating**：$g_t = \sigma(\beta[1 - (1-h_t)(1-\Delta_t)])$ —— 组合两者

**Token 级 loss**：
$$\ell_t^{\text{SDAR}} = g_t \cdot (\log \pi_{\theta^+}(y_t \mid s_t^+) - \log \pi_\theta(y_t \mid s_t))$$

**关键设计**：
- $g_t$ 经过 `sg(⋅)` 截断梯度，gradient 只通过 student log-prob 流动
- 完全 smooth、differentiable、自然 bounded 在 $(0, 1)$
- $\beta > 0$ 控制陡峭度

### 3.3 Skill Retrieval（四种策略测试鲁棒性）

privileged context $c^+$ 来自检索的 skill：

1. **UCB Retrieval**（多臂老虎机）
2. **Keyword Matching**（KM）
3. **Full Retrieval**
4. **Random Retrieval**（用来测试鲁棒性）

**核心发现**：SDAR 在 random retrieval 下**仍然超过 GRPO baseline**——门控成功过滤了低质量 skill 的噪声。

---

## 4. 关键实验结果

### 4.1 主结果（Qwen2.5-3B-Instruct 上）

| 方法 | ALFWorld Avg | Search-QA Avg | WebShop Score | WebShop Acc |
|------|-------------:|-------------:|-------------:|-----------:|
| Vanilla | 21.9 | 31.7 | 6.7 | 0.8 |
| Skill-Prompt* | 28.9 | 23.9 | 0.2 | 0.8 |
| OPSD | 28.1 | 0.0 ⚠️ | 11.3 | 3.1 |
| **GRPO** | **75.0** | 36.4 | **79.8** | 63.3 |
| Skill-GRPO | 60.2 | 34.1 | — | — |
| **SDAR** | **+9.4%** vs GRPO | **+7.0%** vs GRPO | — | **+10.2%** |

**惊人的失败案例**：单独 OPSD 在 Search-QA 上得 **0.0 分**——证实了"multi-turn OPSD instability"的严重性。

### 4.2 与 RL-OPSD 混合基线对比

SDAR 一致击败：
- **Skill-SD**（Wang et al. 2026a）—— 用 rigid 调度
- **HDPO**（Ding 2026）—— 另一种 hybrid
- **RLSD**（Yang et al. 2026a）—— 用 self-divergence re-weight RL advantage

### 4.3 跨模型规模 / 模型族验证

在 **Qwen2.5 + Qwen3 家族**（包括 Qwen3-1.7B）的多个模型规模上一致表现良好——**说明门控设计不是只对某个模型有效**。

### 4.4 鲁棒性消融

| Skill Retrieval | SDAR |
|----------------|------|
| UCB | 最优 |
| Keyword Matching | 接近最优 |
| Full Retrieval | 中等 |
| **Random Retrieval** | **仍超过 GRPO baseline** ✅ |

→ 门控成功**自动过滤噪声**。

---

## 5. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **"诊断 + 修复"型论文的典范**
   - 不堆 SOTA，而是**系统性揭示一个失败模式**（OPSD multi-turn instability + asymmetric trust）
   - 然后**用最小代价**（一个 sigmoid 门）修复
   - 这种风格更易被后续工作引用、扩展

2. **首次形式化「privileged context 的非对称信任」**
   - 之前 RL+distill 都假设 teacher 的所有 token 都可信
   - SDAR 指出：teacher 的 negative gap 可能源于 context 不稳定，不是真的"应该抑制"
   - **这个 insight 可以推广到任何 teacher-student 范式**

3. **Token-Level Gating 成为新工具**
   - 与 TIP（Xu et al. 2026）一脉相承
   - 用 token 级信号控制不同 loss 的强度，成为 2026 H1 一类共同思路

### ⚠ 局限

- 只在 3B / 1.7B / 7B 验证，未在更大模型（30B+）上验证
- 三种门控策略之间的最优选择没有清晰指南
- $\lambda_{\text{SDAR}}$ 和 $\beta$ 的调参依赖经验
- 与 GrandCode 的 Agentic GRPO、Youtu-Agent 的 RL module 没有直接对比

### 🔮 揭示的趋势

1. **Agentic RL 算法层进入精细化阶段**
   - 不再是 GRPO vs PPO 的粗对比
   - 进入"如何把多个监督源（RL reward + distillation + verifiable rewards）协调"的微观工程

2. **OPSD / RL 组合是 2026 的热点**
   - 与 W21 DelTA、Anti-Self-Distillation、Yang et al. 2026 RLSD 形成完整 paper 链
   - SDAR 是其中"最稳"的代表

3. **门控 / 加权式融合**取代硬性 curriculum
   - 让每个 token 决定自己监督强度 = 最细粒度的 self-paced learning
   - 类似思路在 W21 DelTA（discriminative token credit assignment）也出现

### 📊 相关工作链
- 前置：OPSD、RLSD、Skill-SD、HDPO、TCOD、TIP
- 并行 / 后续：W21 DelTA、W21 Anti-Self-Distillation、W08 Experiential RL

---

## 6. 资源

- **arXiv**: https://arxiv.org/abs/2605.15155
- **HF Papers**: https://huggingface.co/papers/2605.15155
- **GitHub**: https://github.com/ZJU-REAL/SDAR
- **作者**: Zhengxi Lu, Zhiyuan Yao, Zhuowen Han, Zi-Han Wang, Jinyang Wu, Qi Gu, Xunliang Cai, Weiming Lu, Jun Xiao, Yueting Zhuang, **Yongliang Shen** (Corresponding)
- **机构**: 浙江大学 + 美团 + 清华
- **联系**: zhengxilu@zju.edu.cn, syl@zju.edu.cn, guqi03@meituan.com
- **说明**: 第一作者实习于美团完成此工作
