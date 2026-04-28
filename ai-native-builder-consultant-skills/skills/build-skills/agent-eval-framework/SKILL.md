---
name: agent-eval-framework
description: Build a three-layer eval system: vibe check, golden dataset, production signals. Use when your agent 'seems to work' but you can't quantify it, before deciding to ship, or diagnosing why users are churning.
---

# Agent Eval Framework


## When to Use

- Pre-launch: need to know if the agent is ready to ship
- Post-launch: users are churning or complaining but you can't pinpoint why
- After a prompt change: need to confirm it didn't regress something that was working
- The agent 'seems fine' in testing but you have no systematic way to verify that claim

## Ask These Questions First

1. What maturity level are you at now — no eval (Level 0), vibe check only (Level 1), or do you have a golden dataset?
2. What are the three most important things this agent must get right? (These anchor the golden dataset.)
3. What production signals do you currently have access to — approval rates, edit rates, explicit feedback, conversation logs?
4. What counts as a regression? Define the threshold before running evals, not after.

## Output Format
```markdown
## Agent Eval 计划

**Agent：** [填入]  **版本：** [填入]

### 当前 Eval 成熟度

- [ ] Level 0：无系统 eval
- [ ] Level 1：Vibe Check 已建立
- [ ] Level 2：Golden Dataset 已建立
- [ ] Level 3：生产监控已接入

**本阶段目标成熟度：** Level [填入]

### Layer 1 — Vibe Check（15 条对话）

| 类型 | 数量 | 举例 |
|------|------|------|
| 核心正常路径 | 6 | [填入] |
| 边界案例 | 5 | [填入] |
| 失败模式 | 4 | [填入] |

### Layer 2 — Golden Dataset

- 规模目标：[填入] 条
- 构建方法：人工标注 / 历史数据 / 用户反馈挖掘
- LLM-as-Judge 提示词：[是否已校准，校准相关系数]

**回归阈值（部署前必须达到，否则不上线）：**

| 指标 | 最低要求 | 预警阈值 |
|------|---------|---------|
| 任务完成率 | [填入]% | [填入]% |
| 输出质量均分 | [填入] / 5 | [填入] / 5 |
| 边界遵守率 | [填入]% | [填入]% |

### Layer 3 — 生产信号

| 信号 | 采集方式 | 目标值 | 预警值 |
|------|---------|--------|-------|
| Zone B 触发率 | 系统日志 | < [填入]% | > [填入]% |
| 人工覆盖放行率 | 审批日志 | < [填入]% | > [填入]% |
| 编辑距离 | 前端埋点 | < [填入]% | > [填入]% |
| 用户回访率 | 产品埋点 | > [填入]% | < [填入]% |

### 每周 Production Review 议程（30 分钟）

1. 拉取上周生产信号，与目标值对比
2. 读 20 条随机对话
3. 读全部负面反馈对话
4. 本周唯一行动项：[填入]
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 3「数据与 Eval 基础」的第二个核心技能，也是上线前和上线后的共同验证底座。

**常见联动：**
- `data-foundation` — 没有数据基础就没有正式 eval
- `launch-risk-review` — 上线评审直接依赖这里定义的验证结果
- `iteration-flywheel` — 生产改进节奏建立在 eval 基础之上

---

## 核心前提

"It seems to work well in testing" is not an eval framework. It's a prayer.

The difference between teams that iterate quickly and teams that thrash is almost always eval infrastructure. Without it, every change feels uncertain, production incidents are surprises, and you can't distinguish "the model got worse" from "the product framing got better."

This skill gives you a three-layer eval system you can build in a week and run continuously. It is deliberately practical — designed for a small team shipping a real product, not an ML research lab with infinite resources.

---

## 数据前置条件

**在运行任何 eval 之前，必须先完成 `data-foundation`。**

`agent-eval-framework` 假设你已经有：
- 可用的标注数据（训练集 + 评估集，按实体划分，物理分离）
- 评估集已覆盖正常路径、边界案例和已知失败模式
- 每条标注记录有来源元数据（标注者、时间、来源、原因）

如果这些不存在，先运行 `data-foundation`，再回来。
在没有数据的情况下建 eval 框架，是在建一个没有燃料的引擎。

---
## Why Agent Eval Is Different From Model Eval

Standard ML eval asks: "How accurate is this model on this benchmark?"

Agent eval asks four harder questions:

1. **Does it do the right thing?** (task completion)
2. **Does it do the thing right?** (output quality)
3. **Does it know what it doesn't know?** (uncertainty calibration)
4. **Does it build or destroy user trust over time?** (production signals)

Benchmarks answer question 1 poorly and questions 2–4 not at all. You need a different approach.

---

## The Three Eval Layers

```
┌─────────────────────────────────────────┐
│  Layer 3: Production Signals            │  Real users, real behavior
│  What actually happens in the wild      │  Continuous, automated
├─────────────────────────────────────────┤
│  Layer 2: Structured Eval               │  Your test suite
│  What you think should happen           │  Run before every deploy
├─────────────────────────────────────────┤
│  Layer 1: Vibe Check                    │  Your intuition
│  Does it feel right?                    │  Run while developing
└─────────────────────────────────────────┘
```

All three layers are necessary. They catch different things.

**Layer 1** catches regressions you'd notice immediately — the agent started responding in English when it should respond in Chinese, the tone went weird, something obviously broke.

**Layer 2** catches regressions you wouldn't notice without testing — the 15th edge case in a specific scenario, a prompt change that improved 80% of cases and broke 20%.

**Layer 3** catches things you couldn't have anticipated — the interaction pattern your users actually have (vs. what you designed for), the trust erosion that only shows up after 3 weeks.

---

## Layer 1: The Vibe Check

This is not rigorous. It is necessary.

Before every significant prompt change or capability addition, run 10–15 representative conversations manually. You are checking:

- Does the agent's personality feel consistent?
- Does the tone match the context?
- Does it handle edge cases with grace (wrong answers delivered gracefully are better than right answers delivered badly)?
- Are there any new failure modes from the change?

**The vibe check conversation set** — maintain 15 conversations in a document. Update it as you learn what matters:

```markdown
## Vibe Check Conversations

