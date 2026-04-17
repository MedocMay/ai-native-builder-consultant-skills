# AI Native Builder Consultant Skills

> 面向 AI Native 产品构建者的咨询型 Skills 仓库。这个首页用于展示版本亮点和快速导航，完整内容在 [`ai-native-builder-consultant-skills/`](./ai-native-builder-consultant-skills/)。

[![Version](https://img.shields.io/badge/Version-v3.0-purple?style=flat-square)](./ai-native-builder-consultant-skills/CHANGELOG.md)
[![Skills](https://img.shields.io/badge/Skills-28-orange?style=flat-square)](./ai-native-builder-consultant-skills/README.md#完整-skills-库一览)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](./ai-native-builder-consultant-skills/LICENSE)

## 中文摘要

这是一个面向中文 AI Native 产品团队的咨询型 Skills 仓库，目标不是教人“怎么用 AI”，而是帮助团队判断“这个需求该不该做 AI、该走 Agent 还是 DL、第一版应该做到哪里、上线前后怎么验证和迭代”。

仓库内容覆盖从前期诊断、需求定义、Agent 设计、数据与 Eval、技术选型、上线风险评审到迭代飞轮的完整路径，适合技术负责人、产品经理、AI 咨询顾问和企业内训场景直接使用。

如果你是第一次进入这个仓库，建议先看 [`CONSULTING-WORKFLOW.md`](./ai-native-builder-consultant-skills/CONSULTING-WORKFLOW.md)，再根据具体问题跳转到对应 Skill。

## 版本迭代简表

| 版本 | 核心变化 | 关键词 |
|------|----------|--------|
| `v1.0` | 首版发布，建立 Agent 咨询主框架与 16 个基础 Skills | Agent 范式、6 模块 SOP |
| `v2.0` | 引入 DL 补充路径，覆盖 CV/OCR/时序等非结构化输入场景 | DL 落地、大小模型连用 |
| `v3.0` | 重构为三阶段九模块，新增前期诊断与需求定义，把咨询入口前移 | 场景分析、伪 AI 识别、项目定义、PRD Lite |

## v3 更新亮点

- 从 16 个 Skills 扩展到 **28 个 Skills**，新增前期诊断、需求定义和 DL 补充路径。
- SOP 重构为 **三阶段九模块**，从“该不该做 AI”一路覆盖到上线与迭代。
- 首页和文档全部切成 **中文主叙事**，更适合国内 AI Native 咨询、内训和工作坊场景。
- 新增 **场景分析 / 伪 AI 需求识别 / AI 项目定义 / PRD Lite / MVP 验证**，把咨询入口前移。
- 新增 **DL 落地技能组**，覆盖 CV、OCR、时序、部署与大小模型协同。
- 明确 **Agent 为主路径，DL 为场景驱动补充**，避免一上来就预设架构。

## 新增能力一览

### 前期诊断
- [`scene-analysis`](./ai-native-builder-consultant-skills/pre-diagnosis/skills/scene-analysis/SKILL.md)：判断需求属于 Agent、Workflow、DL 还是非 AI
- [`pseudo-ai-detection`](./ai-native-builder-consultant-skills/pre-diagnosis/skills/pseudo-ai-detection/SKILL.md)：识别伪 AI 需求，避免错误立项

### 需求定义
- [`ai-project-definition`](./ai-native-builder-consultant-skills/requirement-definition/skills/ai-project-definition/SKILL.md)：把模糊需求收敛成 AI 项目定义卡
- [`prd-lite`](./ai-native-builder-consultant-skills/requirement-definition/skills/prd-lite/SKILL.md)：输出可流转的最小产品文档
- [`mvp-validation`](./ai-native-builder-consultant-skills/requirement-definition/skills/mvp-validation/SKILL.md)：设计最小验证闭环

### DL 补充路径
- [`dl-problem-framing`](./ai-native-builder-consultant-skills/dl-skills/skills/dl-problem-framing/SKILL.md)：判断什么时候该走 DL 路径
- [`dl-model-selection`](./ai-native-builder-consultant-skills/dl-skills/skills/dl-model-selection/SKILL.md)：支持 PaddleX、PyTorch、HuggingFace 等模型框架选择
- [`cv-pipeline`](./ai-native-builder-consultant-skills/dl-skills/skills/cv-pipeline/SKILL.md)
- [`ocr-pipeline`](./ai-native-builder-consultant-skills/dl-skills/skills/ocr-pipeline/SKILL.md)
- [`ts-pipeline`](./ai-native-builder-consultant-skills/dl-skills/skills/ts-pipeline/SKILL.md)

## 快速导航

- 完整说明：[完整 README](./ai-native-builder-consultant-skills/README.md)
- 工作流总览：[CONSULTING-WORKFLOW.md](./ai-native-builder-consultant-skills/CONSULTING-WORKFLOW.md)
- 变更日志：[CHANGELOG.md](./ai-native-builder-consultant-skills/CHANGELOG.md)
- 术语表：[TERMINOLOGY.md](./ai-native-builder-consultant-skills/TERMINOLOGY.md)
- 贡献说明：[CONTRIBUTING.md](./ai-native-builder-consultant-skills/CONTRIBUTING.md)

## 仓库结构

```text
.
├── README.md
└── ai-native-builder-consultant-skills/
    ├── README.md
    ├── CHANGELOG.md
    ├── CONSULTING-WORKFLOW.md
    ├── agent-design/
    ├── build-skills/
    ├── dl-skills/
    ├── mental-models/
    ├── pre-diagnosis/
    └── requirement-definition/
```

## 使用方式

```bash
git clone git@github.com:MedocMay/ai-native-builder-consultant-skills.git
cd ai-native-builder-consultant-skills/ai-native-builder-consultant-skills
```

建议从 [`CONSULTING-WORKFLOW.md`](./ai-native-builder-consultant-skills/CONSULTING-WORKFLOW.md) 开始，再根据具体问题跳到对应 Skill。
