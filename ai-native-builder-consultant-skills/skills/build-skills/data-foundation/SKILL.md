---
name: data-foundation
description: Build the data foundation for an AI agent: collection, labeling, eval set design, and governance. Use before training or eval setup, when the team has no labeled data, or when data quality is causing inconsistent behavior.
---

# Data Foundation

## When to Use

- Starting an AI agent project with no existing labeled data — the most common real-world situation
- The agent's behavior is inconsistent and you suspect data quality rather than model quality is the root cause
- About to build a golden dataset for eval but unclear what to collect and how to label
- A client asks "what data do we need before we can start" and needs a structured answer
- `agent-eval-framework` has been triggered but the data prerequisite hasn't been addressed
- Post-launch: production data is accumulating but not being captured or organized for improvement

## Ask These Questions First

1. What data exists today — historical logs, domain documents, expert records, prior system outputs? Start from what exists, not what's ideal.
2. Who are the domain experts who can judge whether an agent output is good or bad? Their time is the bottleneck; plan accordingly.
3. What is the primary task the agent performs? (Different tasks need radically different data strategies — retrieval needs a document corpus, classification needs labeled examples, generation needs preference pairs.)
4. Is there a cold-start problem — the agent needs data to work, but data only comes from the agent working? How will you break the cycle?
5. What are the data privacy and residency constraints? (This affects where data can be stored and who can label it.)

## Output Format

```markdown
## Data Foundation Plan

**Product:** [填入]  **Agent task type:** [检索 / 分类 / 生成 / 流程自动化]
**Date:** [填入]  **Cold-start deadline:** [何时需要第一批可用数据]

### 现有数据盘点

| 数据源 | 类型 | 规模 | 质量评估 | 可用性 |
|--------|------|------|---------|--------|
| [数据源 1] | 历史日志 / 领域文档 / 专家记录 | [量] | 高/中/低 | 立即 / 需清洗 / 受限 |
| [数据源 2] | | | | |

**冷启动可用数据：** [填入，用于启动第一版]
**数据缺口：** [填入，需要补充的]

### 数据收集管道

| 阶段 | 来源 | 收集方式 | 数量目标 | 时间线 |
|------|------|---------|---------|--------|
| 冷启动 | [历史数据 / 专家构造] | [手工 / 导出 / 爬取] | [N] | 第 [X] 周 |
| 生产积累 | Zone B 确认记录 | 自动捕获 | [N/周] | 上线后持续 |
| 主动采集 | 随机对话样本 | 系统日志 | [N/周] | 上线后持续 |

### 标注策略

**标注维度：**
| 维度 | 标签选项 | 谁来标 | 标注耗时/条 |
|------|---------|--------|------------|
| 整体质量 | 好 / 可接受 / 差 | 领域专家 | ~2 分钟 |
| 失败类型 | prompt / 检索 / 边界 / 无 | 技术负责人 | ~1 分钟 |
| [领域特定维度] | [选项] | [谁] | [时间] |

**标注速度与资源：**
- 每周可用标注时间：[N] 小时
- 预计标注速度：[N] 条/小时
- 达到冷启动目标（[N] 条）需要：[X] 周

**标注质量控制：**
- 一致性检验：同一批次 10% 双人标注，一致率目标 ≥ 85%
- 仲裁流程：[分歧如何处理]
- 标注规范文档：[在哪里，谁维护]

### 评估集构建

**训练集 / 评估集划分原则：**
- 按 [发动机 SN / 对话 ID / 用户 ID]（不是按时间或随机行）划分
- 评估集比例：20%，且不参与任何训练或 prompt 优化

**评估集结构：**
| 类型 | 数量 | 来源 | 覆盖的失败模式 |
|------|------|------|--------------|
| 正常路径 | [N] | 历史合格案例 | — |
| 边界案例 | [N] | 人工构造 / Zone B 历史 | [列出] |
| 已知失败模式 | [N] | 历史不合格案例 + 变体 | [列出] |
| 对抗性输入 | [N] | 人工构造 | 越界 / 注入 / 歧义 |

**评估集更新机制：**
- 每季度评审一次：移除已修复的失败模式，加入新发现的边界案例
- 触发提前更新的条件：发现系统性新失败模式

### 数据治理

| 维度 | 决策 |
|------|------|
| 存储位置 | [路径 / 数据库] |
| 访问权限 | [谁可以读 / 写 / 删] |
| 数据保留期 | [原始对话保留多久] |
| 个人信息处理 | [脱敏方式 / 不收集] |
| 标注记录保留 | [标注者 ID + 时间戳保留多久] |
| 版本控制 | [数据集版本如何管理] |

### 冷启动路径（无历史数据时）

- [ ] 步骤 1：领域专家手工构造 [N] 条（输入 + 理想输出），覆盖核心场景
- [ ] 步骤 2：用构造数据验证信号处理 / 检索管道是否正确
- [ ] 步骤 3：影子模式上线，开始自动积累真实交互数据
- [ ] 步骤 4：领域专家审核前 [N] 条真实交互，确认分布符合预期
- [ ] 步骤 5：首次构建正式评估集（来自步骤 1 + 步骤 4 的审核数据）
```

