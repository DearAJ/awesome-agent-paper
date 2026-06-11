# Code as Agent Harness — Harness Engineering 的奠基综述

> **arXiv**：2605.18747（2026.05）｜**机构**：42 位作者合作（含 UIUC Hanghang Tong、Jingrui He、UCLA Pan Lu 等知名学者）
> **HF 周榜**：W21 / May 17-23，#3，215↑
> **关键词**：Agent Harness · Code as Substrate · Survey · Interface-Mechanism-Scaling
> **GitHub**：Awesome-Code-as-Agent-Harness-Papers（仓库名）

---

## 1. 这篇论文为什么重要

**一句话**：Code as Agent Harness 是 **2026 H1 "harness engineering" 这一新主题的事实奠基综述**——首次把"代码作为 agent 的 operational substrate"系统化为三层架构，把分散在 ReAct / CodeAct / ToolFormer / [[aris]] / [[skillopt]] 等众多工作中的零散思路统一到一张地图上。

为什么这是个**真问题、新主题**：
- 业界长期把 "code" 看作 agent 的**一个 tool**（写代码 + 执行）
- 这篇论文反转视角：**code 才是 agent 的基础设施本身**——agent 的 reasoning、action、environment modeling 全靠 code 这一统一介质串起来
- 与 [[aris]] 的 "harness engineering" 主张呼应、与 [[skillopt]] 的 "text-space optimization" 互补——三者共同定义了 2026 H1 的"系统级 agent 设计"潮流

对 agent 研究者：**这是一张必须挂上墙的 taxonomy 图**。

---

## 2. 三层架构（论文核心组织方式）

### 2.1 Interface Layer（接口层）

**主张**：code 是连接 agent 与三个核心维度的统一介质：

| 维度 | code 如何实现 |
|------|-------------|
| **Reasoning** | thought / scratchpad 写成代码（如 CodeAct） |
| **Action** | tool call / environment 操作写成代码 |
| **Environment Modeling** | world model 本身可用代码表达（如 [[code2world]]） |

这一层回答："**为什么用 code，而不是 JSON / 自然语言 / DSL？**" —— 因为 code 是唯一**可执行 + 可验证 + 可组合**的介质。

### 2.2 Mechanism Layer（机制层）

把 agent 的核心机制都通过 code 实现：

```
┌─────────────────────────────────────┐
│ Planning             →  code 表达计划   │
│ Memory               →  code 操作存储  │
│ Tool Use             →  code = tool   │
│ Feedback-driven Ctrl →  code 评估反馈  │
└─────────────────────────────────────┘
```

每个机制都展示了**为什么 code 是更好的实现选择**：
- Planning：code 自带 control flow（if/for/while），比自然语言计划更精确
- Memory：code 自带数据结构（dict/list/db API），比 RAG 更结构化
- Tool Use：code 直接调用 tool，不需要中间格式转换
- Feedback：execution result 直接是 feedback，闭环天然成立

### 2.3 Scaling Layer（扩展层）

**单 agent → multi-agent**：靠**共享代码工件**做协调、审查、验证。

| 协作模式 | code 介质 |
|---------|----------|
| 协调 | shared workspace (git/repo) |
| 审查 | code review 模式（与 [[aris]] cross-family review 异曲同工）|
| 验证 | unit test / CI 作为 multi-agent contract |

→ **关键洞察**：multi-agent 系统不需要新协议——**软件工程已有的代码协作机制**（git、PR、CI、code review）就是 multi-agent harness 的现成基础设施。

---

## 3. 应用领域 + 关键挑战

### 3.1 覆盖的 7 个应用领域

| 领域 | 代表工作 |
|------|---------|
| 编程助手 | Claude Code, Cursor, Codex CLI |
| GUI / OS 自动化 | Computer-Use Agent, OSWorld |
| 具身智能体 | RynnBrain, MolmoAct2 |
| 科学发现 | [[aris]], AI Scientist, [[mirothinker]] |
| 个性化推荐 | （未细分） |
| DevOps | （未细分） |
| 企业工作流 | （未细分） |

### 3.2 论文提出的开放挑战

1. **评估超越"最终任务成功"**——needs intermediate evaluation
2. **不完整反馈下的验证**——partial / noisy signal 怎么用
3. **无回归的 harness 改进**——改 harness 不能让旧任务变差（呼应 [[skillopt]] validation gate）
4. **多 agent 间一致的共享状态**——distributed state consistency
5. **安全关键操作的人工监督**——human-in-loop 何时强制
6. **多模态环境的扩展**——vision/audio 如何融入 code-as-harness

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **"Harness Engineering" 主题确立**
   - 之前 harness 是 LangChain / AutoGen 等框架的**实现细节**
   - 这篇论文把它升级为**独立研究方向**——配合 [[aris]] 把 "harness 设计" 形式化为 design principle
   - 与 Meta-Harness (Lee et al. 2026)、AgentSPEX（W17）形成三足鼎立

2. **统一了零散工作的概念**
   - CodeAct, ToolFormer, ReAct, OpenInterpreter, [[skillopt]] 此前都在各自术语中
   - "code as substrate" 一句话把它们拉到同一个 frame

3. **42 作者 = 跨机构共识**
   - UIUC、UCLA、UIUC 的 Hanghang Tong / Jingrui He 等 LLM agent 老兵参与
   - 说明这是当前**社区共识**的归纳，而非某一团队的孤立观点

### ⚠ 局限

- **是综述，不是新方法**——没有 SOTA 数字，引用价值大于直接使用价值
- 摘要中未深入讨论 code 的**安全风险**（任意 code execution）——是不是过度乐观？
- "开放挑战 6 条" 罗列偏多，缺乏对哪些是**短期可解 / 长期开放**的优先级判别

### 🔮 揭示的趋势

1. **Agent 设计的"软件工程化"**
   - 把 git/PR/CI 这些软工范式直接搬到 multi-agent
   - 反映了 "agent 系统设计 = 分布式系统设计 + LLM 能力"的认知成熟

2. **三层架构成为 harness 设计模板**
   - 后续 harness 工作大概率沿 interface/mechanism/scaling 三层做对照

3. **Code 作为通用粘合剂**
   - 与 [[aris]] 的 Markdown skills、[[skillopt]] 的 text-space optimization 共同构成 "**文本/代码作为 agent 操作介质**" 的范式

### 📊 同方向工作链

- **前置**：CodeAct (Wang et al. 2024)、ToolFormer、OpenInterpreter
- **并行**：[[aris]]（harness 设计原则）、AgentSPEX（W17，agent 规约语言）、Meta-Harness
- **后续**：所有 2026 H2 的 harness 设计工作

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2605.18747
- **HF Papers**: https://huggingface.co/papers/2605.18747（215↑）
- **GitHub**: Awesome-Code-as-Agent-Harness-Papers（资源汇总仓库）
- **第一作者**: Xuying Ning
- **核心作者**: Pan Lu, Lingming Zhang, Tong Zhang, Hanghang Tong, Jingrui He
