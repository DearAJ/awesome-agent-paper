# Useful Memories Become Faulty When Continuously Updated by LLMs —— 覆盖有害的实证

> **arXiv**：2605.12978（2026.05）｜**机制**：**GATE > OVERWRITE**（实证论证）
> **HF 月榜**：2026-05，19↑｜**机构**：UIUC（Hao Peng 组）
> **关键词**：memory consolidation harm · overwrite degrades · keep raw episodes · ARC-AGI re-failure · gate consolidation
> **一句话定位**：用实验证明 **LLM 持续 consolidation/覆盖记忆反而让记忆变坏**——效用掉到**无记忆基线以下**，**保留原始 episode 反而把准确率翻倍**。

---

## 1. 这篇论文为什么重要

**一句话**：很多记忆系统默认"持续把新信息 consolidate/覆盖进记忆是好事"；这篇**正面否定**了这个假设——**LLM 驱动的持续记忆更新（覆盖式）会让本来有用的记忆退化**，甚至不如完全不用记忆。

它是本专题**最有冲击力的"负面结果"论文**，也是 ReTrace 的**论点护城河**：

- **一句话级别的反直觉结论**：**"持续更新让有用记忆变坏。"** 这把"记忆更新越多越好"的朴素假设打穿。
- **给"别覆盖、要门控"提供硬证据**：保留原始 episode（不 consolidate）反而**翻倍**——直接支持"GATE（门控该不该更新）优于 OVERWRITE（无脑覆盖）"。
- **它就是你早先被建议的"删→失活"的领域级背书**：你 ReTrace 若坚持"矛盾时**失活但保留**旧 evidence、不物理删除"，这篇是你 motivation 段落的**核心引用**。

它与 [[02-stale]]（何时该判失效）、[[07-belief-revision]]（何时该改主意）同属 **GATE** 机制族，三者共同构成"2026 H1 从覆盖转向门控"的主线（见 `00-summary` 第三节趋势 1）。

---

## 2. 核心方法

### 2.1 诊断：持续 consolidation 是 update 原语，但它有害

| 做法 | 假设 | 实测后果 |
| --- | --- | --- |
| **LLM 持续 consolidation/覆盖记忆** | 新信息融进记忆 = 记忆更好 | **记忆退化**，效用**低于无记忆基线** |
| **保留原始 episode（不 consolidate）** | 占空间、看似低效 | 准确率**翻倍** |

> 核心论点：**"Useful memories become faulty when continuously updated by LLMs."** —— 覆盖/巩固这一 update 原语本身会引入退化。

### 2.2 关键证据

| 证据 | 数字 |
| --- | --- |
| GPT-5.4 把**已解决的 ARC-AGI** 重新做错 | **54%** |
| 持续 consolidation vs 无记忆基线 | 效用**低于**无记忆 |
| 保留原始 episode vs 强制 consolidation | 准确率**翻倍** |

**直觉**：每次 LLM 覆盖/改写记忆，都引入一次有损、可能出错的转换；这些误差**累积**，最终把"本来对的"记忆改坏——比根本不动它还差。

### 2.3 主张：门控 consolidation、别覆盖原始证据

论文的设计含义（GATE 机制）：

$$
\text{新信息} \to \underbrace{\text{先判：要不要 consolidate?}}_{\text{GATE}} \begin{cases} \text{否} \to \textbf{保留原始 episode（不覆盖）} \\ \text{是} \to \text{谨慎巩固} \end{cases}
$$

—— **保留原始证据**为默认，consolidation 是需要门控的、有代价的操作，而非无脑常态。

> 摘要未披露：门控的具体判据、在哪些任务上"翻倍"、consolidation 退化的机制分析全貌——**需读 PDF**。

---

## 3. 关键实验结果

