# Agents' Last Exam — Agent 时代的"经济价值未饱和" Benchmark

> **arXiv**：2606.05405（2026.06）｜**机构**：UC Berkeley RDI（Responsible Decentralized Intelligence）+ 250+ 行业专家
> **HF 周榜**：W23 / Jun 7-13，#2，155↑
> **关键词**：Economically Meaningful Tasks · O*NET / SOC 2018 · 1000+ Verifiable Tasks · 2.6% Pass Rate · Living Benchmark
> **GitHub**：rdi-berkeley/agents-last-exam | **官网**：agents-last-exam.org

---

## 1. 这篇论文为什么重要

**一句话**：Agents' Last Exam (ALE) 是 **首个聚焦"经济价值真实部署" 的 agent benchmark**——与 250+ 行业专家合作，按美国职业分类 O\*NET / SOC 2018 设计 1000+ 真实工作任务，主流 harness × backbone 平均完整通过率仅 **2.6%**。

为什么这是 benchmark 设计的重大进步：
- **HLE (Humanity's Last Exam)** = 学术难题难度的极限
- **ALE (Agents' Last Exam)** = **经济价值现实** 的极限
- → 不再问 "模型能不能答对 PhD 物理题"，而问 "**能不能替代真实白领工作**"

**核心论点**：
> "Recent AI systems have achieved strong results on a wide range of benchmarks, yet these gains have not translated into economically meaningful deployment"

→ benchmark 进步 ≠ 经济价值。ALE 把评估转移到**真正能 ROI 化**的维度。

---

## 2. 评测设计

### 2.1 任务分类法（基于 O*NET / SOC 2018）

**O\*NET / SOC 2018**：美国劳工部维护的官方职业分类标准，覆盖几乎所有非物理行业。

```
ALE 任务体系:
├── 13 个行业集群（industry clusters）
│   ├── 法律 / 金融 / 医疗 / 教育 / 媒体 / IT / ...
├── 55 个子领域（subdomains）
│   ├── 每个集群下细分 3-5 个子领域
└── 1000+ 任务（tasks）
    ├── 每个子领域 ~20 个具体任务
    └── 每个任务都设计为可自动验证
```

**关键设计原则**：
1. **长时程**（long-horizon）：单任务跨越多步骤、多工具
2. **经济价值高**：直接映射到真实工作的具体子任务
3. **可验证结果**：尽管真实，但有 deterministic / semi-deterministic 验证

### 2.2 250+ 行业专家命题

- 不是 AI 研究者闭门造车
- 真实领域专家提供任务规约
- 任务标注**真实工作的 ROI / 时间消耗** 等元信息

### 2.3 "Living Benchmark" 持续扩展

- 不是一次性发布固定 set
- 随时新增任务、淘汰已饱和任务
- 类似 GitHub issue tracker 的运营模式

---

## 3. 关键实验结果

### 3.1 震惊性主结果

> "across mainstream harness and backbone configurations, the average **full pass rate is 2.6%**"

| 配置维度 | 通过率 |
|---------|------:|
| **主流 harness × backbone 平均** | **2.6%** |

**对照**：
- HLE 上 Gemini 3 已经 37%+
- GAIA 上 frontier agent 70%+
- **ALE 上几乎全军覆没**

→ "最难的层级远未饱和" —— 给业界一个清晰信号：**距离真实经济价值还有几个数量级**。

### 3.2 与 HLE / ClawBench 的三角对照

| Benchmark | 难度来源 | 顶级表现 |
|-----------|---------|---------:|
| HLE | 学术难题 | ~37% (Gemini 3) |
| **ALE** | **经济价值任务** | **2.6%** |
| [[clawbench]] | 真实日常 web | 33.3% (Claude 4.6) |

→ ALE 是**最难** 的 agent benchmark 之一，揭示了 evaluation–reality gap 的具体大小。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **"Economic Value" 成为 benchmark 新维度**
   - HLE = academic ceiling
   - ALE = **economic ceiling**
   - 后续 benchmark 设计将必须回答 "这个分数对应多大经济价值"

2. **基于职业分类法（O\*NET / SOC）的系统化覆盖**
   - 不再 random 选 task
   - 按真实职业体系**全谱覆盖**——保证没有 cherry-pick
   - 类似 MMLU 引入学科分类法的影响

3. **UC Berkeley RDI 的影响力背书**
   - Dawn Song 通讯——AI safety + AI economics 圈的重量级人物
   - 308+ 作者 + 250+ 专家 = **跨学界 + 产业的共识**

4. **2.6% 这个数字的传播价值**
   - 极具新闻冲击力
   - 给政策制定者一个具体的"AI 还远着呢" 参考点

### ⚠ 局限

- **任务可验证性**：真实工作很多不可单元测试（如"写一份说服力强的法律意见书"）——这部分如何评估？摘要未深入披露
- **2.6% 的样本量**：1000 任务规模相对大，但 13 集群 × 55 子领域 后单格仍偏小
- **依赖美国职业分类**：跨文化适配性？中国市场的对应职业能否套用？
- **专家成本**：250+ 专家的成本极高，难以社区复现 / 扩展
- **Living benchmark 的版本控制**：怎么保证不同月份的得分可比？

### 🔮 揭示的趋势

1. **Agent 评估从 "能力测试" 走向 "替代性测试"**
   - 不是 "AI 能做什么"
   - 而是 "**AI 能替代什么职业的什么子任务**"

2. **"AI 经济价值未饱和" 的实证**
   - 业界宣传与实际部署的差距首次被量化
   - 影响投资人、政策制定者、企业 IT 决策

3. **Benchmark 与劳动经济学的交叉**
   - O\*NET / SOC 这种**劳动经济学标准**进入 AI benchmark 设计
   - 类似 AI safety 借鉴公共政策的演化路径

### 📊 同方向工作

- HLE (Humanity's Last Exam)：学术难题 ceiling，ALE 的命名致敬
- [[clawbench]]（W15）：日常 web 真实任务（互补，难度低但真实）
- W24 ResearchClawBench：端到端科研 benchmark（专业领域版）
- W24 SWE-Explore（上交）：repo 探索 benchmark（编程域）
- GDPval-AA（[[glm-5]] 论文引用）：另一种经济价值评估

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2606.05405
- **HF Papers**: https://huggingface.co/papers/2606.05405（155↑）
- **GitHub**: https://github.com/rdi-berkeley/agents-last-exam
- **官网**: https://agents-last-exam.org
- **机构**: UC Berkeley RDI（Responsible Decentralized Intelligence）
- **第一作者**: Yiyou Sun | **通讯**: **Dawn Song**
- **作者团队**: 308+ 人（主要作者列表后还有 208 位未列出）
- **领域专家**: 250+
