---
name: agent-boundary-design
description: Define what an agent owns, delegates, and refuses. Use when scoping a new agent, splitting a complex agent into multiple agents, or debugging low user trust caused by the agent doing too much or too little.
---

# Agent Boundary Design


## When to Use

- Designing a new agent and need to decide what it owns vs. what it delegates to humans or other agents
- An existing agent is doing too much and users don't trust it, or too little and users don't find it valuable
- Splitting a complex single agent into multiple specialized agents
- Debugging low approval rates or high override rates in a HITL-enabled product

## Ask These Questions First

1. What is the single job this agent is hired for? (Write it as a job story: When X, I want the agent to Y, so I can Z.)
2. What are the three highest-stakes actions the agent could take? Score each on reversibility, error cost, and model confidence.
3. Where is the trust boundary with the user? What can it do without asking (Zone A), what needs confirmation (Zone B), what must a human own (Zone C)?
4. What are the explicit handoff conditions — to a human, to another agent, and back to the agent after human input?

## Output Format
```markdown
## Agent Boundary Spec

**Agent 名称：** [填入]
**版本：** v1.0  日期：[填入]

**Job Story：**
当 [触发情境]，我希望 Agent 完成 [单一动作]，这样我可以 [真正关心的结果]。

**Zone A — 自主执行（无需人工介入）：**
- [动作 1]
- [动作 2]

**Zone B — 需要人工确认：**
| 动作 | 触发条件 | UX 模式 |
|------|---------|---------|
| [动作] | [条件] | Preview+Confirm / Edit+Confirm / Explain+Confirm |

**Zone C — 拒绝执行，强制人工：**
- [场景 1]
- [场景 2]

**Handoff → 人工：**
- 触发条件：[填入]
- 传递内容：[填入]
- 通知方式：[填入]
- 等待策略：[填入]

**Handoff → 恢复 Agent：**
- 恢复入口：[填入]
- 幂等保护：[填入]

**成功指标：**
| 指标 | 目标值 | 含义 |
|------|--------|------|
| [指标 1] | [值] | [含义] |

**上线前必须确认：**
- [ ] [风险项 1]
- [ ] [风险项 2]
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 2「Agent 设计」的核心入口，是所有后续设计文档的锚点。

**常见联动：**
- `system-prompt-engineering` — 把边界写进 SOUL 约束层
- `human-in-the-loop-design` — 为 Zone B 设计确认体验
- `trust-and-autonomy-tradeoff` — 定义边界如何随信任演化

---

---

## The Core Mental Model: The Agency Spectrum

Every agent sits somewhere on this spectrum:

```
COPILOT ←————————————————————————————→ AUTOPILOT

Suggests    Drafts    Executes    Decides    Acts fully
(human      (human    (human      (human     autonomously
confirms    edits)    approves)   informed)
everything)
```

**The right position is not the rightmost you can get away with.**

It's the position where:
- The model's error rate × the cost of that error = acceptable
- The user's trust in the system ≥ the autonomy granted
- The task's reversibility ≥ the autonomy granted

> **Founder pattern from 一个匿名的垂直行业 AI 搭档产品:** We initially designed the vertical-domain professional copilot to execute data lookups AND draft client communications AND suggest follow-up actions — all in one agent. It felt powerful. In practice, professionals didn't trust it because they couldn't tell *which part* was doing *what*. We split into two layers: a lookup/retrieval agent (full autopilot — professionals trusted the data) and a drafting agent (copilot — professionals always edited). Trust went up. Usage went up. This is the boundary insight: **split at the trust boundary, not the capability boundary.**

---

## The Four Boundary Questions

Before designing any agent, answer these four questions in order. Don't skip ahead.

### Q1: What is the single job this agent is being hired for?

Write it as a job story:

> **When** [situation], I want the agent to [one action], so I can [outcome I actually care about].

If you need more than one "I want the agent to" clause — you have multiple agents, not one.

**Anti-pattern:** "I want the agent to research prospects, draft outreach, schedule follow-ups, and track responses." That's a pipeline of four agents, not one agent.

**The test:** Can a new team member understand what this agent does from a single sentence? If not, the boundary is wrong.

---

### Q2: What are the three highest-stakes actions this agent could take?

For each action, score:

| Action | Reversibility (1–5) | Error cost (1–5) | Model confidence (1–5) |
|--------|--------------------|-----------------|-----------------------|
| ...    | ...                | ...             | ...                   |

**Reversibility:** 5 = trivially undoable (suggest text), 1 = permanent (send email, delete record, charge card)

**Error cost:** 5 = catastrophic (wrong medical info, wrong financial advice), 1 = minor inconvenience

**Model confidence:** 5 = high-accuracy structured task, 1 = ambiguous judgment call

**Decision rule:**
- If Reversibility < 3 AND Error cost > 3 → this action requires **human confirmation** (move it outside the agent's autonomous boundary)
- If Model confidence < 3 → this action requires **human input** (the agent should ask, not decide)
- Everything else → the agent can own it

---

### Q3: Where is the trust boundary with the user?

Trust is not binary. Map it explicitly:

**Zone A — Full trust (agent acts, user notified)**
> e.g., "Format this document", "Look up this policy number", "Translate this message"

**Zone B — Partial trust (agent drafts, user confirms)**
> e.g., "Send this message to the client", "Schedule this meeting", "Update this record"

**Zone C — No trust yet (agent suggests options, user decides)**
> e.g., "Recommend this product to this client", "Flag this claim as suspicious", "Escalate this case"

**The mistake most builders make:** They design Zone A and Zone C well, then leave Zone B implicit. Zone B is where the product either builds or destroys trust. Design it explicitly.

Zone B UX patterns:
- **Preview + confirm** — show the action, ask for one-click approval
- **Edit + confirm** — show a draft, let user modify before approving
- **Explain + confirm** — show the action AND why the agent took it, then confirm

---

### Q4: What are the explicit handoff points?

An agent that can't gracefully hand off will eventually do something wrong and have no way to recover.

Define these three handoffs before you build:

**Handoff to human:** When does the agent say "I can't handle this, a human should"?
- Trigger conditions (specific inputs, uncertainty thresholds, topic boundaries)
- What context does it hand over? (don't make the human start over)
- What channel? (in-chat, escalation ticket, async notification)

**Handoff to another agent:** If multi-agent, what is the contract between agents?
- What data does Agent A hand to Agent B? (define the schema explicitly)
- Who owns the conversation state?
- What happens if Agent B fails?

**Handoff back to the agent:** Can the agent resume after human intervention?
- Re-entry point in the workflow
- How does it incorporate the human's input?

> **Founder pattern from Shrimper:** When we built the multi-channel agent platform, the biggest architectural mistake was not defining the handoff contract between the routing layer and the worker agents. Each agent assumed it was getting a clean, structured context object — but the routing layer was passing raw message payloads. We had to add an adapter layer. **Define handoff contracts as explicit data schemas before you build, not after.**

---

## The Single-Agent vs Multi-Agent Decision

Use this decision tree:

```
Does the job require fundamentally different capabilities
(e.g., retrieval + reasoning + generation + action)?
│
├─ YES → Are these capabilities independent enough that
│        errors in one don't cascade to others?
│        │
│        ├─ YES → Multi-agent is probably right
│        │        (but start with one, split when you feel
│        │         the seams)
│        │
│        └─ NO → Keep single-agent, use tool-calling
│                to extend capabilities
│
└─ NO → Single agent. Full stop.
         (Multi-agent for its own sake adds latency,
          complexity, and failure modes with zero benefit)