| 设置 | 结果 | 说明 |
| --- | --- | --- |
| 持续 LLM consolidation | 效用 **< 无记忆基线** | 覆盖式更新**反而有害** |
| GPT-5.4 / ARC-AGI | 已解决题**重做错 54%** | 覆盖把对的改坏 |
| 保留原始 episode | 准确率 **翻倍** | "不覆盖"显著更优 |

> **核心信号**：这是本专题里**论证力最强的一条**——不是"某方法好一点点"，而是**"主流的覆盖式更新方向本身错了"**。它把"记忆更新"从"怎么覆盖得更好"重新框定为"**要不要覆盖**"。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **给覆盖式记忆更新祛魅**：实证"持续 consolidation 有害"，迫使领域从 OVERWRITE 转向 GATE——这是 2026 H1 的范式转向证据。
2. **"保留原始证据"成为默认**：与 RoMem（相位降级不删）、TOKI（audit 行保留）、SERAC（base 冻结）共同指向"**别物理删除/覆盖**"。
3. **把"记忆膨胀"和"记忆退化"权衡摆上台面**：保留原始 episode 翻倍但占空间——这正是 VERSION/保留派必须回答的成本问题。

### ⚠ 局限

- **保留一切 → 记忆膨胀**：翻倍的代价是不 consolidate、记忆无限增长，可扩展性未解（这恰是 ReTrace 必须用剪枝/失活机制回答的）。
- **没给"安全 consolidation"的正面方案**：证明覆盖有害，但"什么时候 consolidate 才安全"留作开放。
- 主要在 ARC-AGI 等任务上验证，跨任务普适性需更多证据。

### 🔮 趋势

1. **GATE 取代 OVERWRITE** 成为记忆更新默认姿态。
2. **保留 + 按需检索 > 提前 consolidate**（与 GAM 的"JIT 记忆"、IterResearch 的"无损保管"同向）。

### 📊 同方向工作

- [[02-stale]]：何时判失效（GATE 的"何时"）—— 与本篇"要不要覆盖"互补。
- [[07-belief-revision]]：CBM 何时 update/preserve/ignore —— preserve 正是本篇主张。
- **LM Need Sleep**（2606.03979）：**反例对照** —— Sleep 把记忆 consolidate 进权重（OVERWRITE），本篇恰恰警告这条路的退化风险。**两篇放一起读张力最大。**

---

## 5. 与 ReTrace 的关系（重点）

| ReTrace 的设计点 | 本篇提供的支持 |
| --- | --- |
| **矛盾时"失活但保留"旧 evidence，不物理删除** | 🔴 **最强外部证据**：保留原始 episode 翻倍、覆盖有害 → ReTrace 的"软删除"方向被实证背书 |
| change-log 保留 before/after（不丢旧值） | 同上——保留历史优于覆盖 |
| summary 有损但 evidence 无损保留 | 呼应"保留原始证据"优于"持续 consolidate" |

**结论**：这篇是 ReTrace **motivation 段落的核心引用**。早先我给你的批评"别删 evidence、要失活"，在这里得到**实证背书**：覆盖/consolidation 会累积有损转换、把对的记忆改坏，而**保留原始证据反而翻倍**。

但它也给 ReTrace **提了一个必须回答的问题**：保留一切会**记忆膨胀**。所以 ReTrace 不能只"保留",必须配**剪枝/失活 + 按需渲染**(对应 Harness-1 的 budget rendering)——即"**逻辑上保留(可追溯)、物理上受控(不爆炸)**"。写作时:用本篇立"为什么不覆盖",用你的剪枝机制答"那怎么不膨胀"。

---

## 6. 资源

- **arXiv**: https://arxiv.org/abs/2605.12978
- **HF Papers**: https://huggingface.co/papers/2605.12978（19↑）
- **机构**: UIUC（Hao Peng 组）
- **机制标签**: GATE > OVERWRITE（实证：门控 consolidation、保留原始证据）
- **对照阅读**: LM Need Sleep（2606.03979，覆盖式巩固的反例）
