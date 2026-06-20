# EvoMem (EvoArena) — patch 化记忆记录"环境怎么演化"（你的引子）

> **arXiv**：2606.13681（2026.06）｜**机制**：**VERSION / patch**
> **HF 月榜**：2026-06，90↑｜**机构**：Salesforce / NUS / MIT
> **关键词**：patch-based memory · structured update history · evolving environment · EvoArena · chain-level
> **一句话定位**：当环境会变时，用 **patch 化的 update history** 记"状态怎么一步步变的"——但**只记演化、不解矛盾、不失效依赖项**。
> **去重说明**：本论文已在 `evolve/18-evoarena.md` 与 `memory/08-evoarena-evomem.md` 收录；本篇**换视角**——专谈 **patch 作为记忆更新机制**及其与 ReTrace 的关系，其余交叉引用。

---

## 1. 这篇论文为什么重要

**一句话**：这是**你 ReTrace 的直接引子**——它把记忆更新做成 **patch（补丁）**：环境每变一次，记一条结构化 update（像 git diff），让 agent 通过"记忆里的变化序列"推理当前环境状态，而非只存当前快照。

放在本专题（记忆更新），它代表 **VERSION 机制**的一个清晰样本，也界定了 ReTrace 要超越的起点：

- **patch = "记变化"而非"覆盖"**：保留"怎么变的"完整轨迹，是 VERSION 机制（与 OVERWRITE 相对）的典型。你的 change-log（before/after/reason/timestamp）**几乎就是 EvoMem 的 patch**。
- **但它停在"记录"**：EvoMem **不做矛盾检测、不裁决冲突、不失效依赖项**——它假设"环境演化"是已知的渐进 update，而非"两条证据打架、要判谁对"。
- **增益小（+1.5%）暴露了 patch 的天花板**：光记演化不够，必须**用演化信息做推理/裁决**——这正是 ReTrace 想补的。

---

## 2. 核心方法（聚焦 update 机制）

### 2.1 EvoArena：把环境变化建模为渐进 update 序列

| 维度 | 设计 |
| --- | --- |
| 建模 | 环境变化 = **progressive update 序列** |
| 域 | terminal / software / social |
| 难点 | 常需完成**一连串相关演化子任务**（chain-level） |

### 2.2 EvoMem：patch 化 update history

| 要素 | 做法 |
| --- | --- |
| **表示** | **patch-based memory**——记忆=结构化 **update histories** |
| **更新** | 环境每变，**追加一条 patch**（而非覆盖旧快照） |
| **使用** | agent **读 update history 序列**推理当前环境状态 |

$$
\text{环境演化}\ \{e_0\to e_1\to\dots\to e_t\}\ \xRightarrow{\text{patch 化}}\ \text{结构化 update history}\ \xRightarrow{}\ \text{据此推理当前态}
$$

**与 OVERWRITE 的区别**：不覆盖旧值，**追加变更记录**——保留演化轨迹（VERSION 机制核心）。

### 2.3 关键缺口（决定 ReTrace 的差异空间）

- ❌ **无冲突检测**：假设 update 是已知的渐进演化，不处理"两源矛盾、判谁对"。
- ❌ **不失效依赖项**：记了"X 变了"，但依赖 X 的下游不会自动失效（级联失效，见 [[04-meme]]）。
- ❌ **patch 自身膨胀**：长演化 → 大量 patch，如何检索/压缩未讨论。

---

## 3. 关键实验结果

| 设置 | 结果 |
| --- | --- |
| 当前 agent（EvoArena） | 平均 **39.6%**（动态环境普遍脆弱） |
| **EvoMem 在 EvoArena** | **仅 +1.5%** |
| EvoMem 在 GAIA / LoCoMo | +6.1% / +4.8% |
| chain-level（连续演化子任务） | +3.7% |

> **核心信号**：**+1.5%** 很小——说明"**光把演化记成 patch 还不够**"。patch 保留了信息，但 agent 没能据此正确裁决/传播。这正是 ReTrace 要补的：不只记 patch，还要在**矛盾时回溯、失效依赖**。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响
1. **把"环境演化"引入记忆与评测**：记忆从"记我看过什么"扩展到"记世界怎么变"。
2. **patch / 版本控制式表示**：用变更序列而非快照，保留演化轨迹。

### ⚠ 局限
- **+1.5% 说明远未解决**：记录有余、裁决/传播不足。
- patch history 可扩展性、冲突裁决、依赖失效全部缺位。

### 🔮 趋势
- 记忆从"快照"走向"变更日志"（与 DCPM supersedes 链、TOKI bitemporal 合流）。

### 📊 同方向工作
- [[01-memforest]]：树+局部更新（VERSION 的树版）vs EvoMem 的 patch 序列。
- [[04-meme]]：MEME 正好测 EvoMem 缺的依赖失效（Cascade）。
- `evolve/18`、`memory/08`：本论文的另两视角（自演化 / working memory）。

---

## 5. 与 ReTrace 的关系（重点）

| 维度 | EvoMem | ReTrace | 差异 |
| --- | --- | --- | --- |
| 表示 | patch / update history | 树 + 节点 change-log | ReTrace 把 patch 挂到**树结构**上 |
| 更新触发 | 环境**已知**演化 | **检测到矛盾**才非加性更新 | 🔴 ReTrace 有**冲突检测**，EvoMem 没有 |
| 冲突裁决 | ❌ 无 | ✅ 回溯 + 分支收敛 | 🔴 ReTrace 核心 |
| 依赖失效 | ❌ 无 | ✅ 沿依赖链失效 | 🔴 解 MEME 的 Cascade |

**结论**：EvoMem 是 ReTrace 的**起点而非对手**——它证明了"patch 化记忆"这条路值得走（HF 90↑、被广泛关注），但**它的 +1.5% 恰恰说明"光记不够"**。ReTrace 的定位 = **EvoMem 的 patch + 冲突检测 + 回溯/分支 + 依赖失效**。写作时:用 EvoMem 立"记忆更新该保留变更历史(VERSION)",然后指出"但 EvoMem 不检测矛盾、不失效依赖,增益仅 +1.5%——ReTrace 在 patch 之上加矛盾驱动的分支回溯与依赖级联失效"。

> 注意:你的 change-log 字段(before/after/evidence/reason/timestamp)与 EvoMem 的 structured update history 高度重合——**这部分别当卖点**(会被 EvoMem 撞),卖点在"矛盾触发的分支 + 收敛 + 依赖失效"。

---

## 6. 资源

- **arXiv**: https://arxiv.org/abs/2606.13681
- **HF Papers**: https://huggingface.co/papers/2606.13681（90↑）
- **机构**: Salesforce / NUS / MIT
- **机制标签**: VERSION / patch（结构化 update history）
- **本库其他视角**: `evolve/18-evoarena.md`（自演化）、`memory/08-evoarena-evomem.md`（working memory）
