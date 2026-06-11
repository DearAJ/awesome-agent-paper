# VideoDR — 首个开网络视频深度研究基准

> **论文标题**：Watching, Reasoning and Searching: A Video Deep Research Benchmark on Open Web for Agentic Video Reasoning
> **arXiv**：2601.06943（2026.01）｜**机构**：兰大 / HKUST(GZ) / PKU / QuantaAlpha 等
> **HF 周榜**：W03 #1，214↑
> **关键词**：Video Deep Research · Multi-hop Reasoning · Workflow vs Agentic · Open-web Benchmark
> **GitHub**：https://github.com/QuantaAlpha/VideoDR-Benchmark

---

## 1. 这篇论文为什么重要

**一句话**：VideoDR 是**首个**「video deep research」基准——视频提供**局部视觉锚点**，开放网络提供**可验证答案**，模型必须做**多帧关联 + 多轮搜索 + 多跳推理**。

这填补了两个已有评测体系的空白：
- **现有 deep research benchmark**（BrowseComp、WebWalker）：**只接受文本 query**，视觉只是辅助
- **现有 video benchmark**（Video-MME、MVBench、LVBench）：**闭合证据**，问题答案都在视频内

**真实场景**：用户拿一段视频问"这个博物馆最近的特展叫什么"——答案不在视频里，得**先识别博物馆，再查官网，再多跳找答案**。这正是 VideoDR 要评估的。

**关键发现（反直觉）**：**Agentic 范式不一定优于 Workflow**。Gemini-3 在 Workflow 上 69%，Agentic 上 76%；但 MiniCPM-V 4.5 在 Workflow 上 25%，Agentic 上 **只剩 16%**。原因：弱模型一旦从视频提取的初始 anchors 漂移，后续 web 搜索会放大错误。

---

## 2. 任务定义

### 形式化

给定视频 $V$、自然语言问题 $Q$、浏览器搜索工具 $S$，模型输出答案 $A$：
$$f: (V, Q; S) \rightarrow A$$

### 任务硬约束（数据集构造时）

1. **多帧推理**：答案必须依赖**多个时间点**的视觉线索（一帧 screenshot 不够）
2. **多跳推理**：视频感知 + 外部搜索必须**至少交互一轮**

### 两个 ablation 测试（严格质量门）

- **Web-only Test**：只给文本 $Q$ + 网络搜索 → 如果能答对 → 视频无依赖 → 丢弃
- **Video-only Test**：只看视频 → 如果能答对 → 退化为纯视频理解 → 丢弃

**只有两个测试都失败的样本才保留**。

---

## 3. 数据集构造

### 流程
1. **候选视频池**：跨平台采样 + 三维分层（来源 / 领域 / 时长）
2. **严格负向过滤**：排除单场景 / 公共话题 / 无可验证证据链
3. **初步筛选**：保留有 **prominent visual anchors** 的视频
4. **问题设计**：标注者必须设计满足多帧 + 多跳约束的问题，并归档证据网页
5. **5 人盲测**：5 个独立标注者作答，统计正确率作为人类难度分级

### 数据集统计（100 个样本）

| 维度 | 分布 |
|------|------|
| **领域** | Daily Life 33%, Economics 16%, Tech 15%, Culture 15%, History 11%, Geography 10% |
| **问题长度** | 平均 25.54 tokens，95-分位 54 tokens |
| **视频时长** | Short ≤ N min / Medium / Long（长尾分布） |
| **难度** | Low (4-5/5 人对) 32, Mid (2-3/5) 36, High (0-1/5) 32 |
| **人类上界** | 平均 50.4%（High 难度只有 10.6%） |

---

## 4. 关键实验结果

### 4.1 主榜（Workflow vs Agentic）

| 模型 | Workflow Avg | Agentic Avg | Δ |
|------|------------:|------------:|---:|
| **Gemini-3-pro-preview** | 69 | **76** | +7 |
| **GPT-5.2** | 69 | 69 | 0 |
| GPT-4o | 42 | 43 | +1 |
| Qwen3-Omni-30B-A3B | 37 | 37 | 0 |
| InternVL3.5-14B | 27 | 30 | +3 |
| **MiniCPM-V 4.5** | 25 | **16** | **-9** ⚠️ |
| Human | 50.4 | 50.4 | — |

**反直觉关键发现**：MiniCPM-V 在 Agentic 模式下**反而退化 9 分**！

