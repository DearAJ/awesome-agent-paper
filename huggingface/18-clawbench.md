# ClawBench — 在真实生产网站上测 AI Agent 的日常能力

> **arXiv**：2604.08523（2026.04）｜**机构**：NAIL-Group (Natural and Artificial Intelligence Lab)
> **HF 周榜**：W15 / Apr 12-18，#5，263↑
> **关键词**：Real Production Websites · Side-effect Interception · Everyday Tasks · Frontier Model Limitations
> **项目页**：claw-bench.com

---

## 1. 这篇论文为什么重要

**一句话**：ClawBench 是 **首个跑在 144 个真实生产网站**上的 agent benchmark——153 个简单日常任务，**Claude Sonnet 4.6 仅 33.3% 完成率**，证明 frontier 模型在"真实环境的简单任务"上仍有巨大缺口。

为什么这个 benchmark 设计独特：
- WebArena / VisualWebArena 等都是 **复刻**的离线 sandbox
- ClawBench 在**真实生产站点**上测——保留了真实网站的复杂性、动态性、反爬虫、A/B 测试
- 但又解决了"如何在不污染真实系统的情况下测"的问题——**轻量 interception 拦截最终提交**

**关键 contribution**：把"真实环境 + 不产生副作用" 这两个互相矛盾的需求**工程上调和**。

---

## 2. 评测设计

### 2.1 任务规模

| 维度 | 数值 |
|------|------|
| **任务数** | 153 |
| **真实平台数** | **144** |
| **任务类别** | 15 类（购物 / 预约 / 求职申请 / 查询 / 注册 / 退订 / ...） |
| **任务难度** | "简单"——模拟普通用户日常 |

→ **"简单"** 这个定位非常关键——证明 frontier 模型连日常任务都做不利索，否则的话 ClawBench 价值就贬值了。结果 Claude 4.6 只有 33.3% 印证了这个设计哲学。

### 2.2 关键技术创新：轻量 Interception Layer

```
Agent → Browser → ⚠ Interception Layer ⚠ → Real Website
                       │
                       └─ 只拦截最终提交（下单/付款/发送）
                          其他所有操作（浏览/搜索/选项）都真实发生
```

**为什么这是个聪明的设计**：
- ✅ 评估保留了**真实网站的复杂性**（导航、搜索、动态加载）
- ✅ 不真正下单 → **不会有真实金钱/服务副作用**
- ✅ 不需要构造离线副本 → **永远是最新版本的网站**
- ❌ 必须 case-by-case 标注"最终提交" 是什么——人工成本

**与离线 sandbox 的对比**：

| 维度 | WebArena (sandbox) | ClawBench (real + intercept) |
|------|---------------------|------------------------------|
| 真实性 | 静态 / 旧 | 动态 / 最新 |
| 复杂性 | 简化 | 完整反爬虫、A/B、推荐 |
| 副作用风险 | 无 | 无（intercept）|
| 维护成本 | 网站升级时需更新 | 自动跟随网站变化 |
| 可复现性 | 高 | 低（网站每天变） |

### 2.3 评估的核心能力

任务设计要求 agent 同时具备：
- 从用户文档中**提取信息**
- 跨平台**多步骤工作流导航**
- 高强度**写操作**（填写复杂表单）

---

## 3. 关键实验结果

### 3.1 主结果（震惊数字）

| 模型 | 完成率 |
|------|------:|
| **Claude Sonnet 4.6** | **33.3%** |
| 其他 6 个 frontier 模型 | 摘要未披露具体数字 |

**论文核心结论**：
> "both proprietary and open-source models can complete only a small portion of these tasks"

→ 即使是 frontier 模型，**真实日常 web 任务的 2/3 都做不完**。

### 3.2 与已有 benchmark 的对照

| Benchmark | 顶级模型完成率 |
|-----------|---------------:|
| WebArena | 50%+ |
| GAIA | [[youtu-agent]] 72.8% |
| **ClawBench** | **33.3%** ⚠ |

→ ClawBench 显著更难，**揭示了离线 benchmark 高估了 agent 的真实能力**。

### 3.3 评测的 7 个 frontier 模型

包含 proprietary + open-source（具体名单未在摘要披露）。

---

## 4. 对领域的影响 / 后续方向

### 🌟 学界 / 工业影响

1. **"离线 benchmark 高估真实能力" 的实证**
   - WebArena/GAIA 等离线榜上模型已经 50-70%
   - ClawBench 真实站点上只有 33%
   - → 业界长期 over-claim agent 能力，ClawBench 给出 reality check

2. **Interception 机制的工程价值**
   - 解决了 "真实环境 + 不污染" 的两难
   - 后续真实环境 benchmark 大概率借鉴这套设计

3. **"日常任务"作为 benchmark 主题的回归**
   - 之前 benchmark 都追求 "**难、专业、PhD-level**"（如 HLE、GPQA）
   - ClawBench 反其道行之——**简单但真实**
   - 与 [[agents-last-exam]] 形成有意思的对照：**专家级难** vs **日常级真实**

### ⚠ 局限

- **不可复现性**：真实网站每天变，分数会漂移
- **interception 必须 per-task 配置**——153 个任务全部人工标注 final submission point，规模化困难
- **地理 / 账号偏差**：测试者必须在特定 region 用特定账号
- 摘要只披露 Claude 4.6 一个数字，缺少完整 leaderboard
- 部分 "真实网站" 可能在测试中变化或下线 → 长期可维护性问题

### 🔮 揭示的趋势

1. **Agent benchmark 走向"真实化"**
   - 离线 sandbox 阶段任务已完成
   - 下阶段是真实生产环境的鲁棒性测试
   - Mind2Web → ClawBench → ? 

2. **"简单任务" benchmark 的价值回归**
   - 在 frontier 模型 PhD-level 任务做得越来越好的时代
   - 揭示**简单任务的失败模式**反而更有诊断价值

3. **真实环境 benchmark 的两条路**
   - ClawBench：真实 + intercept（保护副作用）
   - [[agents-last-exam]]：人类专家命题（保护难度）
   - 共同回答 "AI agent 离真实部署还有多远"

### 📊 同方向工作

- [[agents-last-exam]]（W24，UC Berkeley）：另一种 "agent 真实价值" benchmark，互补
- WebArena / VisualWebArena：离线 web agent benchmark
- W11 OpenClaw-RL：OpenClaw 生态（与 SkillClaw、ClawKeeper 同生态）
- W16 ClawGUI：浙大 GUI agent 框架
- W14 ClawKeeper：OpenClaw 安全防护
- W04 ABC-Bench：agent 后端编码 benchmark
- W09 MobilityBench：路径规划 benchmark

---

## 5. 资源

- **arXiv**: https://arxiv.org/abs/2604.08523
- **HF Papers**: https://huggingface.co/papers/2604.08523（263↑）
- **项目页**: claw-bench.com
- **机构**: NAIL-Group (Natural and Artificial Intelligence Lab)
- **第一作者**: Yuxuan Zhang | **通讯**: Kelsey R. Allen
- **作者团队**: 21 人，含 Wenhu Chen、Dongfu Jiang、Yipeng Zhu 等
