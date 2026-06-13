# Agentic RL 专题 — HuggingFace 月榜（2025.11–2026.06）

> **数据范围**：HuggingFace Papers **月榜 top 50**，2025-11 至 2026-06（共 8 个月）
> **筛选方法**：标题/摘要/关键词**同时**命中 **Agent 信号**与 **RL 信号**（双命中），再人工剔除 world-model / VLA 机器人 / 视频图像生成 / 纯 RLVR 推理（非 agentic）/ 纯 memory-benchmark
> **收录规模**：8 个月 top50 去重 400 篇 → 双命中 126 篇 → **收录 39 篇（15 精读 ⭐ + 24 简述 📄）**

---

## 一句话定位

**Agentic RL** = 把 RL 的「信用分配 / 奖励 / 探索」机制，**适配到 agent 的多轮、长程、工具交互轨迹上**。2025 H2–2026 H1 这条线已从「能不能用 RL 训 agent」走到「**怎么把 rollout 成本、稀疏延迟奖励、多轮信用分配、技能内化做精细**」。

---

## 📚 15 篇精读（按主题分类，编号即分类顺序）

### 🔧 A. RL 算法 / 训练范式（01-04）

> 把"经验合成 / 反思 / next-state 信号 / 极简环境"做进 RL loop——解决 agentic RL 的根本约束（rollout 成本 + 稀疏奖励）。

| # | 论文 | 月/票 | 机构 | 核心贡献 |
| --- | --- | --- | --- | --- |
| **01** | [DreamGym](01-dreamgym.md) | 2025-11 / 83↑ | Meta | **经验合成代替真实 rollout**——把环境动态蒸馏成 reasoning-based experience model，纯合成交互媲美 GRPO/PPO |
| **02** | [Experiential RL (ERL)](02-experiential-rl.md) | 2026-02 / 75↑ | USC + MSR | **experience-reflection-consolidation 循环**嵌入 RL，成果内化进 base policy，部署零开销 |
| **03** | [OpenClaw-RL](03-openclaw-rl.md) | 2026-03 / 156↑ | Princeton | **next-state 信号**统一 terminal/GUI/SWE/对话——evaluative(PRM) + directive(hindsight OPD)，全异步训练 |
| **04** | [LLM-in-Sandbox](04-llm-in-sandbox.md) | 2026-01 / 87↑ | Microsoft Research | **极简 code sandbox + 非 agentic 数据 RL** 激发通用 agent，token -8× |

### 🌱 B. Agent 自演化 & 经验（05-07）

> 自动合成环境/任务 + 大规模 rollout + 失败回收的自维持循环；以及 agentic RL 在专家领域反超闭源。

| # | 论文 | 月/票 | 机构 | 核心贡献 |
| --- | --- | --- | --- | --- |
| **05** | [Agent0](05-agent0.md) | 2025-11 / 110↑ | UNC | **零数据双 agent 共进化**——出题者 vs 解题者 symbiotic competition + 工具集成 RL |
| **06** | [EvoCUA](06-evocua.md) | 2026-01 / 92↑ | 美团 | **万级异步沙箱并行 rollout + 失败回收**，OSWorld 56.7% 开源 SOTA |
| **07** | [CUDA Agent](07-cuda-agent.md) | 2026-03 / 99↑ | ByteDance Seed | **大规模 agentic RL 写 CUDA kernel**，KernelBench 超 torch.compile，L3 比 Claude/Gemini 高 ~40% |

### 🤝 C. Multi-Agent RL（08-10）

> MARL 的三种新解法——异构互学 / 测试时协作 / 宽度扩展。

