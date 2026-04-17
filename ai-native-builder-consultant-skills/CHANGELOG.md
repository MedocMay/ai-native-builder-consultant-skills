# 变更日志

本文档记录 Skills 库的主要变更。格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

---

## [v3.0] - 2026-04-13

### 新增
- **前期诊断阶段**（阶段 0）— 所有 AI 项目的真正入口
  - `scene-analysis` — 场景分析与路径判断（A/B/C/D 分类、技术路径初判）
  - `pseudo-ai-detection` — 伪 AI 需求识别（八项检查清单）
- **需求定义阶段**（阶段 1）— 从模糊需求到可落地项目
  - `ai-project-definition` — AI 项目定义卡（九个必答问题 + 自检表）
  - `prd-lite` — 最小产品文档（11 个模块）
  - `mvp-validation` — MVP 验证三件套（验证说明卡/Demo脚本/展示页）

### 变更
- **SOP 重构** — 从 6 Module 扩展为三阶段九模块
- **核心原则明确** — 整体以 AI Native Agent Builder 为主，DL 为场景驱动的可选补充
- **大小模型连用** — 从预设架构改为场景驱动的结论
- **框架不预设** — `dl-model-selection` 支持 PaddleX + PyTorch + HuggingFace 等多框架

### 文档
- 新增 `README.md` — 开源仓库说明
- 新增 `CONTRIBUTING.md` — 贡献指南
- 新增 `TERMINOLOGY.md` — 术语表
- 新增 `LICENSE` — Apache 2.0

---

## [v2.0] - 2026-04

### 新增
- **DL 落地范式** — 7 个 Skills 覆盖 CV/OCR/时序/语音
  - `dl-problem-framing` — DL 问题定性
  - `dl-data-strategy` — DL 数据策略
  - `dl-model-selection` — 模型选型
  - `dl-deployment-pattern` — 部署与集成
  - `cv-pipeline` — CV 全链条
  - `ocr-pipeline` — OCR 全链条
  - `ts-pipeline` — 时序全链条
- **双路径 SOP** — Agent 范式 + DL 范式并行

### 变更
- 基于 PaddleX 全链条开发范式设计 DL Skills
- 增加大小模型连用架构说明

---

## [v1.0] - 2026-03

### 首版发布
- **16 个 Agent 范式 Skills**
  - 认知基础（4个）
  - Agent 设计（7个）
  - 构建技能（5个）
- **六模块 SOP**
  - Module 1 产品定位诊断
  - Module 2 Agent 边界设计
  - Module 3 数据与 Eval 基础
  - Module 4 技术选型与实施计划
  - Module 5 上线风险评审
  - Module 6 迭代飞轮

---

## 贡献者

感谢所有在项目中提出建议、报告问题、贡献 Skills 的人。

欢迎提交 PR 或 Issue，详见 [CONTRIBUTING.md](CONTRIBUTING.md)。
