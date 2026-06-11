# ARIS — 对抗式多 Agent 协作的自主科研 Harness

> **论文标题**：ARIS: Autonomous Research via Adversarial Multi-Agent Collaboration
> **arXiv**：2605.03042（2026.05）｜**机构**：上海交通大学 + 上海创新研究院 (SII)
> **HF 周榜**：W19 #3，131↑
> **关键词**：Research Harness · Cross-Family Adversarial Review · Evidence-to-Claim Audit · Markdown Skills
> **GitHub**：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep

---

## 1. 这篇论文为什么重要

**一句话**：ARIS 用「**单 agent 长程任务永远不可靠**」这条**严苛假设**作为出发点，提出业界首个**跨模型对抗式协作**的研究 harness——executor 用一个模型族，reviewer 用**另一个**模型族，强制打破 self-refinement 的盲点。

这篇工作切中 2026 自动化科研系统的核心痛点：**plausible unsupported success**（结果看起来真实但证据不足，或被悄悄继承了 executor 的 framing）。

**为什么这是个真问题**：
- AI Scientist (Lu et al., 2024)、AI Scientist v2、Agent Laboratory 等先驱系统都用**同模型 self-refinement**
- 论文给了一个精彩比喻：**single-model self-review 像 stochastic bandit**（可预测的奖励噪声），**cross-model review 像 adversarial bandit**（reviewer 主动找 executor 没料到的弱点）—— **adversarial bandit 在博弈论上更难"骗"**

**对上交学生的关注价值**：作者来自 Shuai Li 实验室（SJTU + SII），是同校的旗舰工作，**可作为本校合作切入点**。

---

## 2. 核心方法

### 2.1 三层架构

```
┌──────────────────────────────────────────────┐
│  Assurance Layer（保障层）                    │
│  - 3 阶段证据-声明审计                          │
│  - 5 遍科学编辑流水线                          │
│  - 数学证明检查                                │
│  - 视觉 PDF 审查                              │
│  - 引文审计                                    │
└──────────────────────────────────────────────┘
                  ↑
┌──────────────────────────────────────────────┐
│  Orchestration Layer（编排层）                │
│  - 5 个端到端 workflow                        │
│  - 4 级 effort（lite/balanced/max/beast）    │
│  - Reviewer routing (Codex/Oracle/MCP)       │
└──────────────────────────────────────────────┘
                  ↑
┌──────────────────────────────────────────────┐
│  Execution Layer（执行层）                    │
│  - 65+ 可复用 Markdown skills                 │
│  - MCP 模型集成                               │
│  - Per-project Research Wiki（持久化记忆）    │
│  - 确定性 FigureSpec 渲染器                   │
└──────────────────────────────────────────────┘
```

### 2.2 跨模型对抗式协作（核心机制）

**Critique-to-action loop**：
1. **Executor** 产出 artifact（代码 / 论文段落 / 实验设计）
2. **Reviewer**（**来自不同模型族**）按 rubric 打分 + 返回结构化 action items
3. Executor 处理 → convergence check → 接受或下一轮
4. 终止条件：评分 ≥ 默认 6/10 且关键 review item 全部解决，**或** 达到默认 4 轮

**默认 cross-family 配对**：
- Claude 系（executor） + GPT 系（reviewer，通过 Codex MCP / Oracle MCP）
- 或反之
- 也支持 Gemini、MiniMax、GLM、Kimi、DeepSeek 等

**Reviewer 独立性原则**：
- Reviewer 直接读 artifact 文件，**不读 executor 的总结**
- 否则 reviewer 评估的是 executor 的 framing，不是底层工作

### 2.3 5 个 Workflow

| Workflow | 作用 |
|---------|------|
| W1: Idea Discovery | 文献调研 + 想法生成 + reviewer-gated 优化 |
| W1.5: Experiment Bridge | code review + auto-debug fallback |
| W2: Auto Review Loop | 多轮 reviewer 评分 + revise + 必要时启动 GPU 实验 |
| W3: Paper Writing | 三阶段：Plan&Generate / Draft&Assure / Compile&Improve |
| W4: Rebuttal | safety gates + stress test |

### 2.4 三阶段证据-声明审计（最有创意的部分）

**Stage 1: Experiment-integrity audit**（`/experiment-audit`）
跨模型 reviewer 审计 5 类完整性失败：
1. **Model-derived reference labels**（用模型自己输出做 ground truth）
2. **Self-normalized scores**（指标分母来自自己的预测）
3. **Phantom results**（论文报的数字与实际输出不符）
4. **Dead-code / unused-metric inflation**（描述了但从未执行的 metric）
5. **Scope inflation**（声明超出测试范围）

输出：`EXPERIMENT_AUDIT.md` + JSON summary

**Stage 2: Result-to-claim mapping**（`/result-to-claim`）
每个候选 claim 评估为三档：
- supported / partially supported / invalidated

输出：**claim ledger**（声明账本）

**Stage 3: Paper-claim audit**（`/paper-claim-audit`）
**Fresh zero-context reviewer**（新开 Codex 线程，无历史）独立交叉核对：
- numerical mismatches
- best-seed cherry-picking
- config mismatch
- aggregation errors
- scope overclaim

每个 claim 标注：exact_match / rounding_ok / number_mismatch / config_mismatch / missing_evidence