| # | 论文 | 月/票 | 机构 | 核心贡献 |
| --- | --- | --- | --- | --- |
| **08** | [HACRL](08-hacrl.md) | 2026-03 / 198↑ | Beihang 等 | **异构 agent 双向互学**——训练时共享 verified rollout、推理时独立；HACPO 半算力胜 GSPO |
| **09** | [MATTRL](09-mattrl.md) | 2026-01 / 92↑ | NUS / MIT | **测试时多 agent RL**——零权重更新，结构化文本经验注入 + turn-level credit assignment |
| **10** | [WideSeek-R1](10-wideseek-r1.md) | 2026-02 / 100↑ | 清华 / RLinf | **宽度扩展**——lead-subagent MARL 并行化广度检索，4B 媲美 671B 单 agent |

### 📚 D. Deep Research / Search Agent RL（11-13）

> 长程上下文管理 + 可验证奖励 + 状态外置，是 DR 训练的三条主线。

| # | 论文 | 月/票 | 机构 | 核心贡献 |
| --- | --- | --- | --- | --- |
| **11** | [IterResearch](11-iterresearch.md) | 2025-11 / 80↑ | 阿里 Tongyi Lab | **Markovian 状态重建**——周期性重建 workspace + 演化 report 作 memory + EAPO，解决 mono-context 膨胀 |
| **12** | [DR Tulu](12-dr-tulu.md) | 2025-11 / 63↑ | AI2 / UW | **RLER 演化式 rubric**——rubric 与 policy 共进化，解决长答案不可验证；8B 媲美 OpenAI DR 便宜 1000× |
| **13** | [Harness-1](13-harness-1.md) | 2026-06 / 52↑ | UIUC / Chroma | **状态外置 harness**——环境侧 working memory，RL 只学语义决策（搜什么/留弃/验证/何时停） |

### 🎯 E. Skill RL（14-15）

> 把 skill 的发现/检索/内化统一进 RL，取代 inference-time 检索。

| # | 论文 | 月/票 | 机构 | 核心贡献 |
| --- | --- | --- | --- | --- |
| **14** | [SkillRL](14-skillrl.md) | 2026-02 / 76↑ | UNC | **递归技能增强 RL**——experience distillation 建 SkillBank + 库与 policy 共进化 |
| **15** | [SKILL0](15-skill0.md) | 2026-04 / 101↑ | 浙大 | **技能内化进参数**——in-context RL，训练时逐步撤除 skill 上下文直到 zero-shot，运行时零检索 |

---

## 🔗 24 篇简述 + 跨库交叉引用

> 详见 [`00-summary`](00-summary-2025.11-2026.06.md) 的按月列表。已在 `huggingface/` 周榜库精读的工作此处仅交叉引用：

| 跨库精读（🔗） | 本库位置 | huggingface/ 位置 |
| --- | --- | --- |
| GrandCode（Agentic GRPO） | 00-summary D类参照 | `huggingface/01-grandcode.md` |
| SDAR（OPSD multi-turn 修复） | 00-summary A类 | `huggingface/08-sdar.md` |
| GLM-5（异步 Agent RL） | 00-summary F类 | `huggingface/06-glm-5.md` |
| Kimi K2.5（Agent Swarm） | 00-summary F类 | `huggingface/13-kimi-k25.md` |
| Youtu-Agent（Training-free GRPO） | 00-summary F类 | `huggingface/05-youtu-agent.md` |
| MiroThinker H1（验证嵌入推理） | 00-summary D类 | `huggingface/12-mirothinker.md` |
| OpenSearch-VL / GrepSeek / Skill1 | 00-summary D/E类 | `huggingface/00-summary` |

---

## 目录结构