### Normal use cases (6 conversations)
1. [Most common user request type]
2. [Second most common]
3. [Third most common]
4. [A request that requires multi-step reasoning]
5. [A request with ambiguous intent]
6. [A request that requires retrieving domain knowledge]

### Edge cases (5 conversations)
7. [A request the agent should handle but is tricky]
8. [A request at the boundary of the agent's scope]
9. [A request that requires admitting uncertainty]
10. [A request with a subtle error the user needs to be corrected on]
11. [An adversarial input — user trying to get the agent to do something it shouldn't]

### Failure modes (4 conversations)
12. [A request completely outside scope]
13. [A request with missing context the agent needs]
14. [A request in an unexpected format or language]
15. [A multi-turn conversation that changes direction mid-way]
```

This takes 20–30 minutes per run. Do it. Skipping it because you're in a hurry is how you ship embarrassing regressions.

---

## Layer 2: Structured Eval

### Building the Golden Dataset

A golden dataset is a set of (input, expected output) pairs you can run automatically and score.

The hardest part is building it without pre-existing labeled data. Here's how:

**Method 1: Expert annotation of production inputs**
Once you have real users, export 200 conversations. Have a domain expert (ideally you, in the early stage) label each as: good / acceptable / bad, with a one-line note on what made it good or bad. This is your first golden set.

**Method 2: Synthetic generation from failure modes**
For each known failure mode (from your vibe check or production incidents), generate 10 variations. "The agent gave a wrong answer when asked about X" → generate 10 different phrasings of X-type questions, label the correct responses. This covers your known blind spots systematically.

**Method 3: User feedback mining**
If you have a thumbs-up/thumbs-down in your product, the thumbs-down conversations are gold. They represent actual failures your users cared enough to report. Build a golden set from them — each thumbs-down becomes a test case for the fixed behavior.

**Golden dataset size targets:**

| Stage | Size | Coverage goal |
|-------|------|---------------|
| MVP pre-launch | 50 | Core happy paths only |
| Post-launch v1 | 200 | Happy paths + known edge cases |
| Scaling | 500+ | + adversarial cases + failure mode coverage |

Don't wait for a "large enough" dataset. 50 test cases run automatically beats 1,000 test cases you never built.

---

### What to Measure

**Task completion rate**
Did the agent do what was asked? Binary per test case.

Simple to score for structured tasks (did the agent retrieve the right product? did the SQL query return the right result?). Hard to score for open-ended tasks. For open-ended tasks, use LLM-as-judge (below).

**Output quality score**
On a 1–5 scale, how good was the response? Dimension-specific:
- Accuracy (did it get the facts right?)
- Completeness (did it cover what it needed to?)
- Tone (did it sound right for this context?)
- Conciseness (did it waste the user's time?)

You can score this manually during dataset building, then use LLM-as-judge to score new cases.

**Uncertainty calibration**
When the agent says it's confident, is it right? When it expresses uncertainty, is the uncertainty warranted?

Track: (cases where agent expressed confidence) → accuracy rate. It should be high.
Track: (cases where agent expressed uncertainty) → how often it was actually uncertain. It should also be high.

An agent that expresses confidence about everything and is wrong 30% of the time is miscalibrated. An agent that expresses uncertainty about everything is useless. You want: high confidence → high accuracy, uncertainty expressed → actually uncertain.

**Boundary adherence**
What fraction of out-of-scope requests were correctly redirected rather than hallucinated answers given?

Build 20 out-of-scope test cases. Run them. You should see 100% correct redirects. If not, your scope constraints need work.

---

### LLM-as-Judge

For open-ended outputs where automated scoring is hard, use another LLM call as the judge.

**The judge prompt template:**

```
You are evaluating the quality of an AI agent's response. 

Context:
- Agent's role: [description]
- User's request: [input]
- Agent's response: [output]

Score the response on each dimension (1-5):

ACCURACY: Are the facts correct? Any hallucinations?
(1=multiple factual errors, 3=minor issues, 5=completely accurate)

HELPFULNESS: Does this actually help the user?
(1=unhelpful/irrelevant, 3=partially helpful, 5=directly addresses the need)

TONE: Is the tone appropriate for this agent's role and this context?
(1=clearly wrong tone, 3=acceptable, 5=exactly right)

SAFETY: Does the response stay within appropriate boundaries?
(1=clear violation, 3=borderline, 5=fully appropriate)

For any score < 4, explain what was wrong in one sentence.

Respond in JSON: {"accuracy": N, "helpfulness": N, "tone": N, "safety": N, "notes": "..."}
```

**LLM-as-judge caveats:**

- It's not ground truth — it's a cheap, scalable proxy for human judgment
- It has known biases (tends to prefer longer responses, more authoritative tone)
- Calibrate it against your own scores on 50 cases before trusting it at scale
- Use it to flag candidates for human review, not as a final verdict on nuanced cases

> **Founder pattern from a production copilot:** We used an LLM judge for output quality scoring, with one calibration step that made a huge difference: we first scored 100 production conversations manually, then compared our scores to the judge's scores. Where they diverged, we updated the judge prompt to encode our actual standards, not the model's default standards. Calibrated LLM-as-judge correlated substantially better with human judgment. The calibration step takes a few hours and is worth it every time.

---

### Running the Eval

```bash
# Minimal eval pipeline structure
eval/
  golden/          # your test cases as JSON
  runner.py        # calls your agent API with each input
  scorer.py        # computes scores (exact match + LLM judge)
  report.py        # generates summary report
  baseline.json    # last known-good scores (your regression threshold)
```

**The regression threshold rule:** Define what counts as a regression before you run evals, not after. Example:

```json
{
  "task_completion_rate": {"min": 0.85, "alert_below": 0.80},
  "output_quality_avg": {"min": 3.8, "alert_below": 3.5},
  "boundary_adherence": {"min": 0.95, "alert_below": 0.90},
  "uncertainty_calibration": {"min": 0.80, "alert_below": 0.75}
}
```

If any metric drops below `alert_below`, the deploy is blocked. If it drops below `min` from baseline, it's a production incident.

---

## Layer 3: Production Signals

This is where you learn things you couldn't anticipate. The two most important signals:

### Signal 1: Behavioral Signals (What Users Do)

| Signal | What it tells you |
|--------|------------------|
| **HITL approval rate** | Are agent recommendations actually good? |
| **HITL edit rate** | Where does the agent need quality improvement? |
| **HITL override rate** | Where does agent judgment conflict with user judgment? |
| **Conversation abandonment rate** | Where do users give up? |
| **Session depth** | Are users actually engaging, or getting one answer and leaving? |
| **Return rate** | Are users coming back? (The ultimate trust proxy) |

Track these by **action type**, not in aggregate. Aggregate approval rates hide the failure modes. "85% approval" might mean "100% approval for simple lookups, 50% approval for recommendations" — and those are completely different problems.

### Signal 2: Sentiment Signals (What Users Say)

- Explicit feedback (thumbs up/down, ratings)
- Implicit feedback (did they follow the agent's recommendation? did they edit heavily?)
- Support tickets that mention the agent
- User interviews (non-scalable but irreplaceable — do at least 2 per month in early stage)

> **Founder pattern from Shrimper:** The most valuable production signal we found wasn't approval rate — it was **edit distance on confirmed drafts**. Low edit distance = agent drafts are close to what users want. High edit distance = agent is producing starting points, not finished work. When edit distance went up on outreach message drafts, we traced it to a prompt change that made the tone more formal. Users were "accepting" the drafts (approval rate looked fine) but then editing them back to informal. The approval rate was a lagging indicator; edit distance was a leading indicator of dissatisfaction.

---

### The Weekly Production Review

30 minutes, every week, until the product is mature:

1. Pull last week's behavioral metrics — flag anything outside normal range
2. Read 20 random conversations from last week — just read them, no scoring
3. Read all conversations with negative feedback — every single one
4. One action item: what's the single highest-leverage thing to fix or test this week?

The conversation reading is the most important part. Metrics tell you what's happening. Conversations tell you why.

---

## The Eval Maturity Model

**Level 0 (shipping blind):** No systematic eval. Changes are made based on feel. Production incidents are discovered by users.

**Level 1 (vibe check):** 15 test conversations run manually before major changes. Catches obvious regressions.

**Level 2 (golden dataset):** 50–200 test cases run automatically before every deploy. Catches non-obvious regressions. Takes ~1 week to build.

**Level 3 (production monitoring):** Behavioral signals tracked continuously. Weekly review cadence. Takes ~2 weeks to instrument.

**Level 4 (eval-driven iteration):** Product decisions are driven by eval results. Fine-tuning and prompt changes are measured against baselines. Takes 1–2 months to reach.

Get to Level 2 before launch. Get to Level 3 within the first month after launch. Level 4 is a competitive advantage — few teams get there.

---

## Common Mistakes

**Optimizing for one metric**
Your agent achieves 95% task completion but users stop coming back after week 2. You were measuring the wrong thing. Always track the return rate as a sanity check on all other metrics.

**Eval set drift**
You built your golden dataset in month 1. It's now month 8. Your users have changed, your product has changed, but your eval set hasn't. Stale eval sets give you false confidence. Review and update your golden set quarterly.

**Testing only happy paths**
Your eval set has 50 normal cases and 0 adversarial cases. A user finds an edge case on day 1 that makes the agent behave badly. Build adversarial cases into your golden set from the start.

**Conflating model quality with product quality**
The agent's output is good but users aren't approving it. This is a product problem (the HITL UX, the trust calibration, the onboarding) — not a model problem. Eval needs to measure both separately.

**The perfect eval fallacy**
Waiting to launch until you have a comprehensive eval framework. You need production data to build a good eval framework. Launch with Level 1, build Level 2 in the first two weeks, instrument Level 3 in the first month. Perfection is the enemy.

---

## Related Skills

- `agent-boundary-design` — Defines what you're evaluating (the scope of the agent)
- `system-prompt-engineering` — The thing you'll be iterating on based on eval results
- `human-in-the-loop-design` — The source of your richest behavioral signals
- `iteration-flywheel` — How to close the loop from eval results back into product improvements
