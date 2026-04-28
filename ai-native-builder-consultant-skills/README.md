# AI Native Builder Consultant Skills

> 专门为 AI Native 产品构建者设计的咨询系统 · 不只是用 AI，而是帮你 **建** AI 产品

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)
[![Skills](https://img.shields.io/badge/Skills-28-orange?style=flat-square)](#完整-skills-库一览)
[![Version](https://img.shields.io/badge/Version-v3.1-purple?style=flat-square)](CONSULTING-WORKFLOW.md)

---

## 🚀 在线体验

部署到 GitHub Pages 后，访问 `https://medocmay.github.io/ai-native-builder-consultant-skills/`，可以选择三个交互界面之一：

| 入口 | 适合谁 | 用途 |
|------|--------|------|
| **`/chat.html`** · 快速咨询 | 想立刻试一下的潜在客户 | 4 个 a16z 经典 AI Native 场景，3 分钟跑完核心咨询路径，结尾可悬赏 |
| **`/explore.html`** · 浏览全部 | 想学习和参考的工程师/PM | 28 个 skills 编辑刊物风格浏览，按分类筛选，可阅读完整内容 |
| **`/workbench.html`** · 工作台 | 咨询师日常使用 | 类 Claude Code 桌面版，左侧目录树 + 主区对话 |

**完整部署指南：** [DEPLOY.md](DEPLOY.md)

---

## 这是什么

一套完整的 AI Native 产品咨询系统：

- **28 个 Skills** — 每个都是一份经过产业实践验证的决策框架
- **完整 SOP** — 从场景分析到上线飞轮的全流程操作指南
- **三个前端界面** — 在线咨询体验、skills 浏览、工作台
- **真实案例** — 包含一个完整的脱敏咨询案例供参考

覆盖的核心问题：

- 这个需求到底该不该用 AI？
- 用大模型还是用小模型？什么时候大小模型连用？
- Agent 的边界在哪里？Zone A/B/C 怎么划分？
- SOUL 文件怎么写？HITL 怎么设计？
- CV/OCR/时序/语音任务怎么落地？选 PaddleX 还是 PyTorch？
- 上线前的检查清单？上线后的迭代飞轮？

---

## 快速开始

### 方式 1：在浏览器里直接体验（5 分钟）

```bash
git clone https://github.com/MedocMay/ai-native-builder-consultant-skills.git
cd ai-native-builder-consultant-skills

# 直接在浏览器打开
open docs/index.html        # macOS
xdg-open docs/index.html    # Linux
start docs/index.html       # Windows
```

### 方式 2：部署到 GitHub Pages

详见 [DEPLOY.md](DEPLOY.md)。

### 方式 3：在 Claude.ai / Claude Code 里使用

```bash
git clone https://github.com/MedocMay/ai-native-builder-consultant-skills.git
# 把 skills/ 目录下的内容拷贝到你的 Claude.ai skills 目录
# 或者在 Claude 对话中说"按 SOP 启动咨询，我的场景是：..."
```

---

## 核心设计原则

**原则 1：整体以 AI Native Agent Builder 为主**
大多数企业 AI 项目的核心是 Agent 范式。DL 小模型是场景驱动的补充。

**原则 2：DL 由场景决定，不是必须**
只有当输入是图像/文档图片/时序数据/语音时，才考虑 DL 小模型。

**原则 3：大小模型连用是结论，不是前提**
两个条件都满足才引入连用：① 有非结构化输入需要感知，② 感知结果需要被理解或生成。

**原则 4：先问"该不该做"，再问"怎么做"**
任何 AI 项目都要先过场景分析和伪需求检查。

**原则 5：最小化原则**
每次只验证一件事，第一版只做最小可用价值，成功标准可测量。

---

## 仓库结构

```
ai-native-builder-consultant-skills/
├── README.md                    ← 你正在读
├── CONSULTING-WORKFLOW.md       ← SOP 主文档（必读）
├── DEPLOY.md                    ← 部署到 GitHub Pages 指南
├── CONTRIBUTING.md              ← 贡献规范
├── TERMINOLOGY.md               ← 术语表
├── CHANGELOG.md
├── LICENSE                      ← Apache 2.0
│
├── skills/                      ← 28 个 SKILL.md，按分类组织
│   ├── pre-diagnosis/           ← 前期诊断（2 个）
│   ├── requirement-definition/  ← 需求定义（3 个）
│   ├── mental-models/           ← 认知基础（4 个）
│   ├── agent-design/            ← Agent 设计（7 个）
│   ├── build-skills/            ← 构建技能（5 个）
│   └── dl-skills/               ← DL 补充（7 个）
│
├── docs/                        ← GitHub Pages 服务的目录（直接部署）
│   ├── index.html               ← 入口页
│   ├── chat.html                ← 快速咨询
│   ├── explore.html             ← 浏览全部
│   ├── workbench.html           ← 工作台
│   └── assets/og-image.svg
│
└── examples/                    ← 真实咨询案例（脱敏）
    └── 材料成本分析系统/
        └── ...（5 个交付物）
```

---

## SOP 总览

```
┌─────────────────────────────────────────────────────────────┐
│  阶段 0：前期诊断（必做，不可跳过）                          │
│  · 场景分析与路径判断                                        │
│  · 伪 AI 需求识别                                            │
├─────────────────────────────────────────────────────────────┤
│  阶段 1：需求定义                                            │
│  · AI 项目定义卡                                             │
│  · PRD Lite                                                │
├─────────────────────────────────────────────────────────────┤
│  阶段 2：技术设计与实施（按路径分叉）                         │
│                                                             │
│  Agent 主路径             DL 补充路径（场景驱动）            │
│  · Agent 边界设计         · DL 数据策略                     │
│  · 数据与 Eval 基础       · 模型选型与训练                   │
│  · 技术选型与实施         · 部署与集成                       │
│  · MVP 验证                                                 │
├─────────────────────────────────────────────────────────────┤
│  阶段 3：上线与持续改进                                      │
│  · 上线风险评审                                              │
│  · 迭代飞轮                                                  │
└─────────────────────────────────────────────────────────────┘
```

完整 SOP：[CONSULTING-WORKFLOW.md](CONSULTING-WORKFLOW.md)

---

## 完整 Skills 库一览

### 前期诊断（2）
- `scene-analysis` — 场景分析与路径判断
- `pseudo-ai-detection` — 伪 AI 需求识别

### 需求定义（3）
- `ai-project-definition` — AI 项目定义卡
- `prd-lite` — 最小产品文档
- `mvp-validation` — MVP 验证三件套

### 认知基础（4）
- `ai-native-vs-ai-enhanced` — AI Native vs AI-enhanced 判断
- `agent-loop-model` — Agent 机械运行模型
- `llm-capability-boundaries` — 大模型能力边界
- `uncertainty-and-hallucination` — 不确定性与幻觉

### Agent 设计（7）
- `agent-boundary-design` — Agent 边界设计（Zone A/B/C）
- `system-prompt-engineering` — SOUL 五层结构
- `human-in-the-loop-design` — HITL 确认体验设计
- `knowledge-injection-strategy` — RAG vs Fine-tuning
- `ai-native-ux-patterns` — AI Native UX 模式
- `tech-stack-selection` — 技术栈选型
- `trust-and-autonomy-tradeoff` — 信任与自主权权衡

### 构建技能（5）
- `data-foundation` — 数据基础与 Eval 设计
- `agent-eval-framework` — Agent 评估框架
- `multi-agent-orchestration` — 多 Agent 编排
- `launch-risk-review` — 上线风险评审
- `iteration-flywheel` — 迭代飞轮设计

### DL 补充（7，场景驱动）
- `dl-problem-framing` — 三条路径决策
- `dl-data-strategy` — DL 数据策略
- `dl-model-selection` — 模型选型（多框架）
- `dl-deployment-pattern` — 部署与大小模型集成
- `cv-pipeline` — CV 全链条
- `ocr-pipeline` — OCR 全链条
- `ts-pipeline` — 时序全链条

---

## 路径判断速查表

| 业务描述 | 推荐路径 | 入口 Skill |
|---------|---------|-----------|
| 对话、问答、知识检索 | Agent/RAG | `scene-analysis` |
| 流程自动化、审批决策 | Agent/Workflow | `scene-analysis` |
| 报告生成、内容创作 | Agent/LLM | `scene-analysis` |
| 图像识别、缺陷检测 | DL-CV | `dl-problem-framing` |
| 票据识别、文档提取 | DL-OCR | `dl-problem-framing` |
| 传感器异常、成本预测 | DL-TS | `dl-problem-framing` |
| 语音识别、录音分析 | DL-Speech | `dl-problem-framing` |
| 质检+原因分析 | DL + Agent（连用） | `scene-analysis` |
| 固定报表、数据统计 | 不需要 AI | 直接写代码 |
| 流程混乱、数据不干净 | 非 AI 优先 | `pseudo-ai-detection` |

---

## 适合谁用

**适合：**
- 正在决定要不要做某个 AI 项目的技术负责人
- 需要给业务方解释"为什么这个需求不适合做 AI"的产品经理
- 第一次构建 Agent 架构的工程师
- 想要规范企业内部 AI 咨询流程的团队

**不太适合：**
- 寻找"AI 提示词技巧"的用户
- 需要填空式模板就能解决问题的场景
- 还没决定要做 AI 的观望者

---

## 贡献

欢迎提交 PR 和 Issue！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

**贡献方向：**
- 补充新的 Skill（特别是垂直行业场景）
- 改进现有 Skill（增加真实案例、修正错误）
- 翻译（英文/日文等）
- 添加术语表和交叉引用

---

## 致谢

本 Skills 库整合了多个来源的产业实践经验：

- **Agent 范式基础** — 基于 OpenClaw 等 AI Agent 框架的实际项目经验
- **DL 落地范式** — 参考 PaddlePaddle PaddleX 全链条开发范式
- **前期诊断工具** — 来自企业内训工作坊实践
- **AI Native 场景启发** — 参考 a16z 的 AI Application Spending Report、Big Ideas 2026 等研究

特别感谢所有在真实项目中踩过坑、积累过经验的从业者。本仓库是这些经验的系统化总结。

---

## 许可证

[Apache License 2.0](LICENSE)

你可以自由使用、修改、商用本仓库内容，只需保留署名。

---

_Last updated: 2026-04 | Version: v3.1_
