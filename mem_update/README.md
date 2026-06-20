# Agent 记忆更新（非增量 / 冲突 / 矛盾）专题 — 2026.01~2026.06

> **专题定位**：聚焦 **memory update**，尤其**非增量更新**——新信息与已存记忆**冲突 / 矛盾 / 使其过时（stale）**时怎么办：**覆盖？门控保留？版本化？保留为分支？**
> **数据范围**：HF 月榜（daily-papers API 绕过截断）+ arXiv，**2026-01 ~ 2026-06**，补必引经典 prior art。
> **与其他专题的区别**：`memory/` 讲 working memory 的表示/更新/使用（上下文增长）；本专题把「**更新**」一维放大，专攻最难的**矛盾更新**。重叠论文（δ-mem、Masking、EvoMem、MemSkill）仅交叉引用。
> **目录定位**：`Q:/awesome-agent-paper/mem_update/`
> **机制标签**：**OVERWRITE（覆盖）/ GATE（门控）/ VERSION（版本patch）/ BRANCH（分支）**

> ⚠️ **数据可靠性**：所有 2026 arXiv id 经 `arxiv.org/abs/<id>` 三方核对、未发现编造；2026 近未来日期致部分 WebFetch reader "无法确认"（cutoff 假象）。正式引用前建议本人抽查 PDF。

---

## 📚 8 篇精读（9 文件）— 按机制组织

> **核心主线**：**覆盖正被证伪 → 转向门控 → 级联失效未解 → BRANCH 空地**。

### 🔴 与 ReTrace 思路最近（优先读）

| # | 论文 | 机制 | 月/票 | 与 ReTrace 的关系 |
|---|---|---|---|---|
| **01** ⭐ | **[MemForest](01-memforest.md)** | VERSION/树 | 05 / 17↑ | **结构最像**：时间有序树+局部 per-node 更新；但无矛盾分支 → ReTrace 必 diff |
| **02** ⭐ | **[STALE](02-stale.md)** | GATE | 05 / 46↑ | **问题最像**：Implicit Conflict（无显式否定的失效）best 仅 55.2%；矛盾判定别用二值 |
| **04** ⭐ | **[MEME](04-meme.md)** | benchmark | 05 / 7↑ | **靶心评测**：级联失效 Cascade ~3%；ReTrace 在此提分=杀手锏 |
| **07** ⭐ | **[Belief Revision & TMS](07-belief-revision.md)** | GATE/BRANCH | — / — | **理论支柱**：TMS 依赖回退=ReTrace 回溯的直系祖先；ATMS 多 env=BRANCH 原型 |

### 🟢 GATE：覆盖正被证伪，转向门控

| # | 论文 | 机制 | 月/票 | 核心 |
|---|---|---|---|---|
| **03** ⭐ | **[Useful Memories Become Faulty](03-useful-memories-faulty.md)** | GATE>OVERWRITE | 05 / 19↑ | **覆盖有害实证**：保留原始翻倍 → "失活不删"的最强背书 |

### 🟡 VERSION：记变更历史（patch / 版本）

| # | 论文 | 机制 | 月/票 | 核心 |
|---|---|---|---|---|
| **05** ⭐ | **[EvoMem](05-evomem.md)** | VERSION/patch | 06 / 90↑ | **你的引子**：patch 记演化，但不解矛盾/不失效依赖（仅 +1.5%） |

### 🔴 OVERWRITE：参数级编辑（主流但栽在级联失效）

| # | 论文 | 机制 | 月/票 | 核心 |
|---|---|---|---|---|
| **06** ⭐ | **[ROME/MEMIT + RippleEdits/MQuAKE](06-rome-rippleedits.md)** | OVERWRITE | 经典+26新 | 主流编辑骨架 + **级联失效根源**（"hopping-too-late"） |

### 🔵 冲突裁决（检测+仲裁该信谁）

| # | 论文 | 机制 | 月/票 | 核心 |
|---|---|---|---|---|
| **08** ⭐ | **[知识冲突一线](08-knowledge-conflict.md)** | GATE | 综述+26新 | 三类冲突分类 + **TRACK：冲突污染推理**（ReTrace 第二收益的依据） |

---

## 🗺 四机制地图（crowded vs underexplored）

| 机制 \ 载体 | 参数 | 外部 flat 记忆 | 外部**结构化（树/图）** |
|---|---|---|---|
| **OVERWRITE** | 🔴 极卷（ROME/MEMIT/MOSE/AlphaEdit…） | 🟡 FluxMem(图) | 🟡 InKH(写入时失效) |
| **GATE** | 🟡 SERAC/GRACE | 🟢 **正热**（STALE/CBM/2605.12978/EvolveMem/TCR） | ⚪ 少 |
| **VERSION（patch/版本）** | — | 🟡 EvoMem(patch) | 🟡 MemForest(树)·RoMem(相位)·DCPM(supersedes 链) |
| **BRANCH（矛盾保留为活分支+收敛）** | 🟡 WISE(side-mem 分片) | ⚪ 近邻 TOKI(audit 行) | ⚪⚪ **几乎空 ← ReTrace** |