## 咨询链位置

**在新版 SOP 中：** 模块 3「数据与 Eval 基础」的起点，负责建立后续验证和飞轮的燃料系统。

**常见联动：**
- `agent-eval-framework` — 使用本 Skill 建立的评估集运行 golden dataset eval
- `knowledge-injection-strategy` — 数据盘点结果决定 RAG vs fine-tuning 的可行性
- `iteration-flywheel` — 数据收集管道和标注策略是飞轮的燃料来源

---

## 核心前提

大多数 AI 项目不是死在算法上，是死在数据上。具体说是三种死法：

**死法一：没有数据就开始建模。** 团队花三周搭好了管道，发现没有标注数据可以训练，也没有评估集可以验证，于是用感觉代替数据做决策。

**死法二：有数据但不知道质量如何。** 历史日志存在，但没有人标注过哪些是好的输出，哪些是坏的。数据量看起来很大，但有效信息密度极低。

**死法三：训练集和评估集没有真正分开。** 按时间划分（前 80% 训练，后 20% 评估）或随机划分——都可能造成数据泄露，导致评估结果虚高，上线后打回原形。

这个 Skill 的目的是在项目开始时建立正确的数据基础，而不是在项目卡住时救场。

---

## 数据分类：三类数据，不同策略

### 类型 1：领域知识数据（给 RAG 用）

**是什么：** 产品文档、政策手册、操作规程、专业知识库——Agent 需要"知道"的内容。

**关键问题：** 文档存在，但不等于可以直接用。需要评估：
- 内容是否准确、最新？（过时文档比没有文档更危险）
- 格式是否适合检索？（PDF 扫描件、HTML 渲染页面、表格——处理复杂度不同）
- 结构是否适合分块？（条款型文档 vs 叙述型文档，分块策略完全不同）

**处理流程：**
```
原始文档
  ↓ 清洗（去除页眉页脚、格式噪声、重复内容）
  ↓ 结构化（表格提取、条款拆分、层级关系保留）
  ↓ 分块（按语义单元，不按字数）
  ↓ 元数据标注（来源、日期、版本、适用范围）
可检索的知识库
```

**治理重点：** 文档更新时，对应的 chunk 必须同步更新。过时知识比没有知识更危险——Agent 会用过时信息给出自信的错误答案。

---

### 类型 2：行为标注数据（给分类器/Fine-tuning 用）

**是什么：** (输入, 期望输出) 对，用于训练分类器或 Fine-tuning 模型。

**来源优先级（从成本最低到最高）：**

**优先级 1：已有专家判断记录**
现有系统中是否有人工判断记录？工厂质检的人工记录、客服的人工审核记录、医疗的专家诊断……这些已经是标注数据，只需要转换格式。

**优先级 2：Zone B 确认记录（上线后自动积累）**
每次 Zone B 人工确认，都是一条标注数据——Agent 建议 + 人工决策。这是最低成本的持续标注来源，零额外工作量。**设计 Zone B 流程时必须同时设计数据捕获**。

**优先级 3：专家构造（冷启动必用）**
无历史数据时，请领域专家直接构造典型案例。构造原则：
- 覆盖所有核心场景，不只是最常见场景
- 主动构造边界案例（正确答案不明显的情况）
- 构造失败模式（Agent 容易犯错的情况）
- 每个案例附带判断依据（不只是标签，还有原因）

**优先级 4：数据增强（谨慎使用）**
对现有案例做变体——改变表达方式、引入同义词、调整语境。可以扩大数量，但不能替代真实多样性。过度依赖增强数据会导致模型在真实分布上泛化差。

---

### 类型 3：评估数据（给 Eval 用，必须独立）

评估集是衡量 Agent 是否在进步的唯一客观标准。它的完整性和独立性决定了整个开发过程的可靠性。

**构建原则：**

