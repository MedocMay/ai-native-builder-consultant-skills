# AI Native 产品咨询工作流 — 完整 SOP

版本：v3.0 | 更新日期：2026-04-13
变更：重构入口，新增场景分析→伪需求识别→项目定义→PRD Lite 前期工作流；明确 DL 为场景驱动的可选项，整体以 AI Native Agent Builder 为主；支持开源发布

---

## 核心设计原则

**原则 1：整体以 AI Native Agent Builder 为主**
大多数企业 AI 项目的核心是"让 AI 理解业务、做出判断、生成内容、驱动流程"，这是 Agent 范式的本质。DL 小模型（CV/OCR/时序/语音）是场景驱动的补充，不是架构预设。

**原则 2：DL 由场景决定，不是必须**
只有当输入是图像/文档图片/时序数据/语音时，才考虑 DL 小模型。没有这类输入，直接走大模型/Agent 路径。

**原则 3：大小模型连用是结论，不是前提**
连用的触发条件：① 有非结构化输入需要感知，② 感知结果需要被进一步理解或生成自然语言。两个条件都满足才引入。

**原则 4：先问"该不该做"，再问"怎么做"**
任何 AI 项目都要先过场景分析和伪需求检查，防止在错误的需求上浪费资源。

**原则 5：最小化原则**
每次只验证一件事，第一版只做最小可用价值，成功标准可以被测量。

---

## SOP 总览：三个阶段，九个模块

