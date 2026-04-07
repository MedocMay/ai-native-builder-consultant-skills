---
name: trust-and-autonomy-tradeoff
description: Calibrate how much autonomy to grant an AI agent as user trust develops. Use when deciding how aggressive automation should be, when users ignore agent actions, or when autonomy is eroding trust.
---

# Trust and Autonomy Tradeoff

## When to Use

- Deciding how much the agent should do autonomously vs. ask for confirmation at launch
- Users are rubber-stamping agent actions without reading — autonomy is too low to be meaningful
- Users are overriding or abandoning agent actions — autonomy has outrun trust
- Planning the roadmap from "agent assists" to "agent leads" and need a principled sequencing
- After an incident where an autonomous agent action caused an unintended outcome

## Ask These Questions First

1. How long have users been using this product, and what is their current approval rate for agent actions? (Trust level determines where on the spectrum to start.)
2. What is the cost of an autonomous action error — reversible inconvenience, or irreversible harm?
3. Have users explicitly requested more automation, or is the autonomy expansion driven by the team's desire to showcase capability?
4. What is the current edit rate on agent drafts? (High edit rate = agent quality doesn't yet justify more autonomy.)
5. Is trust eroding (declining approval rate, increasing overrides) or growing (approvals becoming faster, edits becoming lighter)?

## Output Format

```markdown
## Trust-Autonomy Calibration Plan

**Product:** [填入]  **Date:** [填入]  **Current stage:** Week 1 / Month 1 / Month 3+

### Current trust signals
| Signal | Current value | Healthy range | Status |
|--------|--------------|--------------|--------|
| Approval rate (by action type) | [填入]% | > 85% | 🟢 / 🟡 / 🔴 |
| Edit rate on drafts | [填入]% | < 20% | 🟢 / 🟡 / 🔴 |
| Override rate | [填入]% | < 2% | 🟢 / 🟡 / 🔴 |
| Time-to-decision | [填入]s | < 30s | 🟢 / 🟡 / 🔴 |

### Current autonomy level (per action type)
| Action | Current zone | Justification |
|--------|-------------|---------------|
| [Action 1] | Zone A / B / C | [why this level] |
| [Action 2] | Zone A / B / C | [why this level] |

### Autonomy expansion plan
| Action | From | To | Trigger condition | Timeline |
|--------|------|----|-------------------|---------|
| [Action 1] | Zone B | Zone A | Approval rate > 90% for 4 weeks | [month] |
| [Action 2] | Zone C | Zone B | [condition] | [month] |

### Autonomy reduction triggers (if trust erodes)
| Signal | Threshold | Response |
|--------|-----------|---------|
| Override rate | > 3% | Move highest-override action back to Zone B |
| Approval rate drop | > 10% week-over-week | Freeze all Zone A expansions, investigate |
| User complaint spike | > [N] per week | Immediate review, possible rollback |
```

## 咨询链位置

**在新版 SOP 中：** 模块 2「Agent 设计」里，负责把静态边界扩展成随时间演化的自治计划。

**常见联动：**
- `agent-boundary-design` — 静态边界是自治演化的起点
- `human-in-the-loop-design` — 信任校准决定 Zone B 体验的严格程度
- `agent-eval-framework` — 信任信号会变成生产监控指标

---

## The Core Model: Trust Is Earned, Not Assumed

The most common mistake in AI Native product design is launching with too much autonomy because the team is confident in the model's capability.

Capability and trustworthiness are different things. A model can be highly capable and still not trusted — because trust is a relationship property, not a model property. Trust develops through accumulated experience of the agent doing the right thing in situations the user can verify.

The practical implication: **start with less autonomy than the model deserves, and earn the right to expand.**

```
Trust level         Low ←——————————————→ High
                    │                        │
Autonomy granted    Low                     High
                    │                        │
Confirmation mode   Explain+Confirm    →   Notify only
```

## The Four Autonomy Stages

### Stage 0: Shadow mode (before any autonomy)
Agent runs in parallel with human process. Its outputs are logged and compared, but never acted on. Purpose: calibrate the model before users interact with it.

Duration: 2–4 weeks, or until 200+ data points with human comparison.
Exit condition: Agreement rate with human judgment ≥ 95%.

### Stage 1: Assisted (agent suggests, human decides everything)
Every agent action requires explicit human approval. Zone B for all consequential actions, Zone C for anything high-stakes.

Purpose: establish the baseline trust relationship. Users learn what the agent can do; the agent (in aggregate) reveals where it's reliable.

Duration: first 4 weeks of user-facing operation.
Exit condition: approval rate > 90% on at least two action types for two consecutive weeks.

### Stage 2: Collaborative (agent acts on high-confidence routine tasks)
High-confidence, reversible, routine actions move to Zone A. Novel situations, edge cases, and high-stakes actions remain Zone B/C.

The key signal: users are approving actions without reading them carefully. This means they've internalized what the agent does and trust it for that action type. This is the signal to expand autonomy — not a problem to fix.

Duration: months 2–4.
Exit condition: edit rate on Zone B drafts < 10% for four consecutive weeks on a given action type.

### Stage 3: Delegated (agent leads, human monitors)
Most actions Zone A. Human reviews exceptions (Zone B) and edge cases. Regular audit rather than per-action approval.

This stage is appropriate only when:
- The agent has been in Stage 2 for ≥ 3 months
- No significant incidents in the previous 60 days
- Override rate < 1% consistently
- Users are actively requesting reduced confirmation friction

## Reading the Trust Signals

**Approval rate by action type (not aggregate)**
Aggregate approval rate hides the failure modes. 90% aggregate might mean 99% approval on lookups and 60% approval on recommendations. These require completely different interventions.

Track approval rate per action type. The action types with lowest approval are where the agent's judgment is weakest or the framing is wrong.

**Edit rate on Zone B drafts**
Edit rate is a leading indicator. If users are approving Zone B drafts but heavily editing them, approval rate looks healthy while trust is actually eroding — users have learned to treat drafts as starting points, not finished work.

Low edit rate (< 10%) on a draft action type = ready for Zone A promotion.
High edit rate (> 40%) = agent quality problem that must be solved before any autonomy expansion.

**Time-to-decision on Zone B**
If users take > 2 minutes to decide on a Zone B confirmation, the information presented is too complex or the stakes feel too high. Slow decisions signal poor confirmation UX, not careful users.

**Override rate**
Override = user reversed an autonomous Zone A action after the fact. This is the most serious trust signal. Any override rate > 2% on a specific action type means autonomy has outrun trust for that action. Move it back to Zone B.

## The Autonomy Expansion Protocol

Never expand autonomy on multiple action types simultaneously. One action type at a time, with a 2-week observation period after each expansion.

```
Week 1-4:    Observe signal for target action type
Week 4:      Review: does it meet expansion criteria?
             Yes → expand this action type to Zone A
             No  → diagnose why and fix before re-evaluating
Week 4-6:    Observe the newly expanded action in Zone A
Week 6:      Review: any new problems? override rate acceptable?
             Yes → proceed to next action type
             No  → roll back to Zone B, diagnose
```

This cadence feels slow. It is slow. It is correct. Trust lost through a premature autonomy expansion takes months to rebuild. Trust built steadily through a careful protocol compounds.

## The Autonomy Reduction Protocol

When trust signals deteriorate, reduce autonomy immediately and visibly.

**Do not:**
- Quietly adjust thresholds in the background
- Wait for the next sprint cycle to address
- Attribute declining trust signals to "user behavior" rather than agent behavior

**Do:**
- Move the specific action type showing degradation back to Zone B in the same week
- Communicate the change to users ("We've added a confirmation step for X while we investigate")
- Investigate root cause before re-expanding

Transparent autonomy reduction builds more trust than invisible autonomy that occasionally fails.

## Related Skills

- `agent-boundary-design` — The Zone A/B/C framework that autonomy calibration operates within
- `human-in-the-loop-design` — The UX design of Zone B confirmation changes as trust evolves
- `agent-eval-framework` — Trust signals are production metrics; this skill defines what to measure