**原则 1：按实体划分，不按时间或随机划分**
发动机质检场景：按发动机 SN 划分。客服场景：按用户 ID 划分。保险场景：按保单号划分。

为什么？同一台发动机的多次检测数据存在相关性——如果按随机划分，相关数据可能同时出现在训练集和评估集，造成数据泄露，评估结果虚高。

**原则 2：评估集必须覆盖失败模式，不只是正常路径**
只有正常案例的评估集，只能告诉你 Agent 在正常情况下表现如何，无法告诉你边界情况的表现。按以下比例构建：
- 60%：正常路径（高频、典型场景）
- 25%：边界案例（接近判断边界的情况）
- 15%：已知失败模式（Agent 曾经出错的类型，改进后是否修复）

**原则 3：评估集不参与任何优化**
Prompt 调整、检索参数调优、阈值设定——都不能看着评估集来做。一旦评估集被用于优化决策，它就失去了衡量进步的能力，变成了另一个训练集。

---

## 冷启动数据策略（最常见的困境）

**困境：** Agent 需要数据来工作，但数据只来自 Agent 工作后的积累。

**破解路径：**

```
第 1 周：领域专家构造 50-100 条（输入 + 理想输出）
    ↓
第 2 周：用构造数据验证技术管道（不是验证 Agent 质量）
    ↓
第 3-4 周：影子模式上线——Agent 给建议，人工做最终决策，所有数据记录
    ↓
第 5 周：审核前 200 条真实交互，选出 50 条作为评估集种子
    ↓
第 6 周起：Zone B 记录自动进入训练候选池，人工每周审核 30-50 条
```

**关键：** 影子模式不只是技术验证，更是数据积累的启动期。每次人工判断都是一条有价值的标注数据，必须从第一天就捕获。

---

## 数据治理的最小可行方案

治理不需要从一开始就很复杂，但有三件事从第一天就必须做：

**1. 训练集和评估集物理分离**
不是逻辑上"我知道哪些是评估集"，而是放在不同目录、不同数据库，访问权限不同。开发过程中需要看评估集性能时，通过 eval 脚本访问，不直接查看数据。

**2. 每条标注记录都有来源元数据**
```json
{
  "id": "case_20260401_001",
  "input": "...",
  "expected_output": "...",
  "label": "pass",
  "label_reason": "...",
  "annotator": "expert_01",
  "annotation_time": "2026-04-01T10:30:00",
  "source": "zone_b_confirmation",
  "split": "eval"
}
```
没有元数据的标注数据，六个月后没有人知道它从哪里来、为什么这样标、还是否有效。

**3. 数据集版本控制**
每次发布新模型版本，记录使用的训练集版本和评估集版本。这样才能在出问题时追溯：是数据变了、还是模型变了、还是两者都变了。Git 管理数据集元数据（不是数据本身），数据本身用 DVC 或简单的带版本号的目录结构管理。

---

## 常见的数据陷阱

**陷阱 1：把"数据量大"和"数据质量高"混淆**
一万条未经审核的历史日志，不如一百条经过专家仔细标注的案例。数据量解决不了数据质量问题。先把一百条做好，再扩展。

**陷阱 2：标注维度太多**
一开始就设计十个标注维度，标注者难以保持一致，标注速度极慢，数据很快积压。从两个维度开始（整体质量 + 失败类型），其他维度当数量和流程稳定后再加。

**陷阱 3：评估集从不更新**
评估集在第一个月构建，然后用了一年。但 Agent 的任务分布变了，用户的提问方式变了，评估集却还是原来的。季度评审评估集是非可选项。

**陷阱 4：领域专家只标注容易的案例**
专家倾向于构造自己熟悉的、答案明确的案例。边界案例、自己不确定的案例，往往被回避。这会导致评估集过于乐观。明确要求：每构造三个正常案例，构造一个边界案例。

**陷阱 5：Zone B 数据不捕获**
Zone B 确认是最自然的标注数据来源，但很多系统只记录最终结果（放行/拦截），不记录 Agent 的原始推理和人工修改。丢失了最有价值的改进信号。Zone B 数据捕获必须在系统设计阶段就规划，不能事后加。

---

## Related Skills

- `agent-eval-framework` — 使用本 Skill 建立的数据运行 eval
- `knowledge-injection-strategy` — 数据盘点决定知识注入策略的可行性
- `iteration-flywheel` — 持续数据采集和标注是飞轮的燃料
- `agent-boundary-design` — Zone B 设计决定了哪些数据会自动被捕获
