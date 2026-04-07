---
name: iteration-flywheel
description: Build the data-model-product feedback loop that compounds agent quality over time. Use after launch when production data is accumulating but not being systematically used to improve the agent, or when improvement velocity has stalled.
---

# Iteration Flywheel

## When to Use

- The agent has launched and production data is accumulating, but improvements feel ad-hoc
- Improvement velocity has slowed — each change feels like guesswork
- Zone B trigger rate is stable but not improving, despite ongoing effort
- The team is making model/prompt changes without a systematic way to verify they worked
- A client asks "how do we keep improving this after you hand it off"

## Ask These Questions First

1. What data is currently being captured from production interactions? (If the answer is "not much," the flywheel can't spin.)
2. Who is responsible for reviewing production data, and how often? (If no one owns it, it won't happen.)
3. What has been the most common failure mode in the last 30 days? (This tells you where to focus first.)
4. When the team makes a change to the prompt or model, how do they know if it helped or hurt?
5. What is the current data labeling process — who labels, how fast, at what quality?

## Output Format

```markdown
## Iteration Flywheel Design

**Product:** [填入]  **Date:** [填入]

### Current state
| Dimension | Current | Target (3 months) |
|-----------|---------|-------------------|
| Labeled data volume | [N] examples | [N] examples |
| Labeling cadence | [ad-hoc / weekly / daily] | weekly |
| Eval run frequency | [ad-hoc / per PR / weekly] | per PR + weekly |
| Improvement cycle time | [how long from problem to fix] | < 1 week |
| Primary failure mode (last 30 days) | [填入] | — |

### Data pipeline

**What we capture:**
- [ ] Full interaction logs (input + output + metadata)
- [ ] Zone B events + human decisions
- [ ] Zone C events + escalation outcomes
- [ ] User edits on Zone B drafts (edit distance)
- [ ] Explicit feedback (thumbs up/down, if implemented)

**Labeling strategy:**
| Source | Volume | Label type | Owner | Cadence |
|--------|--------|-----------|-------|---------|
| Zone B human decisions | [N/week] | Correct/incorrect | QA team | Automatic |
| Random sample | [N/week] | Quality 1-5 | [role] | Weekly |
| Failure cases | All | Root cause | [role] | Weekly |

### Improvement cycle

| Step | Who | Frequency | Output |
|------|-----|-----------|--------|
| Production review | [role] | Weekly (30 min) | Top 3 issues |
| Root cause analysis | [role] | Per issue | Fix hypothesis |
| Change implementation | [role] | Per hypothesis | Prompt/model/retrieval change |
| Eval validation | [role] | Per change | Go/No-Go on regression threshold |
| Deploy | [role] | Per validated change | — |
| Monitor | [role] | 48h post-deploy | Confirm improvement, no regression |

### Flywheel metrics (track monthly)
| Metric | Month 0 | Month 1 | Month 2 | Month 3 |
|--------|---------|---------|---------|---------|
| Labeled dataset size | | | | |
| Golden dataset task completion rate | | | | |
| Zone B trigger rate | | | | |
| User override rate | | | | |
| Improvement cycle time (avg) | | | | |
```

## 咨询链位置

**在新版 SOP 中：** 模块 6「上线后飞轮」的核心技能，负责把生产数据变成稳定的改进节奏。

**常见联动：**
- `data-foundation` — 没有数据采集和标注管道，飞轮转不起来
- `agent-eval-framework` — 飞轮里的每次改动都需要 eval 验证
- `launch-risk-review` — 通过上线评审后，飞轮才正式进入持续运行阶段

---

## 数据前置条件

**飞轮能转的前提是有数据管道在持续输出燃料。**

在使用这个 Skill 之前，确认以下已经到位（来自 `data-foundation`）：
- Zone B 确认记录被自动捕获（每条记录含 Agent 原始输出 + 人工决策）
- 随机对话样本的采集管道已建立
- 评估集已构建并与训练数据物理分离
- 标注规范文档存在，标注者知道怎么用

如果数据管道不存在，飞轮的第一步（生产数据 → 分析）就无法执行。

---
## The Flywheel Model

Every AI agent deployed in production generates data. That data contains the information needed to improve the agent. The question is whether the organization has the process to extract that information and act on it.

Without a flywheel, improvement is random: someone notices something, files a ticket, a change gets made, no one knows if it helped.

With a flywheel, improvement compounds: production data → systematic labeling → eval validation → targeted fixes → better production behavior → better data.

```
Production data
      ↓
  Weekly review          ← 30 minutes, same day, every week
      ↓
  Root cause analysis    ← what category of failure is this?
      ↓
  Fix hypothesis         ← prompt change / retrieval change / model change / boundary change
      ↓
  Eval validation        ← does the fix pass regression? does it actually fix the problem?
      ↓
  Deploy
      ↓
  Monitor (48h)          ← did it work in production? any new regressions?
      ↓
Production data
```

## The Four Improvement Levers

When something is wrong, there are four places to intervene. Diagnosing which lever to pull before making changes prevents random thrashing.

**Lever 1: Prompt / SOUL change**
When: the agent has the right information and capability but behaves incorrectly — wrong tone, wrong boundary adherence, wrong uncertainty expression.
How to validate: run golden dataset before and after, confirm improvement in the specific failure category without regression elsewhere.
Risk: prompt changes can have non-obvious side effects. Always run the full eval, not just the cases you're trying to fix.

**Lever 2: Retrieval change**
When: the agent is hallucinating or giving outdated information — a retrieval quality problem, not a model problem.
How to validate: inspect retrieved context for failure cases. If the correct information isn't in the retrieved context, it's a retrieval problem. If it is, it's a model problem.
Common sub-issues: chunking strategy, query-document mismatch, stale index, missing metadata filtering.

**Lever 3: Boundary / Zone change**
When: Zone B trigger rate is persistently too high (model keeps hitting gray area) or too low (model is too confident on cases that warrant review).
How to validate: sample 20 Zone B cases from the past week. Are they genuinely ambiguous, or is the threshold miscalibrated?
Risk: loosening Zone B (fewer confirmations) expands autonomous risk. Always check override rate after any Zone A expansion.

**Lever 4: Data / training change**
When: the problem is systematic and consistent across many examples — the model has a genuine capability gap that prompt engineering can't fix.
How to validate: the failure persists across prompt variations; a fine-tuned model or different base model fixes it.
This is the most expensive lever. Exhaust levers 1-3 before pulling lever 4.

## The Weekly Production Review (30 Minutes)

This is the most important ritual in the iteration flywheel. Skipping it breaks the cycle.

**Agenda (fixed, don't change it):**

*Minutes 1-5: Metrics dashboard*
Pull Zone B trigger rate, override rate, error rate for the past week. Compare to previous week. Flag any metric that moved by > 20%.

*Minutes 6-15: Random conversation sample*
Read 10 random conversations from the past week. Not the worst ones — random. You are looking for: unexpected patterns, emerging use cases you didn't design for, conversations that worked well and might reveal what's going well.

*Minutes 16-25: Failure case review*
Read every conversation with negative feedback (thumbs down, Zone C escalation, user override). All of them. For each: is this a prompt problem, a retrieval problem, a boundary problem, or a data problem? Tag it.

*Minutes 26-30: One action item*
Choose exactly one thing to fix this week. Not three. Not five. One. The one with the highest impact relative to effort. Write it down with an owner and a deadline.

## Labeling Strategy

Labels are the fuel for the flywheel. Without labels, you have data but no improvement signal.

**The minimum viable labeling program:**

Zone B human decisions are already labels. When a user confirms or overrides a Zone B action, that's a signal. Capture it automatically. This requires no additional labeling effort and produces 50-200 labels per week for active products.

**The sustainable manual labeling program:**

One person, two hours per week, labeling 30-50 examples from the random conversation sample. Label format: quality (1-5) + failure category (prompt / retrieval / boundary / none). This is sufficient for monthly model retraining or prompt refinement.

**When to scale up:**

If improvement velocity plateaus despite weekly reviews and prompt changes, you need more labeled data. Scale to daily labeling sessions, consider annotation tooling, consider LLM-assisted pre-labeling with human review.

## LLM-as-Judge in the Flywheel

LLM-as-judge enables fast, scalable quality assessment without manual labeling for every case. Use it to:
- Score the full production sample weekly (not just the 30-50 manually labeled cases)
- Flag cases for human review (low scores) rather than reviewing everything manually
- Track quality trends over time at scale

**Calibration is mandatory.** Before trusting LLM-as-judge scores, manually score 50 cases and compare. If correlation is < 0.80, recalibrate the judge prompt. An uncalibrated judge produces noise, not signal.

**What LLM-as-judge cannot replace:**
- Root cause analysis (the judge can flag a problem, not diagnose it)
- Business judgment (whether a technically correct response serves the user's actual need)
- Novel failure mode detection (the judge is calibrated on known patterns; new failure modes require human review)

## Common Failure Modes of the Flywheel

**The flywheel that never starts**
No systematic data capture. Reviews happen "when there's time." Labels accumulate in a spreadsheet no one updates. Fix: designate one person, one fixed time slot per week. Non-negotiable. The flywheel runs on routine, not inspiration.

**The flywheel that spins but doesn't improve**
Reviews happen, issues are identified, changes are made, but improvement doesn't compound. Usually caused by: changes aren't validated against the eval before deployment, or the same failure modes recur because root cause isn't diagnosed (symptoms are treated, not causes).

**The flywheel that optimizes the wrong thing**
Team is laser-focused on reducing Zone B trigger rate, at the cost of increasing override rate (autonomy outrunning trust). Or optimizing task completion rate while quality degrades. Track all flywheel metrics together, not one in isolation.

**The flywheel with no fuel (no labels)**
Improvement requires labeled examples. A flywheel with no labeling program will plateau quickly. The team makes prompt changes based on intuition and a few cherry-picked examples. This is not a flywheel — it's random walk.

## Related Skills

- `agent-eval-framework` — The eval infrastructure the flywheel runs on
- `launch-risk-review` — The launch gate before the flywheel starts spinning
- `trust-and-autonomy-tradeoff` — Zone expansion decisions are part of the flywheel
- `knowledge-injection-strategy` — Retrieval changes are one of the four improvement levers
