# Awesome Agent Paper

> 个人维护的 **AI Agent / Deep Research / Agentic RL** 论文阅读笔记库。
>
> 目标：跟踪 2025 H2 – 2026 H1 LLM Agent 领域的关键工作，按"主题 + 系列"两种方式组织，做到**可检索、可串读、可复用**。

---

## 仓库结构

```
awesome-agent-paper/
├── README.md                      # 本文件，总索引
│
├── huggingface/                   # HuggingFace Daily Papers 周榜精选（按主题分类）
│   ├── README.md                  # 20 篇精读 + 10 大趋势观察
│   ├── 00-summary-2026-W01-W24.md # 24 周 top10 命中论文汇总
│   └── 01~20-*.md                 # 20 篇精读笔记
│
└── tongyi-deepresearch/           # 通义 DeepResearch 系列（按时间演化）
    ├── README.md                  # 11 篇系列论文笔记 + 演化脉络
    └── papers/                    # 原始 PDF
```

---

## 两条子线索

### 1. [huggingface/](./huggingface/) — 横向：2026 H1 全景

**数据范围**：HuggingFace Papers 周榜 2026-W01 ~ W24（共 24 周）

**筛选方法**：每周 top 10 中标题/摘要明确涉及 Agent / Deep Research / Agentic RL / Tool Use 的论文

**组织方式**：按主题分 7 类，共 20 篇精读

| 主题 | 篇数 | 代表论文 |
|---|---|---|
| Agentic RL & 训练算法 | 4 | GrandCode, SDAR, DelTA, Weak-Driven |
| 数据 / Skill 合成 | 3 | TermiGen, SkillOpt, SkillClaw |
| Deep Research | 3 | VideoDR, ARIS, MiroThinker |
| 旗舰模型 & Agent 框架 | 3 | GLM-5, Youtu-Agent, Kimi K2.5 |
| Multi-Agent & World Model | 3 | Gamma-World, Agentic World Modeling, Recursive MAS |
| Agent Benchmarks | 3 | SkillsBench, ClawBench, Agents' Last Exam |
| Harness Engineering | 1 | Code as Agent Harness |

→ 详见 [huggingface/README.md](./huggingface/README.md)

### 2. [tongyi-deepresearch/](./tongyi-deepresearch/) — 纵向：通义 DR 系列演化

**对象**：阿里通义实验室开源的 Web Agent / Deep Research 系列（对标 OpenAI Deep Research）

**配套模型**：Tongyi-DeepResearch-30B-A3B（MoE，30B 总参 / 3B 激活）

**11 篇系列论文**，按时间演化串读：

```
WebWalker → WebDancer → WebSailor → WebShaper → WebWatcher
   → WebResearcher → ReSum → WebWeaver → WebSailor-V2
   → AgentFounder（CPT）→ AgentScaler（环境扩展）
```

主线：**数据合成 → Agentic CPT → Agentic SFT → Agentic RL** 端到端方法学

→ 详见 [tongyi-deepresearch/README.md](./tongyi-deepresearch/README.md)

---

## 阅读建议

### 想快速了解 2026 H1 Agent 领域全景
→ 直接看 [huggingface/README.md](./huggingface/README.md) 的"最短路径 4 篇"

### 想理解一个完整的开源 Deep Research 体系如何搭建
→ 按时间顺序通读 [tongyi-deepresearch/README.md](./tongyi-deepresearch/README.md)

### 想找特定主题
- **RL 训练算法**：huggingface/01, 08, 09, 10
- **数据合成**：huggingface/02, 03, 11 + tongyi WebShaper / AgentFounder
- **Deep Research**：huggingface/04, 07, 12 + 整个 tongyi 系列
- **旗舰模型**：huggingface/05, 06, 13
- **World Model**：huggingface/14, 15, 16
- **Benchmarks**：huggingface/17, 18, 19
- **Harness Engineering**：huggingface/20 + 07

---

## 笔记格式约定

每篇精读笔记统一 5 个章节：

1. **为什么重要** — 一句话核心 + 学界影响
2. **核心方法** — 关键技术贡献（配图/公式/伪代码）
3. **关键实验结果** — 重要数字、ablation
4. **领域影响 / 后续方向** — 学界冲击、局限、并行工作
5. **资源** — arXiv / GitHub / 作者

---

## License

笔记内容仅供学习交流，原始论文版权归各作者所有。