```

**When to split into multiple agents:**

1. **Trust split:** Different parts of the job have radically different trust requirements (see Q3 above)
2. **Latency split:** One part is fast (retrieval), one is slow (generation) — parallelize
3. **Specialization split:** One part needs deep domain context that would bloat the other's system prompt
4. **Auditability split:** Compliance requires a clear audit trail between retrieval and recommendation

**When NOT to split:**

- "It feels cleaner" — complexity is a cost, not an aesthetic
- "The model will be better at one thing" — use tool-calling instead
- "It's easier to test in isolation" — integration testing of multi-agent is harder than unit testing of a single agent with mocks

---

## The Boundary Design Output

After running through the four questions, produce this one-page spec:

```markdown
## Agent Boundary Spec: [Agent Name]

**Job Story:**
When [situation], I want [agent] to [one action], so I can [outcome].

**Autonomous boundary (Zone A):**
- [Action 1]
- [Action 2]

**Confirmation boundary (Zone B):**
- [Action] → [UX pattern: preview/edit/explain + confirm]

**Out of scope — requires human (Zone C):**
- [Action / topic / input type]

**Handoff to human:**
- Trigger: [condition]
- Context passed: [data]
- Channel: [how]

**Handoff to other agents (if applicable):**
- Agent: [name] | Trigger: [condition] | Contract: [data schema]

**Success metric:**
How will we know the boundary is right? [measurable signal]
```

---

## Common Anti-Patterns

**The Feature Creep Agent**
Every new requirement gets added to the same agent. The system prompt grows to 10,000 tokens. The agent becomes unpredictable. The fix: every new requirement must pass the Q1 test. If it doesn't fit the job story, it's a new agent.

**The Permission Debt Agent**
The agent is built with minimal permissions "for now", with a plan to add more later. Users hit walls constantly. Trust erodes before it's built. The fix: design the permission model before building, not after.

**The Silent Failure Agent**
The agent hits its boundary and silently does nothing (or worse, hallucinates an answer). The fix: design explicit failure modes. Every boundary has an "I can't do this, here's why, here's what you can do instead" response.

**The Infinite Loop Agent**
In multi-agent systems, Agent A calls Agent B which calls Agent A. The fix: agents never call agents that called them. Draw the call graph before building.

---

## Worked Example: Vertical-Domain Professional Copilot

**Initial (wrong) design:** One agent handling information retrieval, communication drafting, judgment-heavy compliance checking, and option comparison — all in one.

**Problem discovered:** Users trusted the retrieval component (structured data, high accuracy) but didn't trust the compliance-checking component (judgment call, high stakes). They stopped using the whole product because they couldn't trust one part of it.

**The insight:** The trust boundary doesn't follow capability lines. It follows *stakes × model confidence* lines. Retrieval is high-confidence and low-stakes-per-query. Compliance judgment is low-confidence and high-stakes. These cannot share a boundary.

**Redesigned boundary:**

| Agent | Job | Zone A | Zone B | Zone C |
|-------|-----|--------|--------|--------|
| Retrieval Agent | Fetch structured domain data | All data lookups | — | Ambiguous queries |
| Drafting Agent | Draft outbound communications | Format, translate | Send, schedule | Judgment-heavy language |
| (Human) | High-stakes judgment calls | — | — | All compliance / risk decisions |

**Result:** Each agent had a clear, trustable boundary. Users engaged autonomously with retrieval, reviewed drafts quickly, and retained full control of high-stakes decisions. Trust built incrementally — the pattern that always works.

---

## Related Skills

- `system-prompt-engineering` — Once the boundary is defined, how to encode it in a system prompt
- `human-in-the-loop-design` — Designing the Zone B UX patterns in detail
- `agent-eval-framework` — How to measure whether the boundary you chose is correct
- `multi-agent-orchestration` — Detailed patterns for multi-agent handoff contracts