```
┌─────────────────────────────────────────────────────────────┐
│  阶段 0：前期诊断（必做，不可跳过）                          │
│  Module 0  场景分析与路径判断                                │
│  Module 0.5 伪 AI 需求识别                                   │
├─────────────────────────────────────────────────────────────┤
│  阶段 1：需求定义                                            │
│  Module 1  AI 项目定义                                       │
│  Module 1.5 PRD Lite（最小产品文档）                         │
├─────────────────────────────────────────────────────────────┤
│  阶段 2：技术设计与实施（按路径分叉）                         │
│                                                             │
│  Agent 路径（主路径）      DL 补充路径（场景驱动）           │
│  Module 2  Agent 边界设计  DL-M2  数据策略                   │
│  Module 3  数据与 Eval     DL-M3  模型选型与训练             │
│  Module 4  技术选型        DL-M4  部署与集成                 │
│  Module 4.5 MVP 验证                                        │
├─────────────────────────────────────────────────────────────┤
│  阶段 3：上线与持续改进                                      │
│  Module 5  上线风险评审                                      │
│  Module 6  迭代飞轮                                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 阶段 0：前期诊断

### Module 0 — 场景分析与路径判断

**核心 Skill：** `scene-analysis`

**目标：** 在进入任何技术讨论之前，判断需求属于哪类、推荐哪条路径、DL 是否必要。

**四步判断框架：**
1. 需求分类（A类：大模型/Agent / B类：Workflow / C类：规则/BI / D类：暂缓）
2. 技术路径初判（知识库RAG / Workflow / Agent / DL / 规则代码）
3. 优先级评估（业务价值 × 实施难度 × 当前基础）
4. DL与大模型组合决策（场景驱动，不预设）

**路径分叉：**
```
A/B 类 → Module 0.5（伪需求检查）→ Module 1（项目定义）
C 类（DL）→ `dl-problem-framing` → DL-Module 2-4
D 类（规则）→ Vibe Coding，不进入 AI SOP
D 类（暂缓）→ 记录原因，等条件具备
```

**交付物：** `场景分析报告`（含分类/路径/优先级/DL判断）

**Go/No-Go：** 路径清晰，团队对"做什么"和"不做什么"达成一致

---

### Module 0.5 — 伪 AI 需求识别

**核心 Skill：** `pseudo-ai-detection`

**目标：** 防止在错误的需求上启动项目。八项检查清单，识别伪需求。

**八项检查：**
1. 没有明确用户？
2. 没有具体任务？
3. 写得像愿景不像项目？
4. 第一阶段无法验证？
5. 更像流程问题？
6. 更像数据治理问题？
7. 自己推不动（依赖太多外部条件）？
8. 做错风险过高？

**四种结论：**
- 继续推进 → Module 1
- 缩小重写 → 重新做 Module 0
- 非AI优先解 → 先做流程/数据治理
- 暂缓 → 说明缺失条件

**交付物：** `伪需求检查结果`（含结论和重写建议）

---

## 阶段 1：需求定义

### Module 1 — AI 项目定义

**核心 Skill：** `ai-project-definition`

**目标：** 把一句模糊需求改写成可落地的项目定义。

**九个必须回答的问题：**
1. 目标用户是谁？（具体角色，不是"员工"）
2. 当前具体问题是什么？（不写口号，写实际问题）
3. 当前这个问题怎么被处理？（现有流程）
4. 当前处理方式最大的痛点？（选1-2个，加具体说明）
5. AI 最适合介入哪一步？（具体的一步，不是全流程）
6. 哪一步必须保留人工？（Zone C 的雏形）
7. 第一版最小可用价值是什么？（一句话，可验证）
8. 成功标准是什么？（最多3条，可测量）
9. 当前明确不做什么？（至少2条排除项）

**自检表：** 8项全部通过后才能进入下一模块

**交付物：** `AI 项目定义卡`（含自检结果）

---

### Module 1.5 — PRD Lite

**核心 Skill：** `prd-lite`

**目标：** 把项目定义升级为可流转的最小产品文档，是进入技术设计的正式输入。

**11个模块：**
项目背景 → 目标用户 → 典型使用场景 → 当前问题与痛点 → 核心任务流（4-6步）→ 输入与输出 → MVP范围 → 技术路径初判 → 风险与边界 → 成功验证标准 → 下一步建议

**质量红线：** 9项检查清单，全部通过后进入技术设计阶段

**交付物：** `PRD Lite`

---

## 阶段 2：技术设计与实施

### 路径 A：Agent 范式（主路径）

适用场景：决策辅助、流程自动化、对话系统、知识问答、多步骤推理

#### Module 2 — Agent 边界设计

**核心 Skills：** `agent-boundary-design` · `system-prompt-engineering` · `human-in-the-loop-design` · `knowledge-injection-strategy` · `trust-and-autonomy-tradeoff` · `ai-native-ux-patterns`

**目标：** 定义 Agent 的三个区域，设计 Handoff，写 SOUL 文件。

**四个核心问题：**
1. 这个 Agent 的单一 Job Story 是什么？
2. 三个最高风险动作的可逆性/错误代价/模型置信度评分？
3. Zone A/B/C 如何划分？
4. Handoff 到人工和回 Agent 的触发条件？

**Zone 定义：**
- Zone A：AI 自主执行，无需人工介入
- Zone B：AI 草稿，人工确认后执行
- Zone C：拒绝执行，强制人工

**交付物：** `Agent Boundary Spec` + `SOUL.md` + `HITL Design Spec` + `Knowledge Architecture Decision` + `Trust-Autonomy Calibration Plan`

**Go/No-Go：** Zone C 明确，Zone B 有 UX 模式，SOUL 文件稳定

---

#### Module 3 — 数据与 Eval 基础

**核心 Skill：** `data-foundation`

**目标：** 建立数据管道、标注策略、评估集。

**五个关键问题：**
1. 现有什么数据？
2. 谁能判断输出好坏？
3. 主要任务类型（检索/分类/生成/流程）？
4. 有没有冷启动问题？
5. 数据合规约束？

**交付物：** `Data Foundation Plan` + `Eval Plan`

**Go/No-Go：** 数据可导出，实体对齐，评估集已定义

---

#### Module 4 — 技术选型与实施计划

**核心 Skills：** `tech-stack-selection` · `multi-agent-orchestration`（条件）· `llm-capability-boundaries`

**五个约束优先：**
1. 部署环境（云/边缘/内网）
2. 数据合规（是否能出内网）
3. 响应时间要求
4. 团队技术栈
5. 当前阶段（MVP/生产）

**框架选择原则（不预设框架）：**
- 国产硬件优先 PaddleX/PaddlePaddle 生态
- NVIDIA GPU 两者皆可
- 团队熟悉什么用什么
- 任务匹配度优先于框架统一性

**交付物：** `Tech Stack Decision` + `Build/Rollout Plan`

---

### 路径 B：DL 补充路径（场景驱动，仅在需要时引入）

**引入条件：** 核心输入是图像/文档图片/时序数据/语音时才引入

#### DL-Module 2 — 数据策略

**核心 Skill：** `dl-data-strategy`

**核心原则：**
- 先体验预训练模型基线，再决定标注量
- 标注质量 > 标注数量
- PaddleX check_dataset 格式校验先行
- 训练/验证集按实体或时间划分，不能随机

**数据量参考（微调场景）：**

| 任务 | 最少可用 | 建议 |
|------|---------|------|
| 图像分类（每类） | 50张 | 200张 |
| 目标检测（每类） | 100张 | 500张 |
| 异常检测（正常样本） | 200张 | 500张 |
| 时序预测 | 1000个时间点 | 10000+ |
| OCR文本识别 | 500条 | 2000+ |

**交付物：** `DL 数据策略报告`

---

#### DL-Module 3 — 模型选型与训练

**核心 Skills：** `dl-model-selection` · `cv-pipeline` / `ocr-pipeline` / `ts-pipeline`

**框架选型地图（不限于 PaddleX）：**

| 任务 | PaddleX 方案 | 其他框架方案 |
|------|------------|------------|
| CV 目标检测 | PP-YOLOE/PicoDet | YOLOv8/RT-DETR（PyTorch）|
| CV 异常检测 | STFPM | PatchCore/FastFlow（PyTorch）|
| OCR 通用 | PP-OCRv4 | — |
| 文档信息抽取 | PP-ChatOCRv3/v4 | LayoutLMv3（HuggingFace）|
| 时序预测 | PatchTST/DLinear | Informer/iTransformer（PyTorch）|
| 语音识别（中文） | ❌ | FunASR/Paraformer |
| 语音识别（多语言） | ❌ | Whisper（OpenAI）|
| NLP 分类 | ❌ | BERT/ERNIE（HuggingFace）|

**五步选型法：**
1. 一行命令体验预训练模型
2. 在真实业务数据上评估 gap
3. 决定微调 vs 从头训练
4. 选择模型规格（精度 vs 速度）
5. 评估集验证

**交付物：** 训练模型 + `DL 模型选型决策`

---

#### DL-Module 4 — 部署与大模型集成

**核心 Skill：** `dl-deployment-pattern`

**三种部署模式：**
- 高性能推理：云端，追求极致速度，`use_hpip=True`
- 服务化部署：多系统调用，`paddlex --serve`
- 端侧部署：工厂现场，模型量化+Paddle Lite

**大小模型集成接口设计：**
- 小模型输出必须是结构化 JSON
- 大模型只在需要理解/分析时才调用（控制成本）
- 大模型失败时有 fallback

**交付物：** `DL 部署方案` + API 文档

---

### Module 4.5 — MVP 验证（两条路径共用）

**核心 Skill：** `mvp-validation`

**目标：** 防止做着做着跑偏，确保每次迭代都在验证正确的事情。

**三个工具：**
1. 原型最小验证说明卡（迭代开始时填写）
2. Demo 展示脚本（展示前填写）
3. 项目展示页（向更广受众展示）

**关键原则：**
- 每次迭代只验证一件事
- Demo 要用真实数据，不只是测试数据
- 验证结论要有明确的 Go/No-Go 判断

**交付物：** `MVP 验证说明卡` + `Demo 展示脚本` + `项目展示页`

---

## 阶段 3：上线与持续改进

### Module 5 — 上线风险评审

**核心 Skill：** `launch-risk-review`

**目标：** 正式 Go/No-Go 决策，确保系统安全上线。

**六大检查领域：**
1. 模型行为（Vibe Check + Golden Dataset 回归）
2. 信任区与 HITL（Zone A 可逆性/Zone B UX 测试/Zone C 升级路径）
3. 失败模式（优雅降级/级联防护/超时处理）
4. 数据与隐私（数据合规/审计追踪）
5. 运营（监控/回滚/值班）
6. 用户准备度（真实用户测试/SOP文档）

**三种结论：**
- Go：全部通过
- Conditional Go：Section 4-6 有条件，有owner和deadline
- No-Go：Section 1-3 有未解决的问题

**交付物：** `Launch Risk Review` + `Rollout Strategy`

---

### Module 6 — 迭代飞轮

**核心 Skill：** `iteration-flywheel`

**目标：** 建立生产数据 → 持续改进的正向循环。

**核心仪式：** 每周 30 分钟生产复盘（必做）

**四个改进杠杆（优先级从高到低）：**
1. Prompt/SOUL 修改（成本最低，首选）
2. 检索/RAG 改进（中等成本）
3. Zone 边界/阈值调整（低成本，按需）
4. 数据/模型层改进（最高成本，最后考虑）

**DL 系统的双飞轮：**
- 数据飞轮：低置信度样本标注 → 更新训练集 → 再训练
- 模型飞轮：精度监控 → 分布偏移检测 → 触发再训练

**交付物：** `Iteration Flywheel Design`

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
| 质检+原因分析 | DL-CV + Agent（连用）| `scene-analysis` |
| 票据识别+财务入账 | DL-OCR + Agent（连用）| `scene-analysis` |
| 固定报表、数据统计 | 不需要 AI | Vibe Coding |
| 流程混乱、数据不干净 | 非AI优先 | `pseudo-ai-detection` |

---

## 完整 Skills 库一览（v3.0，28个）

### 前期诊断（新增）
| Skill | 说明 |
|-------|------|
| `scene-analysis` | 场景分析与路径判断，所有项目的入口 |
| `pseudo-ai-detection` | 伪AI需求识别，防止在错误需求上浪费资源 |

### 需求定义（新增）
| Skill | 说明 |
|-------|------|
| `ai-project-definition` | AI项目定义卡，把模糊需求变成可落地定义 |
| `prd-lite` | 最小产品文档，进入技术设计的正式输入 |
| `mvp-validation` | MVP验证说明卡+展示脚本+项目展示页 |

### Agent 范式（原有）
| Skill | 说明 |
|-------|------|
| `ai-native-vs-ai-enhanced` | AI Native vs AI-enhanced 判断 |
| `agent-loop-model` | Agent 机械运行模型理解 |
| `llm-capability-boundaries` | 大模型能力边界 |
| `uncertainty-and-hallucination` | 不确定性与幻觉处理 |
| `agent-boundary-design` | Agent边界设计（Zone A/B/C） |
| `system-prompt-engineering` | SOUL文件五层结构写法 |
| `human-in-the-loop-design` | Zone B确认体验设计 |
| `knowledge-injection-strategy` | RAG vs Fine-tuning决策 |
| `ai-native-ux-patterns` | AI Native UX 六种模式 |
| `tech-stack-selection` | 技术栈选型（不预设框架）|
| `trust-and-autonomy-tradeoff` | 信任与自主权权衡 |
| `data-foundation` | Agent数据基础与Eval设计 |
| `agent-eval-framework` | Agent评估框架 |
| `multi-agent-orchestration` | 多Agent编排 |
| `launch-risk-review` | 上线风险评审 |
| `iteration-flywheel` | 迭代飞轮设计 |

### DL 补充（新增，场景驱动）
| Skill | 说明 |
|-------|------|
| `dl-problem-framing` | 三条路径判断：只用DL/只用大模型/连用 |
| `dl-data-strategy` | DL数据策略（支持PaddleX及其他框架）|
| `dl-model-selection` | 模型选型（PaddleX+PyTorch+HuggingFace）|
| `dl-deployment-pattern` | 三种部署模式+大小模型集成接口 |
| `cv-pipeline` | CV全链条（分类/检测/分割/异常检测）|
| `ocr-pipeline` | OCR全链条（PP-OCRv4/PP-ChatOCRv3）|
| `ts-pipeline` | 时序全链条（预测/异常/分类）|

---

## 硬依赖规则

**阶段 0 依赖：**
- 没有场景分析报告 → 不启动任何技术设计
- 没有通过伪需求检查 → 不进入项目定义

**Agent 范式依赖：**
- 没有 PRD Lite → 不开始 Agent 设计
- 没有 Boundary Spec → 不写 SOUL
- 没有 Data Foundation Plan → 不声称 Eval 就绪
- 没有通过 Launch Risk Review → 不上线

**DL 范式依赖：**
- 没有跑预训练模型基线 → 不开始大规模数据标注
- 没有通过 check_dataset → 不开始训练
- 没有评估集精度报告 → 不部署到生产
- 没有回滚方案 → 不上线

---

## 开源说明

本 SOP 和 Skills 库基于 Apache 2.0 协议开源。

**贡献指南：**
- 每个 Skill 是独立的 Markdown 文件，YAML frontmatter 包含 name 和 description
- Skills 按使用场景组织，不按技术类型组织
- 新增 Skill 需要同时更新本 SOP 的引用和 Skills 库一览
- 提交 PR 时，需要说明：这个 Skill 解决什么问题、在 SOP 的哪个位置使用、与现有 Skills 的关系

**仓库结构：**
```
ai-native-builder-consultant-skills/
├── CONSULTING-WORKFLOW.md          ← 本文件（SOP主文档）
├── TERMINOLOGY.md                  ← 术语表
├── CONTRIBUTING.md                 ← 贡献指南
├── README.md                       ← 项目介绍
├── mental-models/skills/           ← 认知基础类
├── pre-diagnosis/skills/           ← 前期诊断类（新增）
│   ├── scene-analysis/
│   ├── pseudo-ai-detection/
│   └── ...
├── requirement-definition/skills/  ← 需求定义类（新增）
│   ├── ai-project-definition/
│   ├── prd-lite/
│   └── mvp-validation/
├── agent-design/skills/            ← Agent设计类
├── build-skills/skills/            ← 构建技能类
└── dl-skills/skills/               ← DL补充类（新增）
    ├── dl-problem-framing/
    ├── dl-model-selection/
    ├── cv-pipeline/
    ├── ocr-pipeline/
    └── ts-pipeline/
```

---

_Last updated: 2026-04-13 | Version: v3.0 | 开源许可：Apache 2.0_
_核心设计：整体以 AI Native Agent Builder 为主，DL 为场景驱动的可选补充_
