# TermiGen — 终端 Agent 的高保真环境与韧性轨迹合成

> **arXiv**：2602.07274（2026.02，ICML 2026）｜**机构**：UCSB MLSec
> **HF 周榜**：W07 #3，210↑
> **关键词**：Terminal Agent · Environment Synthesis · Error Injection · Trajectory Distillation
> **GitHub**：https://github.com/ucsb-mlsec/terminal-bench-env

---

## 1. 这篇论文为什么重要

**一句话**：TermiGen 是 **首个端到端的终端 agent 数据合成框架**，让 32B 开源模型在 TerminalBench 上拿到 **31.3% pass rate（开源 SOTA）**，**击败 OpenAI o4-mini (20%)**。

这篇工作回答了 2026 年开源 agent 圈最迫切的问题：**为什么开源模型在 agent 任务上离闭源差这么远？**

答案：**数据**。专门指出两个 data-centric 瓶颈：
1. **可执行环境稀缺**：手工构建太贵（TerminalBench 200 个任务都靠人工），LLM 合成轨迹有严重幻觉
2. **专家轨迹缺乏「错误恢复」示范**：强模型很少犯简单错误，学生模型学不到怎么修自己的错

---

## 2. 核心技术贡献

### 2.1 整体两阶段流水线

```
Phase I: 高保真任务 + 环境合成
       ↓
       3500+ 已验证的 Docker 容器环境
       ↓
Phase II: 主动错误注入的轨迹合成（Generator-Critic）
       ↓
       3291 条富含 "error → diagnosis → recovery" 循环的轨迹
       ↓
SFT on Qwen-2.5-Coder-32B → TermiGen 模型
```

### 2.2 Phase I：高保真环境合成（解决"环境稀缺"）

**多 agent 分工**生成可执行环境：

1. **Task Generation**：用 11 类 / 3 层级的分类法（从「Linux 内核 gcc 依赖」到「Coq 定理证明」），LLM 生成 task seed
2. **Proposer-Evaluator 架构**：Proposer 给出具体任务规约，Evaluator 在三个维度打分（环境复杂度、数据可生成性、验证确定性），只保留 >4 分的
3. **Environment Synthesis**：File Planner → Construct Agent → Env Agent 三步生成 Dockerfile
4. **关键创新——Docker 验证循环**：每个生成的 Dockerfile 实际 docker build，失败 → stderr 反馈 → 修正，**保证 100% 环境可执行**
5. **Unit Test 生成 + Judge Agent 验证**：确保任务可解、单测有判别力

**对比传统方法**：
- 基于 GitHub repo 挖掘（如 SWE-smith）：缺多样性、偏 bug fixing
- 纯 LLM 模拟环境：26% 的轨迹有 observation 错误（spurious verbosity 53%、semantic deviation 35%、state inconsistency 12%）

### 2.3 Phase II：主动错误注入（解决"错误恢复缺失"）

**Generator-Critic 三步循环**：

```
Step 1: Intent Sampling  →  I_t ~ Bernoulli(ε=0.2)
                            {正确 with 0.8, 错误 with 0.2}

Step 2: Context-Aware Generation → 按 I_t 生成动作
        如果 I_error，从 5 类失败模式中采样：
          - Analysis Errors（误解环境状态）
          - Command Errors（语法错）
          - Hallucinations（假设不存在的工具）
          - Requirement Violations（忽略约束）
          - Verification Failures（没做自检）

Step 3: Critic 验证：确认 action 与 intent 对齐
```

**关键设计**——**Recovery 机制**：当 intent 切回 correct 时，Step Generator 被强制 **诊断前一步的错误 + 合成修复动作**，产生 `error → diagnosis → recovery` 循环。

**反直觉发现**：随机注入的错误**不会降低任务成功率**，因为后续步骤被强制恢复。

### 2.4 数据集统计

- **3500+ 已验证环境**（覆盖 11 个领域）
- **3291 条轨迹**（平均 25.5 turns，8722 tokens）
- **420 个独特的 CLI 工具**（16 个功能域）
- 包含**成功 + 失败**轨迹

---

## 3. 关键实验结果

### 3.1 TerminalBench 1.0 主榜（pass rate）

| 模型 | Pass Rate |
|------|----------|
| Apex2 + Claude-4.5-Sonnet | 64.5% |
| Codex CLI + GPT-5-Codex | 42.8% |
| Claude Code + Claude-3.7-Sonnet | 35.2% |
| **TermiGen-Qwen2.5-Coder-32B** | **31.3%** 🥇（开源 SOTA）|
| **TermiGen-Qwen3-32B** | 27.5% |
| Codex CLI + o4-mini | 20.0% |
| Reptile + Devstral-22B | 18.9% |
| GPT-OSS-120B | 19.2% |
| Qwen3-32B (base) | 10.4% |
| Qwen2.5-Coder-32B (base) | **4.5%**（提升 7x ！）|

### 3.2 三个核心消融实验

**RQ1：可验证环境 vs LLM 模拟**（控制 800 样本）
| 设置 | Pass Rate |
|------|----------|
| Simulation-based | 22.5% |
| **Verifiable Docker** | **25.0%**（+2.5%，相对提升 ~11%）|

**RQ2：错误注入 vs 标准轨迹**（控制 800 样本）
| 设置 | Pass Rate |
|------|----------|
| Standard expert trajectories | 21.8% |
| **+ Error injection (ε=0.2)** | **25.0%** |

**RQ3：质量过滤的「反常识」结果**
| 通过率过滤阈值 | Pass Rate |
|---------------|----------|
| τ = 100%（只留完美轨迹）| 较低 |
| τ = 50% | 中等 |
| **τ = 0%（不过滤）** | **最高** |

→ **不完美数据反而最有价值**：失败轨迹中的局部 recovery 片段提供高质量「诊断+修复」监督

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **「Environment-as-data」范式确立**：从「人工写 benchmark」转向「自动合成 + 自动验证」
   - 类似工作如 W17 Agent-World、W13 CUA-Suite 都遵循类似思路
2. **错误注入成为 agent SFT 的标准技巧**：纠正了「expert trajectory 越干净越好」的旧思维
3. **开源追赶闭源的清晰路径**：32B 模型只要数据合成做得好，可以打过 o4-mini

### ⚠ 局限

- 仅 SFT，未做 RL（论文已承认是 future work，因为环境已经提供 deterministic verification 信号）
- 简单 BashAgent 架构，无记忆模块
- 合成环境是隔离 sandbox，与真实分布式生产系统仍有 gap

### 🔮 揭示的趋势

1. **Data Synthesis is the new model architecture**：数据合成的质量决定开源 agent 上限
2. **「错误数据」比「干净数据」更有价值**——一种反直觉但正在被反复验证的发现
3. **Multi-agent 协作不只用在 inference，也用在 data generation**

### 📊 后续 follow-up（已经在 W08-W24 出现）
- W08 RynnBrain（具身环境合成）
- W13 OpenResearcher（research trajectory 合成）
- W17 Agent-World（通用环境合成）
- W21 Video2GUI（视频→GUI 轨迹合成）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2602.07274
- **HF Papers**: https://huggingface.co/papers/2602.07274
- **GitHub**: https://github.com/ucsb-mlsec/terminal-bench-env
- **作者**: Yuzhou Nie, Yijiang Li, ..., Wenbo Guo (UCSB)
