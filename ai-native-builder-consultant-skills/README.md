# AI Native Builder Consultant Skills

> 专门为 AI Native 产品构建者设计的咨询系统 · 不只是用 AI，而是帮你 **建** AI 产品

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)
[![Skills](https://img.shields.io/badge/Skills-28-orange?style=flat-square)](#完整-skills-库一览)
[![Version](https://img.shields.io/badge/Version-v3.0-purple?style=flat-square)](CONSULTING-WORKFLOW.md)

---

## 这是什么

一套完整的 AI Native 产品咨询系统，包含：

- **28 个 Skills** — 每个都是一份经过产业实践验证的决策框架
- **完整 SOP** — 从场景分析到上线飞轮的全流程操作指南
- **工作坊工具** — 伪需求识别清单、项目定义卡、PRD Lite 等即用模板

覆盖的核心问题：

- 这个需求到底该不该用 AI？
- 用大模型还是用小模型？什么时候大小模型连用？
- Agent 的边界在哪里？Zone A/B/C 怎么划分？
- SOUL 文件怎么写？HITL 怎么设计？
- CV/OCR/时序/语音任务怎么落地？选 PaddleX 还是 PyTorch？
- 上线前的检查清单？上线后的迭代飞轮？

---

## 快速开始

### 安装方式

**方式 1：Claude.ai 使用（推荐）**
```bash
# 下载并解压到 Claude.ai 的 skills 目录
git clone https://github.com/MedocMay/ai-native-builder-consultant-skills.git
# 参考 Claude.ai 的 Skills 使用文档
```

**方式 2：直接阅读使用**
```bash
git clone https://github.com/MedocMay/ai-native-builder-consultant-skills.git
cd ai-native-builder-consultant-skills
# 从 CONSULTING-WORKFLOW.md 开始读
```

### 使用流程

```
1. 读 CONSULTING-WORKFLOW.md（整体咨询流程）
2. 遇到具体问题时，找对应的 Skill 使用
3. 从 scene-analysis 开始（所有 AI 项目的入口）
4. 按 SOP 路径走，每一步都有对应的 Skill
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
- [`scene-analysis`](pre-diagnosis/skills/scene-analysis/SKILL.md) — 场景分析与路径判断
- [`pseudo-ai-detection`](pre-diagnosis/skills/pseudo-ai-detection/SKILL.md) — 伪 AI 需求识别

### 需求定义（3）
- [`ai-project-definition`](requirement-definition/skills/ai-project-definition/SKILL.md) — AI 项目定义卡
- [`prd-lite`](requirement-definition/skills/prd-lite/SKILL.md) — 最小产品文档
- [`mvp-validation`](requirement-definition/skills/mvp-validation/SKILL.md) — MVP 验证三件套

### 认知基础（4）
- [`ai-native-vs-ai-enhanced`](mental-models/skills/ai-native-vs-ai-enhanced/SKILL.md) — AI Native vs AI-enhanced 判断
- [`agent-loop-model`](mental-models/skills/agent-loop-model/SKILL.md) — Agent 机械运行模型
- [`llm-capability-boundaries`](mental-models/skills/llm-capability-boundaries/SKILL.md) — 大模型能力边界
- [`uncertainty-and-hallucination`](mental-models/skills/uncertainty-and-hallucination/SKILL.md) — 不确定性与幻觉

### Agent 设计（7）
- [`agent-boundary-design`](agent-design/skills/agent-boundary-design/SKILL.md) — Agent 边界设计（Zone A/B/C）
- [`system-prompt-engineering`](agent-design/skills/system-prompt-engineering/SKILL.md) — SOUL 五层结构
- [`human-in-the-loop-design`](agent-design/skills/human-in-the-loop-design/SKILL.md) — HITL 确认体验设计
- [`knowledge-injection-strategy`](agent-design/skills/knowledge-injection-strategy/SKILL.md) — RAG vs Fine-tuning
- [`ai-native-ux-patterns`](agent-design/skills/ai-native-ux-patterns/SKILL.md) — AI Native UX 模式
- [`tech-stack-selection`](agent-design/skills/tech-stack-selection/SKILL.md) — 技术栈选型
- [`trust-and-autonomy-tradeoff`](agent-design/skills/trust-and-autonomy-tradeoff/SKILL.md) — 信任与自主权权衡

### 构建技能（5）
- [`data-foundation`](build-skills/skills/data-foundation/SKILL.md) — 数据基础与 Eval 设计
- [`agent-eval-framework`](build-skills/skills/agent-eval-framework/SKILL.md) — Agent 评估框架
- [`multi-agent-orchestration`](build-skills/skills/multi-agent-orchestration/SKILL.md) — 多 Agent 编排
- [`launch-risk-review`](build-skills/skills/launch-risk-review/SKILL.md) — 上线风险评审
- [`iteration-flywheel`](build-skills/skills/iteration-flywheel/SKILL.md) — 迭代飞轮设计

### DL 补充（7）
- [`dl-problem-framing`](dl-skills/skills/dl-problem-framing/SKILL.md) — 三条路径决策
- [`dl-data-strategy`](dl-skills/skills/dl-data-strategy/SKILL.md) — DL 数据策略
- [`dl-model-selection`](dl-skills/skills/dl-model-selection/SKILL.md) — 模型选型（多框架）
- [`dl-deployment-pattern`](dl-skills/skills/dl-deployment-pattern/SKILL.md) — 部署与大小模型集成
- [`cv-pipeline`](dl-skills/skills/cv-pipeline/SKILL.md) — CV 全链条
- [`ocr-pipeline`](dl-skills/skills/ocr-pipeline/SKILL.md) — OCR 全链条
- [`ts-pipeline`](dl-skills/skills/ts-pipeline/SKILL.md) — 时序全链条

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
- **大小模型连用** — 来自真实工业质检、财务分析等场景

特别感谢所有在真实项目中踩过坑、积累过经验的从业者。本仓库是这些经验的系统化总结。

---

## 许可证

[Apache License 2.0](LICENSE)

你可以自由使用、修改、商用本仓库内容，只需保留署名。

---

_Last updated: 2026-04-13 | Version: v3.0_