> **三条结论**（详见 [00-summary 第〇/三节](00-summary-2026.01-2026.06.md)）：
> 1. **覆盖（overwrite）正被证伪**：2605.12978 实证 consolidation 覆盖有害 → 全领域转 GATE。
> 2. **级联失效（cascade）是公认未解难题**：参数侧 RippleEdits/MQuAKE ⊕ 记忆侧 MEME（~3%）同一个洞。
> 3. **BRANCH 几乎空**：保留矛盾为并行活分支+收敛裁决，最近邻 TOKI/DCPM/ATMS 但都非此 → **ReTrace 的位置**。

---

## 📂 目录结构

```
mem_update/
├── README.md                          # 本文件 — 索引 + 四机制地图
├── 00-summary-2026.01-2026.06.md      # 6 个月命中论文（4 机制分类 ~40 篇）+ 经典 prior art + 趋势
│
│ ── 与 ReTrace 最近 ──
├── 01-memforest.md                    # 时间有序树 + 局部更新（结构最像）
├── 02-stale.md                        # Implicit Conflict + CUPMem（问题最像）
├── 04-meme.md                         # 级联失效基准 Cascade ~3%（靶心评测）
├── 07-belief-revision.md              # AGM/TMS/ATMS + CBM（理论支柱）
│
│ ── GATE ──
├── 03-useful-memories-faulty.md       # 覆盖有害实证（失活不删的背书）
│
│ ── VERSION ──
├── 05-evomem.md                       # patch 化 update history（引子）
│
│ ── OVERWRITE ──
├── 06-rome-rippleedits.md             # 参数编辑骨架 + 级联失效根源
│
│ ── 冲突裁决 ──
└── 08-knowledge-conflict.md           # 三类冲突 + TRACK 冲突污染推理
```

---

## 🎯 阅读顺序

### 最短路径（4 篇抓住主线）
1. **[00-summary 第〇节](00-summary-2026.01-2026.06.md)** — 四机制框架 + 三条结论
2. **[03 Useful Memories Become Faulty](03-useful-memories-faulty.md)** — 为什么覆盖错了（转向 GATE）
3. **[04 MEME](04-meme.md)** — 级联失效：全领域谷底（~3%）
4. **[07 Belief Revision & TMS](07-belief-revision.md)** — 经典理论怎么解（依赖回退 + 分支）

### 为 ReTrace 服务的路径
- **结构对照**：[01 MemForest](01-memforest.md)（树+局部更新，必 diff）
- **判定难点**：[02 STALE](02-stale.md)（矛盾判定 55.2%，别二值）→ [08 知识冲突](08-knowledge-conflict.md)（仲裁信号库）
- **理论外衣**：[07 TMS/AGM](07-belief-revision.md)（ReTrace = TMS 的 LLM 检索实例）
- **靶心评测**：[04 MEME](04-meme.md)（打 Cascade）+ [05 EvoMem](05-evomem.md)（起点对照）
- **主流对照**：[06 ROME/RippleEdits](06-rome-rippleedits.md)（参数 OVERWRITE 的级联失效，反衬外部记忆路线）

### 按机制串读
- **OVERWRITE → GATE 的转向**：[06](06-rome-rippleedits.md)（参数覆盖天花板）→ [03](03-useful-memories-faulty.md)（覆盖有害）→ [02](02-stale.md)（门控判失效）
- **VERSION 谱系**：[05 EvoMem](05-evomem.md)（patch）→ [01 MemForest](01-memforest.md)（树）→ RoMem/DCPM/TOKI（见 00-summary B 类）
- **冲突裁决 + 理论**：[08 知识冲突](08-knowledge-conflict.md)（检测+仲裁）→ [07 belief revision](07-belief-revision.md)（决策化+理论）

---

## 📈 关键趋势速览

> 完整见 [00-summary 第三节](00-summary-2026.01-2026.06.md)。

1. **OVERWRITE → GATE 转向**：从"直接覆盖/巩固"到"先判该不该更新、保留原始证据"（2605.12978 实证覆盖有害）。
2. **记忆侧 ⊕ 参数侧合流**：知识编辑（参数 OVERWRITE）与 agent 记忆（外部 VERSION/GATE）**栽在同一个级联失效**。
3. **"不删"成共识**：RoMem 相位降级、TOKI audit 行、DCPM supersedes 链、SERAC/GRACE base 冻结。
4. **评测从"检索"转"真更新"**：MEME/MINTEval/AgingBench/SubtleMemory 专测 contradiction/cascade/revision-aging。
5. **belief revision 被 LLM 复活**：CBM/Struct-Searcher/Bayesian-Agent/BeliefBank 显式调用——TMS/AGM 回台前。

**仍待解决**：① 级联失效（dependent invalidation）；② 矛盾判定本身难（STALE 55.2%）；③ **BRANCH 空地**；④ 覆盖 vs 保留的膨胀权衡；⑤ update 评测碎片化。

---

## 📝 格式说明

每篇精读 5 章节（为什么重要 / 核心方法[表+公式] / 关键结果[未披露标"需读 PDF"] / 影响·局限·趋势·同方向 / 资源）+ **「与 ReTrace 的关系」**专节（点明占四机制哪格、ReTrace 差异点）。

> **数据来源**：HF daily-papers API + arXiv（2026-01~06）+ 经典 prior art；所有 2026 id 经 arXiv 核对。与 `memory/`、`evolve/`、`deep-research/` 去重，重叠仅交叉引用。
