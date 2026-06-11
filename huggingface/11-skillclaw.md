# SkillClaw — 让 Skill 在多 Agent 生态里集体进化

> **arXiv**：2604.08377（2026.04）｜**机构**：8 作者（机构未明确披露），通讯 Yuxiang Ji
> **HF 周榜**：W15 / Apr 12-18，#4，293↑
> **关键词**：Collective Skill Evolution · Autonomous Evolver · Multi-user Trajectories · Skill Refine + Extend
> **基准 / 模型**：WildClawBench + Qwen3-Max

---

## 1. 这篇论文为什么重要

**一句话**：SkillClaw 把"**单 agent 自演化 skill**"升级为"**多 user 生态级 skill 群体进化**"——把跨用户、跨时间的交互轨迹作为改进 skill 的**主要信号**，让一个用户发现的 skill 提升能**自动同步给所有用户**。

为什么这是个工业级的真问题：
- 部署在生产的 LLM agent（如 OpenClaw、Claude Code）服务**百万用户**
- 不同用户**重复发现相同 workflow、相同 tool 用法、相同失败模式**
- 但 skill 部署后基本**静态**——系统从不利用这些集体经验改进
- SkillClaw 把这部分**集体经验**变成可挖掘的训练信号——典型工业产品级思维

类似工作：
- [[skillopt]]（W22）= **单用户/单任务的 text-space 优化**
- SkillClaw（W15）= **多用户的集体 skill 进化**

两者互补、不冲突。

⚠ 注意 SkillClaw 与 [[skillsbench]] 的**潜在张力**：SkillsBench 实证发现"self-generated skill 平均无收益"，这对 SkillClaw 的核心假设（让 agent 自己发现 skill 提升）是个挑战。SkillClaw 的关键防御是**"跨用户多次观察"**——单用户的 self-generation 不可靠，但 N 个用户独立产生同一 pattern 时信号很强。

---

## 2. 核心方法

### 2.1 三层 pipeline

```
┌───────────────────────────────────────┐
│ Layer 1: 轨迹聚合（Trajectory Aggregation）│
│   持续收集所有用户的交互 trace          │
└───────────────────────────────────────┘
                  ↓
┌───────────────────────────────────────┐
│ Layer 2: Autonomous Evolver           │
│   识别**重复出现**的行为模式            │
│   (workflow / tool use / failure)     │
└───────────────────────────────────────┘
                  ↓
┌───────────────────────────────────────┐
│ Layer 3: 技能更新机制                  │
│   • Refine: 优化已有 skill              │
│   • Extend: 扩展新 skill                │
│   → 写入共享 skill 仓库                  │
│   → 同步给所有用户                       │
└───────────────────────────────────────┘
```

### 2.2 与 [[skillopt]] 的核心差异

| 维度 | [[skillopt]] | SkillClaw |
|------|----------|-----------|
| **优化主体** | 单 task / 单用户 | 整个用户生态 |
| **信号来源** | optimizer model 的 reflection | 多用户 trajectory 聚合 |
| **更新粒度** | 单文档编辑 | refine + extend 全 skill 库 |
| **更新频率** | epoch | 持续在线 |
| **关键风险** | 需要强 optimizer model | 需要跨用户去重 / 隐私 |

### 2.3 "跨用户重复观察 = 信号" 的核心 insight

> **原文核心表述**："treats cross-user and over-time interactions as the primary signal for improving skills"

这条 insight 解决了 [[skillsbench]] 的 "self-generation 不可靠" 问题：
- **单用户**单次发现：可能是噪声
- **N 个用户**独立发现同一 pattern：高置信度的真实需求
- → Evolver 只把**多用户共识 pattern** 转化为 skill 更新

---

## 3. 关键实验

> 摘要明确标注 "Work in progress"，**未给出具体数字**。

| 实验维度 | 摘要披露 |
|---------|---------|
| **基准** | WildClawBench |
| **模型** | Qwen3-Max |
| **定性结论** | "significantly improves performance in real agentic scenarios under limited interactions and feedback" |

**待补**：
- skill 数量增长曲线（refine vs extend 占比）
- 跨用户**收敛速度**（多少次 trajectory 后 skill 稳定）
- 与 [[skillsbench]] 上的横向对比

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **Skill 的"产品化"思维**
   - 之前 skill 工作都是研究环境（单用户、单 benchmark）
   - SkillClaw 把 skill 当作**产品 feature**——多用户、持续在线、共享更新
   - 这是 OpenClaw 等生产 agent 落地必经路线

2. **挑战 [[skillsbench]] 的"self-generation 无效"结论**
   - SkillsBench：单 user 的 self-generation 平均无收益
   - SkillClaw：**多 user 共识** 把噪声平均掉，self-evolution 可行
   - 两者实际上是**互补**——SkillsBench 揭示了单点的局限，SkillClaw 给出生态级的解

3. **集体智能（collective intelligence）的 LLM 时代实现**
   - 类似 Stack Overflow 的"集体经验"机制——但**全自动 + 实时**
   - 与 Wikipedia / 开源社区的演化机制有相似哲学

### ⚠ 局限

- **隐私 / 数据合规**——跨用户聚合 trajectory 涉及 GDPR / 中国个保法等合规问题（论文未触及）
- **"显著提升"无数字**——work in progress 状态，未来版本应补完整 ablation
- **Autonomous Evolver 的具体实现未披露**——是 prompt-based、聚类、还是 RL trained？
- **skill 退化（regression）防护**——多用户共识可能引导错误方向，需要 validation gate（类似 [[skillopt]] 的设计）

### 🔮 揭示的趋势

1. **Agent 从"工具"走向"生态"**
   - 多用户共享 → 集体进化 → 网络效应
   - 这是 agent 产品的 long-term moat

2. **"Skill as a Service"**
   - skill 库作为可订阅 / 可贡献的资源池
   - 用户既是消费者也是贡献者

3. **Trajectory 的二次价值**
   - 之前 trajectory 是训练数据
   - 现在 trajectory 是**持续在线 skill 改进**的信号
   - 类似 RLHF 把用户反馈作为持续训练信号的逻辑

### 📊 同方向工作

- [[skillopt]]（W22）：单 task 的 text-space 优化，与 SkillClaw 形成"单任务 vs 生态"对照
- [[skillsbench]]（W08）：skill 评测标杆，对 SkillClaw 是验证场
- W19 Skill1：skill-augmented agent 的 RL 统一进化
- W23 COLLEAGUE.SKILL：专家蒸馏生成 skill（人工版的 Evolver）
- W24 LatentSkill：把 in-context skill 内化为 latent skill（与 SkillClaw 的 in-context 共享路线分叉）

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2604.08377
- **HF Papers**: https://huggingface.co/papers/2604.08377（293↑）
- **状态**: Work in progress
- **第一作者**: Ziyu Ma | **通讯**: Yuxiang Ji
- **生态绑定**: OpenClaw（W11 OpenClaw-RL、W14 ClawKeeper 同生态）
