# AI Native Builder Consultant Skills

> 专为**构建** AI Native 产品的人设计的顾问级 Skills 库——不是教你用 AI 做事，而是教你把 AI 做成产品。

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)
[![作者](https://img.shields.io/badge/作者-Medoc%20May-teal?style=flat-square)](https://github.com/MedocMay/ai-native-builder-consultant-skills)

---

## 从这里开始

这个仓库不是单纯的 skills 集合，而是一套 **AI Native 咨询工作系统**。

它提供三层东西：

- `Skills`：用来做关键产品决策
- `Templates`：把决策沉淀成可交付文档
- `Workflow` 文档：把一次完整咨询从定位、设计、上线到迭代串起来

第一次进入仓库，建议按这个顺序看：

1. 先读 [CONSULTING-WORKFLOW.md](CONSULTING-WORKFLOW.md)
2. 再看 [TERMINOLOGY.md](TERMINOLOGY.md)
3. 然后从 `agent-boundary-design` 开始
4. 用 [`templates/`](templates/) 目录把输出沉淀成正式交付物

### 核心交付物

一次完整咨询通常会产出以下大部分文档：

- `AI 产品定位声明`
- `Agent Boundary Spec`
- `SOUL.md`
- `HITL Design Spec`
- `知识架构决策`
- `信任-自治校准计划`
- `数据基础计划`
- `Eval 计划 + 每周 Production Review 议程`
- `技术栈决策`
- `Launch Risk Review`
- `迭代飞轮设计`

---

## 为什么要有这个仓库

GitHub 上有很好的 Skills 库，教你用 AI 做 PM 工作。  
有很好的 Skills 库，教你用 AI 写代码。

但没有任何一个库，系统性地回答这些问题：

- 我的 Agent 边界应该划在哪里？什么该它做，什么该人做？
- 用 RAG 还是 Fine-tuning？依据是什么？
- 为什么我的 Agent 在 demo 里很好，到真实用户手里就崩？
- 怎么把「信任」设计进 AI 产品，而不是最后才想起来加？
- AI Native 的 UX，到底和套一个聊天框有什么本质区别？

这些问题，不在任何一本书里。只能靠构建、上线、出错、再构建来回答。

**这个库，是那个过程的蒸馏。**

---

## 适合谁用

**你是对的人，如果：**

- 你已经决定要做一个 AI Native 产品，正在面临真实的架构决策
- 你是亲手做产品的技术创始人或技术 PM，不是把 AI 工作外包给「AI 团队」的人
- 你要的是决策框架，不是通用建议
- 你在用 Claude Code、OpenClaw 或类似的 Agent 开发环境

**你不是对的人，如果：**

- 你在找日常任务的 Prompting 技巧
- 你想要填空式模板
- 你还没决定要做什么——这个库假设你已经在做了

---

## 库的结构

所有 Skills 分三层，每一层都建立在上一层的基础上：

### 第一层 — 心智模型（Mental Models）
*做任何决策之前，需要先建立的基础概念。*

| Skill | 它回答的问题 |
|-------|------------|
| `ai-native-vs-ai-enhanced` | 我的产品是真正的 AI Native，还是只是加了个聊天框？ |
| `agent-loop-model` | Agent 在机械层面到底在做什么？ |
| `llm-capability-boundaries` | 我能把什么交给模型，什么不能？ |
| `uncertainty-and-hallucination` | 为什么 Agent 会「说谎」，怎么处理？ |

### 第二层 — 设计决策（Design Skills）
*决定你的产品能不能做成的那些选择。*

| Skill | 它回答的问题 |
|-------|------------|
| **`agent-boundary-design`** ← 从这里开始 | 我的 Agent 边界在哪？该自主做什么，该交给人做什么？ |
| `system-prompt-engineering` | 怎么写一个在生产环境撑得住的 System Prompt？ |
| `human-in-the-loop-design` | 怎么设计人机确认的交互体验，让用户越用越信任？ |
| `knowledge-injection-strategy` | RAG、Fine-tuning、Context Engineering 三选一，怎么决策？ |
| `ai-native-ux-patterns` | AI Native 的 UX 有哪些模式？什么阶段用哪个？ |
| `trust-and-autonomy-tradeoff` | 给 Agent 多少自主权？信任如何随时间校准？ |
| `tech-stack-selection` | 框架、模型、基础设施怎么选？ |

### 第三层 — 构建技艺（Build Skills）
*让 Agent 在生产环境真正好用的工程判断。*

| Skill | 它回答的问题 |
|-------|------------|
| `data-foundation` | 数据从哪里来？评估集怎么建？标注怎么治理？— `agent-eval-framework` 和飞轮的前置条件 |
| `agent-eval-framework` | 怎么知道我的 Agent 到底有没有在好好工作？ |
| `multi-agent-orchestration` | 多 Agent 系统怎么设计才不会在生产里出问题？ |
| `launch-risk-review` | 上线前要检查哪些东西？ |
| `iteration-flywheel` | 怎么建立数据→模型→产品的正向迭代飞轮？ |

---

## 快速开始

这个仓库以**可移植 Skills 内容库**的形式发布，可以用三种方式使用：

### 1. 单独使用某个 Skill

打开任意 `SKILL.md`，把内容复制进你的 AI 工作流作为上下文。

### 2. 导入到本地 skills 目录

如果你的 Agent 环境支持本地 skills 文件夹，可以把对应集合下的 `skills/` 内容复制进去。

```bash
# 示例：复制某一组 skills 到本地目录
cp -R agent-design/skills/* /path/to/your/local/skills/
```

### 3. 作为产品设计参考仓库使用

按顺序使用这些 Skills，把每一步输出沉淀成产品团队的决策文档。

如果你想先看项目级方法，先读 [CONSULTING-WORKFLOW.md](CONSULTING-WORKFLOW.md)。  
如果你想统一交付物命名，见 [TERMINOLOGY.md](TERMINOLOGY.md)。  
如果你想直接套用交付模板，打开 [`templates/`](templates/) 目录。

---

## 推荐路径

如果你想用最短路径跑一遍完整方法，建议按这个顺序：

```text
ai-native-vs-ai-enhanced
  → agent-boundary-design
  → system-prompt-engineering
  → human-in-the-loop-design
  → knowledge-injection-strategy
  → data-foundation
  → agent-eval-framework
  → tech-stack-selection
  → launch-risk-review
  → iteration-flywheel
```

`multi-agent-orchestration` 只在系统复杂度真的需要时启用。

---

## 如何使用这些 Skills

在支持本地或导入 skills 的工具里，相关 Skills 可能会自动加载。也可以显式调用：

```
# 中文也可以
请用 agent-boundary-design skill 帮我设计这个文档处理 agent 的边界。
```

**Skills 设计为链式使用。** 完整的从零到上线链路：

```
ai-native-vs-ai-enhanced      ← 确认你在做什么
  → agent-loop-model          ← 理解机制
    → agent-boundary-design   ← 定边界
      → system-prompt-engineering  ← 写宪法
        → human-in-the-loop-design ← 设计人机界面
          → agent-eval-framework   ← 量化是否 work
```

**Skills 是对话式的，不是文档式的。** 每个 Skill 会先问你几个关于你的具体产品、用户和约束条件的问题，然后给出针对你的情况的建议。通用建议没有价值——这些 Skills 的设计目标就是给出具体答案。

---

## 这个库和其他库有什么不同

### 知识来源不同

大多数 Skills 库的知识来自书籍和方法论。

这个库的知识来自生产事故。

每一个 Skill 都植根于至少一个真实的产品决策和真实的后果：

- 多渠道 Agent 平台——连接飞书、钉钉、微信、Telegram、Discord 的路由与编排架构
- 垂直行业 AI SaaS 产品——System Prompt 从第一版到生产版的完整迭代历程
- 多个客户项目：教育数据 Agent、数字人系统、企业 SaaS 试点

当一个 Skill 说「这种方法在生产里会出问题」，是因为它真的出过问题。当它说「这个模式能建立用户信任」，是因为我们量过。

**这不是从互联网上汇总的最佳实践，是我们自己交了学费换来的判断。**

### 立场不同

这个库站在**咨询师**的立场，不是工具书的立场。

工具书告诉你「有哪些选项」。顾问告诉你「在你的情况下，你应该选哪个，为什么，以及如果你选错了会发生什么」。

每个 Skill 都被设计为做后者。

### 三条指导原则

**1. 决策优先，解释在后。**
每个 Skill 从一个你需要做的决策出发。框架存在是为了帮你做这个决策，不是为了教你这个话题。

**2. 具体胜于通用。**
Skills 会问你的产品、你的用户、你的约束条件。输出是针对你的情况的建议，不是你还需要自己翻译的通用框架。

**3. 生产是唯一的检验标准。**
Demo 会说谎。Skills 的校准基准是生产行为——什么真的会出问题，什么真的能建立信任，什么真的能扩展。如果一个模式在 demo 里好看但在生产里崩，我们会明说。

---

## 中文本土化说明

现有所有主流 Skills 库都是英文的，且都是西方 AI 产品语境下的产物。

中国 AI 创业者面临独特的挑战，在英文资源里找不到答案：

- **平台生态**：飞书、钉钉、微信的 IM 嵌入式 Agent 与 Slack/Teams 有不同的设计约束
- **合规要求**：ICP 备案、数据本地化、等级保护对 Agent 架构的影响
- **用户信任曲线**：中国职场用户对 AI 产品的信任建立路径有其特点
- **本土基础设施**：阿里云、腾讯云、百度 AI 的选型逻辑与 AWS/GCP 不同

这些本土化考量会融入相关 Skills 的决策节点，而不是机械翻译英文内容。

---

## 仓库地图

- `mental-models/`：基础认知和能力边界
- `agent-design/`：最核心的 Agent 与产品设计决策
- `build-skills/`：验证、上线和迭代
- `templates/`：可直接套用的咨询交付模板
- `CONSULTING-WORKFLOW.md`：项目级咨询流程
- `TERMINOLOGY.md`：术语和命名标准

---

## 当前状态

**第一层 — 心智模型（`mental-models` plugin）**
- [x] `ai-native-vs-ai-enhanced` — 完成
- [x] `agent-loop-model` — 完成
- [x] `llm-capability-boundaries` — 完成
- [x] `uncertainty-and-hallucination` — 完成

**第二层 — 设计决策（`agent-design` plugin）**
- [x] `agent-boundary-design` — 完成
- [x] `system-prompt-engineering` — 完成
- [x] `human-in-the-loop-design` — 完成
- [x] `knowledge-injection-strategy` — 完成
- [x] `ai-native-ux-patterns` — 完成
- [x] `tech-stack-selection` — 完成
- [x] `trust-and-autonomy-tradeoff` — 完成

**第三层 — 构建技艺（`build-skills` plugin）**
- [x] `data-foundation` — 完成
- [x] `agent-eval-framework` — 完成
- [x] `multi-agent-orchestration` — 完成
- [x] `launch-risk-review` — 完成
- [x] `iteration-flywheel` — 完成

**原则：** 每个 Skill 产生一个具名的、可签字的咨询交付物，不是知识摘要。实际咨询会按决策链推进：

```
定位诊断：
ai-native-vs-ai-enhanced     → AI 产品定位声明

Agent 设计：
agent-boundary-design        → Agent Boundary Spec  ← 从这里开始
system-prompt-engineering    → SOUL.md
human-in-the-loop-design     → HITL Design Spec
knowledge-injection-strategy → 知识架构决策
trust-and-autonomy-tradeoff  → 信任-自治校准计划

数据与 Eval 基础：
data-foundation              → 数据基础计划
agent-eval-framework         → Eval 计划 + 每周 Production Review 议程

技术决策：
tech-stack-selection         → 技术栈决策
multi-agent-orchestration    → 多 Agent 架构 Spec（如适用）

上线与迭代：
launch-risk-review           → Launch Risk Review
iteration-flywheel           → 迭代飞轮设计（上线后）
```

有些交付物是必选，有些按项目复杂度启用，但原则一致：每一个关键决策都应该沉淀成客户可审阅、可签字、可执行的文档。

这些交付物的起始模板已经放在 [`templates/`](templates/) 目录中。

---

## 贡献

如果你做过 AI Native 产品，有一个值得分享的决策框架——欢迎开 Issue 讨论。

门槛只有一条：**必须来自生产经验**。看过的论文不算，听说的故事不算，自己亲手踩过的坑才算。要有决策框架（不只是概念解释），要见过失败模式，不只是成功案例。

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

## License

Apache 2.0。自由使用，注明出处即可。

---

*由 [Medoc May](https://github.com/MedocMay/ai-native-builder-consultant-skills) 写成——一个用两年时间把 AI Native 产品开发的所有坑都踩了一遍的创始人。*
