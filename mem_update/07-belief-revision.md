# Belief Revision & Truth Maintenance — ReTrace 的理论原型（AGM / TMS + LLM 复活）

> **本笔记合并"经典理论 + LLM 复活"：**
> - **经典**：AGM belief revision（1985）· **JTMS/TMS**（Doyle 1979）· **ATMS**（de Kleer 1986）
> - **LLM 复活**：BeliefBank（2109.14723, EMNLP'21）· **CBM / Contextual Belief Management**（2605.30219, 2026.05, 浙大）· Struct-Searcher（2606.07689）· Bayesian-Agent（2606.08348）
> **机制**：**GATE / BRANCH**（理论层面）
> **一句话定位**：经典 AI 早就研究过"信念矛盾时怎么修订"——**TMS 的依赖网络 + 矛盾回退**正是 ReTrace"回溯到矛盾源、失效依赖链"的**直系祖先**；2026 H1 这套理论被 LLM 重新激活。

---

## 1. 这篇（合并）笔记为什么重要

**一句话**：ReTrace 的核心机制（矛盾触发回溯 + 失效依赖）不是凭空发明——它在经典 AI 里有成熟的理论原型 **belief revision（AGM）** 和 **truth maintenance system（TMS）**；把 ReTrace 挂到这条谱系上，既给它**学术正当性**，又能解释"**为什么矛盾要沿依赖链回退**"。

它对本专题和 ReTrace 的价值：

- **提供 ReTrace 的"理论外衣"**：审稿人看到"又一个 agent 记忆树"会问"这有什么理论依据？"——答案是 **TMS/AGM**。ReTrace = **TMS 在 LLM 检索场景的实例**。
- **TMS 的依赖网络 = 解级联失效的经典答案**：Doyle 的 **dependency-directed backtracking**（依赖定向回溯）正是"改了 X→沿 justification 链失效依赖 X 的 Y"——这恰好是 [[04-meme]] 的 Cascade、[[06-rome-rippleedits]] 的 ripple 都没解决的问题的**经典解法**。
- **ATMS 的多 environment = BRANCH 的经典原型**：de Kleer 的"并行维护多个 assumption-set"正是 ReTrace"保留矛盾为并行分支"的理论先驱。
- **2026 H1 这套理论正被复活**：CBM、Struct-Searcher、Bayesian-Agent 显式调用 belief revision——说明 ReTrace 踩在一个**正在升温**的点上。

---

## 2. 核心方法

### 2.1 经典理论（ReTrace 的祖先）

| 理论 | 来源 | 核心机制 | 与 ReTrace 的对应 |
| --- | --- | --- | --- |
| **AGM belief revision** | Alchourrón-Gärdenfors-Makinson 1985 | **expansion / revision / contraction** 三操作 + **最小改动**原则 + 保持一致性 | "加性更新=expansion，矛盾更新=revision/contraction"——ReTrace 的加性 vs 非加性正对应 |
| **JTMS / TMS** | Doyle 1979 | 维护 **justification（依赖）**；矛盾时 **dependency-directed backtracking** 回退culprit假设，传播 IN/OUT 标签 | 🔴 **ReTrace 回溯的直系原型**：依赖网络 + 矛盾回退 = "回溯到矛盾源 + 失效依赖链" |
| **ATMS** | de Kleer 1986 | 并行维护**多个 environment（assumption-set）**；矛盾 = "nogood" 集 | 🔴 **BRANCH 的原型**：并行保留多个信念状态 = ReTrace 保留矛盾为活分支 |

$$
\text{TMS: 矛盾检测} \to \underbrace{\text{dependency-directed backtracking}}_{\text{沿 justification 链找 culprit}} \to \text{失效 culprit + 传播 OUT 标签到依赖项}
$$

—— 这几乎是 ReTrace"回溯到出现矛盾 evidence 的节点、失效该 evidence、波及下游"的**逐字理论对应**。

### 2.2 LLM 时代的复活

| 工作 | 做法 | 机制 |
| --- | --- | --- |
| **BeliefBank**(2109.14723) | 外部**符号信念记忆** + **MaxSAT 求解器**修订冲突信念；反馈把已知信念作上下文重问 | OVERWRITE via 约束求解（TMS-in-LLM 锚点） |
| **CBM / Contextual Belief Management**(2605.30219) | 把"何时 **update / preserve / ignore** 信念"做成决策；BeliefTrack 基准；**RL + belief-state 奖励砍失败 70.9%** | GATE（belief revision 决策化） |
| **Struct-Searcher**(2606.07689) | 🔴 **search agent** 显式建在 belief-revision 上，维护演进结构图，**裁决跨模态矛盾证据** | VERSION/BRANCH（与 ReTrace 极近） |
| **Bayesian-Agent**(2606.08348) | 记忆条目=假设+**后验**，按证据更新；动作 patch/split/compress/retire/explore | GATE+patch（贝叶斯信念修订） |

### 2.3 三分类：信念该怎么处理（CBM 的可操作框架）

CBM 把 belief revision 落成三个决策（直接可用于 ReTrace 的"加性 vs 非加性"判定）：

