# Tree-Memory 专题 — Search Agent Memory × Tree × Evolution 调研

> **专题定位**：一份**交叉点调研**——回答「**search agent 的记忆 + 树结构 + 进化**结合，有没有相关工作？空白在哪？」
> **与其它专题的区别**：`memory/` 聚焦工作记忆、`evolve/` 聚焦自演化/skill、`deep-research/` 聚焦 search/DR、`agentic-rl/` 聚焦 agentic RL。本专题是**横切三者的交叉视角**，所有重叠论文**只交叉引用、不重复精读**。
> **数据范围**：本仓库四个 00-summary（2025-11 ~ 2026-06 月榜）+ HuggingFace 2026-06 月榜**全部 105 篇**核对 + Arbor 全文（arXiv 2606.11926）。
> **目录定位**：`Q:/awesome-agent-paper/tree-memory/`

---

## 一句话结论

**三者（树 + 记忆 + 进化）真正合一的只有 [Arbor](https://arxiv.org/abs/2606.11926)**——hypothesis tree 即记忆、四个 refinement 操作即进化——**但它做的是自主科研 / 代码优化（MLE-Bench），不是检索式 search/deep research**。而检索式 search 这边（Harness-1 / GAM / SearchSwarm）记忆又是**扁平的、不进化**。

> **空白 = ①检索式 search + ②树状工作记忆 + ③搜索中演化**：让"证据-子问题"组成一棵会**展开分支（变异）、按证据剪枝（选择）、把证据上卷成结论（经验泛化）、只采纳已核实 claim（适应度门控）**的演化树。**目前无人做。**

---

## 三主题维恩图（速览）

```
                    ② TREE 树（MCTS · ToT 背景）
                            │
        🟠 树+记忆          🔴 三者合一          🟡 树+进化
        GAM 2511.18423      Arbor 2606.11926     PSN 2601.03509
        IterResearch        OneManCompany        CORAL 2604.01658
        2511.07327          2604.22446
   ─────────────────────────┼─────────────────────────────────
   ① SEARCH-AGENT MEMORY    🟢 记忆+进化         ③ EVOLUTION 进化
     Harness-1 2606.02373   Harness-1 · MIA      SkillRL/Skill1/MemSkill
     SearchSwarm 2606.09730 EvoArena 2606.13681  EvoMem/SCOPE/OpenSkill
     MIA 2604.04503         SkillRL 2602.08234   Need-Sleep 2606.03979
```

- **🔴 正中只有 Arbor + OneManCompany**——但都**不做检索式 search**。
- **① 记忆圈里做检索的（Harness-1/SearchSwarm/GAM/MIA/IterResearch）全部不在 ② 树圈核心**——记忆是扁平环境对象/文件系统/演进报告/page-store。
- **SearchSwarm 的委派树（单层）+ Harness-1 的 evidence links** 是最接近"检索+类树+记忆"的两块拼图，但都缺"持久演化的树"。

---

## 目录结构

```
tree-memory/
├── README.md                              # 本文件 — 索引 + 结论速览
├── 00-survey-tree-memory-evolution.md     # 主调研笔记（三维定义 + 维恩图 + 逐篇梳理 + 空白区 + 机会）
└── 01-proposal-phylosearch.md             # 研究点提案：PhyloSearch（演化证据树即工作记忆的检索式 DR agent）
```

---

## 阅读顺序

1. **先看本 README** 的"一句话结论"与维恩图 → 30 秒抓住"空白在哪"。
2. **读 [00-survey](00-survey-tree-memory-evolution.md) 第二节的 Arbor 段** → 看目前最接近的参照怎么把"树=记忆、四操作=进化"做出来（节点 schema `⟨h, ι, μ⟩` + 六步 coordinator + 四 refinement 操作）。
3. **读 00-survey 第四节"空白区与机会"** → 看缺的那块（检索式 + 树记忆 + 演化）+ 6 个可探索的设计角度。
4. **读 [01-proposal-phylosearch](01-proposal-phylosearch.md)** → 把空白点展开成**可投稿的研究提案**：节点=自包含 Markovian 状态（压缩 summary + 最新检索）、四个生物进化算子、实验设计 + 6 个消融 + 新颖性定位。
5. **想深入某篇** → 按 00-survey 第五节的交叉引用表跳到 `memory/` `evolve/` `deep-research/` `agentic-rl/` 的对应精读。

---

## 核心论文一览

| 论文 | arXiv | 交叉类型 | 一句话 |
| --- | --- | --- | --- |
| **Arbor** | [2606.11926](https://arxiv.org/abs/2606.11926) | 🔴 树+记忆+进化 | hypothesis tree 即语义记忆，四操作（写回/上卷/剪枝/门控）即进化——但做优化不做检索 |
| **OneManCompany** | [2604.22446](https://arxiv.org/abs/2604.22446) | 🔴 树+记忆+进化 | E²R 树搜索 + Talent 组织记忆 + 招募重配进化（企业式编排，非检索） |
| **Harness-1** | [2606.02373](https://arxiv.org/abs/2606.02373) | 🟢 记忆+进化（最接近检索式） | 环境侧 working memory（evidence links）+ RL，但记忆扁平非树 |
| **SearchSwarm** | [2606.09730](https://arxiv.org/abs/2606.09730) | 🟢 记忆+进化（最接近类树） | 委派=单层树、子 agent 返回 summary，但不是持久演化树 |
| **GAM** | [2511.18423](https://arxiv.org/abs/2511.18423) | 🟠 树+记忆 | JIT page-store 层级保管 + 运行时重建，无进化 |
| **PSN** | [2601.03509](https://arxiv.org/abs/2601.03509) | 🟡 树+进化 | 可执行符号程序网络随经验进化，是 skill 网不是检索记忆树 |
| **EvoArena/EvoMem** | [2606.13681](https://arxiv.org/abs/2606.13681) | 🟢 记忆+进化 | patch 式演化记忆，扁平非树 |

> 完整 13 篇表 + 与生物进化树（phylogenetic tree / coalescent）的概念呼应见 [00-survey 第五节](00-survey-tree-memory-evolution.md#五资源与交叉引用表)。