### 2.5 5 遍科学编辑流水线（`/paper-write`）

灵感来自 Sainani 的科学写作教学：
1. **Clutter removal**：删除填充词、冗余
2. **Active voice**：被动 → 主动
3. **Sentence structure**：改善 topic 定位
4. **Terminology consistency**：跨章节术语一致性检查
5. **Numerical consistency**：交叉核对数字与表/图/引文

### 2.6 Research Wiki（持久化项目记忆）

四个实体类型作为结构化 Markdown 页：
- papers / ideas / experiments / claims

八种 typed relationship：
- extends / contradicts / addresses_gap / inspired_by / tested_by / supports / invalidates / supersedes

**关键设计**：**保留被拒绝的 idea**。没有持久化记忆，pipeline 会反复尝试同一个 dead-end；有了 wiki，被拒 idea 进入 banlist，已验证 claim 成为下一轮 ideation 的基础——**把 one-shot research 变成 spiral learning**。

### 2.7 Meta-Optimization（原型外环优化）

三个组件：
1. **Passive event logging**：Claude Code hooks 记录到 `.aris/meta/events.jsonl`
2. **Pattern analysis**：`/meta-optimize` 分析使用统计，提出 SKILL.md patch 建议
3. **Reviewer-gated application**：GPT-5.4 xhigh 审 patch，仅评分 ≥ 7/10 的推荐给用户**（永不自动应用）**

---

## 3. 关键实验 / 部署证据

### 3.1 系统规模（v0.4, 2026 April）

| 指标 | 数值 |
|------|------|
| 核心 skills | 21 → **65+** |
| MCP bridges | 6 个（Codex, GPT-5.4 Pro, Gemini, Claude, MiniMax, OpenAI-compatible） |
| 支持的 executor 平台 | Claude Code, Codex CLI, Cursor（社区适配指南再 +3） |

### 3.2 一次过夜运行的实例（Workflow 2 演示）

**单次 8 小时运行**：
- 完成 **4 轮 review-revise**
- Reviewer 评分从 5.0 提升到 **7.5/10**
- 启动 **20+ GPU 实验**
- 移除了证据不足的 claim

**论文明确警告**：这是**单一观察证据**，不能因果归因到 ARIS 本身，需要 controlled benchmark（Appendix E 列为 future work）。

### 3.3 与其它自主研究系统对比

ARIS 区别于：
- **AI Scientist (Lu et al., 2024)** / **AI Scientist v2**：端到端但用同模型 self-refinement
- **AI co-scientist**：偏 hypothesis generation
- **Agent Laboratory**：human-in-loop checkpoints
- **data-to-paper (Ifargan et al., 2025)**：标注数据→论文，强调可追溯性

**ARIS 独有**：
- ✅ Cross-family executor-reviewer 分离（默认配置）
- ✅ 65+ 可复用 Markdown skill 规约
- ✅ Per-project research wiki 持久化记忆
- ✅ 跨多个 executor 平台可移植
- ✅ 完整 assurance stack

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **"plausible unsupported success"成为新概念**
   - 把研究 agent 的失败模式从"明显崩溃"重新定义为"看起来真实但证据不足"
   - 给自主研究系统设计带来全新评估维度

2. **跨模型审查正式确立为 design principle**
   - 之前 multi-agent debate（Du et al. 2024）只是 reasoning trick
   - ARIS 把它升级为**系统级架构原则**

3. **同领域并行 / 后续工作**
   - W21 **AutoResearchClaw**（自演化 DR + 人机协作）
   - **EvoScientist**（多 agent 进化式科研）
   - **AutoSoTA**（自动化 SOTA 探索）
   - 论文里专门提到了 Pantheon 项目作为 auto-research 系统全景图

### ⚠ 局限（论文自己承认得很坦诚）

- ❌ 没有受控对照评估（cross-family 优势仅靠观察证据）
- ❌ **No guarantee of correctness** ——LLM 仍会幻觉
- ❌ Reviewer bias amplification 风险（loop 可能 overfit reviewer 偏好）
- ❌ Repository-level review 可能泄露敏感代码
- ❌ Audit 是 advisory，不是 formal verification

### 🔮 揭示的趋势

1. **"Harness engineering" 成为新研究主题**
   - 系统级 logic（context、storage、retrieval）的设计与模型权重同等重要
   - Meta-Harness (Lee et al. 2026) 形式化了这个领域

2. **持久化记忆（research wiki）取代 ephemeral context**
   - 从 stateless LLM → file-system-as-state

3. **Cross-model accountability 可推广到其他 AI 系统**
   - 论文末尾推测：可用于 self-improvement loop（避免 model collapse、judge bias）
   - 与 Constitutional AI、RLAIF 形成 oversight layer

### 📊 同方向工作（2026）
- W21 AutoResearchClaw
- W24 ResearchClawBench
- W23 Crafter（multi-agent scientific figure generation）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.03042
- **HF Papers**: https://huggingface.co/papers/2605.03042
- **GitHub**: https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
- **ARIS-Code CLI**：标准 Rust 二进制，集成所有 skills
- **作者**: Ruofeng Yang, Yongcan Li, **Shuai Li** (SJTU + SII)
- **机构**: 上海交通大学 + 上海创新研究院
- **联系**: shuaili8@sjtu.edu.cn