| 决策 | 何时 | ReTrace 对应 |
| --- | --- | --- |
| **update（更新）** | 新证据更可信、旧的确实过时 | 非加性更新（回溯+改） |
| **preserve（保留）** | 旧信念仍有效，新证据存疑 | 不更新（拒绝低质新证据） |
| **ignore（忽略）** | 新证据无关/噪声 | 加性更新或丢弃 |

> 失败三类（BeliefTrack）：**Failed Stay**（该保留却改了）、**Failed Update**（该改却没改）、**Failed Isolation**（更新波及了不该波及的）——**Failed Isolation 正是级联失效的反面**（波及过度）。

---

## 3. 关键结果

| 结果 | 出处 |
| --- | --- |
| RL + belief-state 奖励**砍失败 70.9%** | CBM (2605.30219) |
| MaxSAT 信念修订提升一致性 | BeliefBank |
| Struct-Searcher 裁决跨模态矛盾（search agent） | 2606.07689 |

> **核心信号**：经典 belief revision 不是故纸堆——**CBM 用 RL 把它落地、Struct-Searcher 把它用进 search agent**。这说明 ReTrace"把 belief revision 做进检索树"是一个**有理论根基、且正在被验证**的方向。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响
1. **给 LLM 记忆更新提供理论框架**：AGM/TMS 让"矛盾时怎么改"有了公设级的参照，而非纯工程启发。
2. **依赖网络是级联失效的经典答案**：TMS 早就解决了"更新沿依赖传播"——现代记忆系统正在重新发现它。
3. **belief revision 的决策化（CBM）**：把"何时改主意"做成可学习决策，是经典理论的现代化。

### ⚠ 局限
- **ATMS 的组合爆炸**：并行维护多 environment 成本指数级——BRANCH 派必须解决的老问题（ReTrace 的分支预算/剪枝正对此）。
- **符号 TMS 难直接套 LLM**：BeliefBank 用 MaxSAT，可扩展性受限；如何在自然语言/检索场景做 TMS 是开放问题。
- CBM 等仍偏评测/小规模，落地深度待验证。

### 🔮 趋势
1. **TMS/AGM 在 LLM 复活**：belief revision 从经典 AI 回到 agent 记忆台前。
2. **依赖结构化成为解级联失效的共识方向**。

### 📊 同方向工作
- [[04-meme]]：Cascade 是 TMS 依赖回退要解的问题的现代基准。
- [[02-stale]]：CUPMem 的 propagation-aware search ≈ TMS 的依赖传播。
- [[06-rome-rippleedits]]：参数编辑的 ripple 失败 = 缺少 TMS 式依赖网络。

---

## 5. 与 ReTrace 的关系（重点 —— 这是 ReTrace 的理论支柱）

| ReTrace 机制 | 经典原型 | 用法 |
| --- | --- | --- |
| 加性 vs 非加性更新判定 | AGM 的 expansion vs revision/contraction；CBM 的 update/preserve/ignore | 🔴 直接借 CBM 三分类做判定 |
| 回溯到矛盾源节点 | **TMS 的 dependency-directed backtracking** | 🔴 ReTrace 的回溯 = TMS 依赖回退 |
| 失效依赖该 evidence 的下游 | **TMS 的 IN/OUT 标签传播** | 🔴 解级联失效（[[04-meme]]）的经典法 |
| 保留矛盾为并行分支 | **ATMS 的多 environment** | 🔴 BRANCH 的理论原型 |
| 回溯收敛/不震荡 | AGM 最小改动原则 | 约束分支预算 |

**结论**：这是 ReTrace **related work 的理论支柱**。写作建议——

1. **把 ReTrace 定位为"TMS 在 LLM 检索 agent 上的实例"**：*"ReTrace operationalizes truth-maintenance (Doyle 1979): retrieval evidence forms a justification network; on contradiction, dependency-directed backtracking invalidates the culprit evidence and propagates to dependents — instantiated over an LLM search tree with summaries."* 这一句就把"又一个树记忆"提升为"有 40 年理论根基的 belief revision 系统"。
2. **用 CBM 的三分类做你的"加性 vs 非加性"判定**（update/preserve/ignore），并引 Failed Isolation 警示"别过度波及"（级联失效的反面）。
3. **用 ATMS 解释你的 BRANCH**：保留矛盾为并行分支 = ATMS 多 environment；并主动承认 ATMS 的组合爆炸 → ReTrace 用分支预算/收敛裁决控制（这把你早先的"防震荡/防爆炸"批评接到经典理论）。
4. **Struct-Searcher / Bayesian-Agent 是必引同期工作**：它们也把 belief revision 用进 agent，但 Struct-Searcher 偏多模态图、Bayesian-Agent 偏 skill 后验——ReTrace 的差异是**检索证据树 + 矛盾回溯分支**。

---

## 6. 资源

- **AGM**（1985, J. Symbolic Logic）· **Doyle TMS**（1979, AIJ）· **de Kleer ATMS**（1986, AIJ）—— 经典，无 arXiv
- **BeliefBank** — https://arxiv.org/abs/2109.14723
- **CBM** — https://arxiv.org/abs/2605.30219（浙大 zjunlp，26↑）
- **Struct-Searcher** — https://arxiv.org/abs/2606.07689 ｜ **Bayesian-Agent** — https://arxiv.org/abs/2606.08348
- **机制标签**: GATE / BRANCH（belief revision 理论层）
