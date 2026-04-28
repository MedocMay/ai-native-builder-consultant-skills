# 变更日志

本文档记录 Skills 库的主要变更。格式参考 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

---

## [v3.1] - 2026-04 · 前端发布版

### 新增
- **三个交互式前端界面**（在 `docs/` 目录，可直接部署到 GitHub Pages）
  - `index.html` — 统一入口页，让用户从三个界面中选择
  - `chat.html` — 快速咨询（chatbot 形式 + 4 个 a16z 经典 AI Native 场景 + 悬赏 + 提示词模板/增强器 + 时间轴 + 会话历史 + 分享卡片）
  - `explore.html` — 编辑刊物风格的 skills 浏览器
  - `workbench.html` — 类 Claude Code 桌面版工作台
- **真实 Claude API 后端对接**（chat.html 的 LIVE 模式）
  - DEMO / LIVE 双模式切换
  - API Key 配置弹窗（localStorage 保存）
  - Skills 按当前 SOP 阶段自动注入 system prompt
  - 流式响应（SSE 解析）
  - Token / 成本追踪
  - 错误处理与 fallback
- **DEPLOY.md 部署指南** — 5 分钟部署到 GitHub Pages 的完整流程，包括自定义品牌、升级为后端代理、嵌入 iframe 等
- **examples/ 目录** — 包含一个真实咨询案例（材料成本分析系统）的 5 个脱敏交付物
- **OG 分享图** — `docs/assets/og-image.svg`

### 改动
- README 新增"在线体验"和"仓库结构"两个 section
- skills/ 目录扁平化（去掉中间的 `/skills/` 一层）

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

---

## 贡献者

感谢所有在项目中提出建议、报告问题、贡献 Skills 的人。

欢迎提交 PR 或 Issue，详见 [CONTRIBUTING.md](CONTRIBUTING.md)。