```
agentic-rl/
├── README.md                              # 本文件 — 总索引（按分类）
├── 00-summary-2025.11-2026.06.md          # 8 个月 top50 命中汇总（按月）+ 趋势
│
│ ── A. RL 算法 / 训练范式 ──
├── 01-dreamgym.md                         # 经验合成代替真实 rollout (Meta)
├── 02-experiential-rl.md                  # 反思-内化循环嵌入 RL (MSR)
├── 03-openclaw-rl.md                      # next-state 信号统一 agent RL (Princeton)
├── 04-llm-in-sandbox.md                   # 极简 sandbox + 非 agentic 数据 RL (MSR)
│
│ ── B. Agent 自演化 & 经验 ──
├── 05-agent0.md                           # 零数据双 agent 共进化 (UNC)
├── 06-evocua.md                           # 万级异步沙箱 + 失败回收 (美团)
├── 07-cuda-agent.md                       # agentic RL 写 CUDA kernel (字节 Seed)
│
│ ── C. Multi-Agent RL ──
├── 08-hacrl.md                            # 异构 agent 双向互学 (Beihang)
├── 09-mattrl.md                           # 测试时多 agent RL (NUS/MIT)
├── 10-wideseek-r1.md                      # 宽度扩展 MARL (清华/RLinf)
│
│ ── D. Deep Research / Search Agent RL ──
├── 11-iterresearch.md                     # Markovian 状态重建 (阿里 Tongyi)
├── 12-dr-tulu.md                          # 演化式 rubric 奖励 (AI2/UW)
├── 13-harness-1.md                        # 状态外置 harness (UIUC/Chroma)
│
│ ── E. Skill RL ──
├── 14-skillrl.md                          # 递归技能增强 RL (UNC)
└── 15-skill0.md                           # 技能内化进参数 (浙大)
```

---

## 阅读顺序建议

### 🎯 最短路径（4 篇抓住主线）

1. **[DreamGym](01-dreamgym.md)** — 看 agentic RL 的第一性约束（rollout 成本）与"经验合成"解法
2. **[OpenClaw-RL](03-openclaw-rl.md)** — 看"奖励信号丰富化"（next-state / directive）
3. **[HACRL](08-hacrl.md)** — 看 multi-agent RL 的工程化（异构互学）
4. **[CUDA Agent](07-cuda-agent.md)** — 看 agentic RL 在专家领域反超闭源

### 📚 按主题串读

- **降本两路**：DreamGym（合成经验）↔ EvoCUA（规模化真实 rollout + 失败回收）
- **奖励丰富化三切面**：OpenClaw-RL（next-state）· ERL（反思）· DR Tulu（演化 rubric）
- **MARL 三解法**：HACRL（异构）· MATTRL（测试时）· WideSeek-R1（宽度）
- **环境职责切分两极**：LLM-in-Sandbox（环境做薄）↔ Harness-1（环境做厚、状态外置）
- **Skill × RL 两条路**：SkillRL（外部库共进化）↔ SKILL0（内化进参数）
- **自演化系列**：Agent0（能力共进化）→ SkillRL（技能库共进化）→ MetaClaw（部署中演化，见 summary）

---

## 与本仓库其他专题的关系

- **`huggingface/`**（周榜全景）：本专题是其"Agentic RL"主线的**月榜深挖版**——周榜每周 top10 命中较少，月榜 top50 能捞到更多专门的 RL 算法/训练工作。交叉引用见上表。
- **`deep-research/`**（DR 月榜）：D 类（IterResearch / DR Tulu / Harness-1）与之高度交叉——本专题侧重"DR 的 **RL 训练**"，DR 专题侧重"DR 系统全貌"。
- **`evolve/`**（自演化月榜）：B 类（Agent0 / EvoCUA）与之交叉——本专题侧重"自演化的 **RL 机制**"。
- **`tongyi-deepresearch/`**：IterResearch 即通义 DeepResearch 系工作。

---

## 报告结构说明

每篇精读包含 5 个标准章节（与 `huggingface/` 库一致）：

1. **这篇论文为什么重要** — 一句话核心 + 学界影响背景
2. **核心方法** — 关键技术贡献，配 mermaid 图/表
3. **关键实验结果** — 重要数字、ablation
4. **对领域的影响 / 后续方向** — 影响、局限、趋势、并行工作
5. **资源** — arXiv / GitHub / 作者