### 4.2 难度分层（高难度下的鲜明对比）

| 模型 | Low (W/A) | Mid (W/A) | High (W/A) |
|------|----------|----------|-----------|
| Gemini-3 | 90.6/93.8 | 61.1/69.4 | 56.3/65.6 ⬆ |
| GPT-5.2 | 84.4/84.4 | 61.1/58.3 | 62.5/65.6 |
| GPT-4o | 50.0/62.5 | 30.6/38.9 | 46.9/**28.1** ⬇ |

**关键洞察**：Agentic 增益**取决于模型能否在长搜索链中维持初始视觉 anchors**。
- 强模型（Gemini）：能用 anchors → Agentic 增益
- 中等模型（GPT-4o）：High 难度下 anchors 漂移 → Agentic 反而退化

### 4.3 视频时长分层

- **长视频**（>10min）：弱模型在 Agentic 下大幅退化（Qwen3-Omni 50%→20%，MiniCPM 30%→10%）
- **强模型**（Gemini）反而在 Long 上 +20 分 → **能保持 anchors 的模型受益于直接看视频**

### 4.4 错误分析（5 类错误类型）

| 错误类型 | 主导地位 |
|---------|---------|
| **Categorical Error**（视觉感知偏差导致归类错）| 所有模型都最多 |
| Numerical Error | 即使 Gemini 也无法解决 → 当前 MLLM 的持续短板 |
| Reasoning Error | 极少（0-1 例）→ **不是推理问题，是感知 + 搜索协同问题** |

### 4.5 工具使用统计

- 工具调用次数**不与准确率正相关**
- Gemini-3：think/search 2.89/2.52 → 76% 准确率
- Qwen3-Omni：think/search 1.80/1.21 → 37%（多搜索 ≠ 更准）
- MiniCPM-V：Agentic 用更多工具 + 跑更快，但准确率反而降 9 分

---

## 5. 对领域的影响 / 后续方向

### 🌟 学界影响

1. **首个挑战「Agentic 一定更好」的实证基准**
   - 业界长期默认 Agentic > Workflow，VideoDR 用数据证明这是模型能力依赖的
   - 给 deep research 系统设计者一个明确警钟：**弱模型用 Workflow，强模型才用 Agentic**

2. **「Goal Drift + Long-horizon Consistency」被确立为新瓶颈**
   - 不是推理能力不够，而是初始视觉 anchor 在长搜索链中漂移
   - 给后续 follow-up（如 W12 MiroThinker）指明方向

3. **多模态 Deep Research 范式定型**
   - 视频 → 视觉 anchor 提取 → query 生成 → web 检索 → 多跳验证 → 答案
   - 这个 pipeline 被 W06 Vision-DeepResearch、W12 MiroThinker、W19 OpenSearch-VL 等沿用

### ⚠ 局限

- **仅 100 个样本**：作为开创性基准合理，但统计稳定性受限
- 仅评测 6 个模型，未覆盖 Claude 系列
- 标注者的搜索路径主观性会影响"标准答案"的稳定性

### 🔮 揭示的趋势

1. **Deep Research 走向多模态**：文本-only DR 已经"卷"得差不多，视频 / 图像 / 音频 DR 是新蓝海
2. **Agentic ≠ 万能**：业界需要根据模型能力选择 paradigm，而不是无脑上 agentic
3. **评测标准从「准确率」转向「过程」**：Goal Drift、tool-use 效率等过程指标成为新焦点

### 📊 相关 / 后续工作
- W06 Vision-DeepResearch（在 MLLM 中激励 DR 能力）
- W12 MiroThinker H1（heavy-duty research agents via verification）
- W13 OpenResearcher（开源 DR trajectory 合成）
- W21 AutoResearchClaw（自演化 DR + 人机协作）

---

## 6. 资源

- **arXiv**: https://arxiv.org/abs/2601.06943
- **HF Papers**: https://huggingface.co/papers/2601.06943
- **GitHub**: https://github.com/QuantaAlpha/VideoDR-Benchmark
- **机构**: 兰州大学（lead） / HKUST(GZ) / UBC / 复旦 / PKU / USC / NUS / UCAS / QuantaAlpha
- **通信作者**: pengh@lzu.edu.cn, chenronghao@alumni.pku.edu.cn, wanghuacan17@mails.ucas.ac.cn
