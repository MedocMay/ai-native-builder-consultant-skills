---
name: launch-risk-review
description: Run a pre-launch risk review for an AI agent covering behavior, trust zones, failure modes, compliance, and rollback. Use before first launch or before expanding the agent's autonomy to a new action type.
---

# Launch Risk Review

## When to Use

- Before the first user-facing deployment of any AI agent
- Before expanding an agent's Zone A (autonomous actions) to cover new action types
- After a significant change to the system prompt, model, or retrieval pipeline
- When a client asks "are we ready to launch" and needs a structured answer
- Before handing off a delivered system to a client for production operation

## Ask These Questions First

1. Has the agent been through shadow mode with ≥ 200 real interactions compared against human judgment? What was the agreement rate?
2. What is the single worst thing that could happen if the agent makes an error on its most consequential Zone A action? Is that consequence reversible?
3. Is there a working rollback path — can you instantly revert to human-only operation without downtime?
4. Have real users (not the development team) interacted with the agent in a controlled setting? What surprised them?
5. For China deployments: has the content passed platform review requirements? Are data residency constraints confirmed?

## Output Format

```markdown
## Launch Risk Review

**Product:** [填入]  **Version:** [填入]  **Review date:** [填入]
**Reviewer:** [填入]  **Go / No-Go decision:** 🟢 Go / 🔴 No-Go / 🟡 Conditional Go

### Blockers (must resolve before launch)
| # | Issue | Owner | Deadline |
|---|-------|-------|---------|
| 1 | [填入] | [填入] | [填入] |

### Conditions (Conditional Go — must resolve within X days of launch)
| # | Condition | Owner | Deadline |
|---|-----------|-------|---------|
| 1 | [填入] | [填入] | [填入] |

### Checklist results
[See sections below — ✅ pass / ⚠ conditional / ❌ blocker]

### Rollback plan
- Trigger condition: [what makes you roll back]
- Rollback action: [exact steps, who executes]
- Recovery time: [how long to full human-only operation]
- Communication: [who gets notified, what they're told]
```

## 咨询链位置

**在新版 SOP 中：** 模块 5「上线准备」的核心关卡，在边界、数据、eval 和回滚方案都明确之后执行。

**常见联动：**
- `agent-eval-framework` — Golden Dataset 与生产信号为上线评审提供证据
- `trust-and-autonomy-tradeoff` — 决定上线时 Zone A 的范围是否合理
- `iteration-flywheel` — 上线通过后进入持续改进机制

---

## The Full Checklist

### Section 1: Model Behavior

**1.1 Vibe check passed**
- [ ] 20-case vibe check run against current model + system prompt version
- [ ] All 15 standard cases pass; edge cases reviewed and accepted

**1.2 Golden dataset regression**
- [ ] Task completion rate ≥ [target]%
- [ ] Output quality score ≥ [target] / 5
- [ ] Boundary adherence rate ≥ [target]%
- [ ] No regressions from previous version (if applicable)

**1.3 Adversarial inputs**
- [ ] Tested with inputs designed to bypass identity (prompt injection attempts)
- [ ] Tested with out-of-scope requests — agent redirects correctly, does not hallucinate answers
- [ ] Tested with contradictory instructions — agent follows SOUL, not user override

**1.4 Hallucination profile**
- [ ] Known hallucination-prone task types identified and either: (a) routed to RAG/tool, or (b) explicitly flagged as "I may be uncertain"
- [ ] Factual recall tasks have retrieval grounding, not parametric memory reliance

---

### Section 2: Trust Zones and HITL

**2.1 Zone A actions**
- [ ] Every Zone A action is reversible OR has been explicitly accepted as irreversible with stakeholder sign-off
- [ ] Zone A actions have audit logging (what was done, when, by which agent invocation)
- [ ] Zone A error rate in shadow mode was < [target]%

**2.2 Zone B flow**
- [ ] Zone B confirmation UI tested by ≥ 3 real users who are not on the development team
- [ ] All three options (confirm / cancel / alternative) are functional
- [ ] Timeout behavior is defined and tested (what happens if user doesn't respond)
- [ ] Idempotency: double-confirm does not double-execute

**2.3 Zone C escalation**
- [ ] Every Zone C trigger has a tested escalation path to a human
- [ ] The human receiving the escalation knows what to do with it (SOP exists)
- [ ] Zone C events are logged separately for review

---

### Section 3: Failure Modes

**3.1 Graceful degradation**
- [ ] If the LLM API is unavailable: agent fails gracefully, user is informed, no silent failures
- [ ] If retrieval/RAG fails: agent either falls back to known information with caveat, or declines and explains
- [ ] If a tool call fails: agent surfaces the failure explicitly, does not fabricate a result

**3.2 Cascade prevention (multi-agent only)**
- [ ] Maximum call depth enforced
- [ ] No cycles in call graph (verified by inspection or test)
- [ ] Worker agent failure does not crash router

**3.3 Rate limits and timeouts**
- [ ] LLM API rate limits understood; backoff and retry logic implemented
- [ ] Per-request timeout defined and tested (agent does not hang indefinitely)
- [ ] Queue or load shedding strategy defined for traffic spikes

---

### Section 4: Data and Privacy

**4.1 Data handling**
- [ ] Personal data that enters the agent context is not logged in plain text
- [ ] Conversation history retention policy defined and implemented
- [ ] Data that should not leave the organization does not leave (checked against retrieval pipeline)

**4.2 China-specific compliance (if applicable)**
- [ ] Data processed entirely within China (no cross-border data transfer)
- [ ] If using foreign model APIs: confirmed permissible under applicable regulations
- [ ] Content moderation: outputs reviewed for platform compliance (especially for consumer-facing products)
- [ ] ICP and relevant licenses confirmed for the deployment domain

**4.3 Audit trail**
- [ ] Every consequential action (Zone A and B) has a record: timestamp, input summary, output, agent version
- [ ] Audit records are tamper-evident (not editable by agent)
- [ ] Retention period defined

---

### Section 5: Operations

**5.1 Monitoring**
- [ ] Zone B trigger rate is being measured and alerted
- [ ] Error rate (any 5xx or agent failure) is being measured and alerted
- [ ] Latency (p50, p95) is being measured
- [ ] At least one person is responsible for checking monitoring daily during first two weeks

**5.2 Rollback**
- [ ] Rollback can be executed in < 15 minutes without engineering involvement (if ops team is responsible)
- [ ] Rollback has been tested in staging (not just designed)
- [ ] Rollback trigger criteria are written down and agreed — not left to judgment in the moment

**5.3 On-call**
- [ ] Someone is reachable during first 48 hours after launch if something goes wrong
- [ ] That person has access to logs and can execute rollback

---

### Section 6: User Readiness

**6.1 User testing**
- [ ] At least 3 target users (not developers) have interacted with the agent in a realistic scenario
- [ ] Their feedback has been reviewed; any blockers addressed
- [ ] Zone B flow specifically tested with real users — they understood what was being asked of them

**6.2 Training and SOP**
- [ ] Users know how to use the Zone B confirmation interface
- [ ] Users know the escalation path for Zone C events
- [ ] A simple FAQ or SOP document exists (doesn't need to be long — a one-pager is fine)

---

## The Go / No-Go Decision

**🟢 Go:** All Section 1-3 items pass. Section 4-6 items pass or have accepted conditional timelines.

**🟡 Conditional Go:** One or more Section 4-6 items are incomplete, but: (a) they are not safety-critical, (b) there is a committed owner and deadline within 2 weeks of launch, (c) a monitoring signal exists that would catch any related problems.

**🔴 No-Go:** Any of the following:
- Section 1.2 (golden dataset) not passing
- Section 2.1 (Zone A reversibility) not addressed for any Zone A action
- Section 2.3 (Zone C escalation) path untested
- Section 3.1 (graceful degradation) not implemented
- Section 5.2 (rollback) not tested
- Any blocker without an owner and a deadline

## The Most Common Launch Mistakes

**Launching without shadow mode data**
The team believes in the model. Real users find edge cases the team never imagined. Shadow mode is not optional — it is the minimum evidence base for a launch decision.

**Treating the vibe check as sufficient**
The vibe check catches obvious regressions. It does not catch long-tail failures at scale. The golden dataset exists for this reason. A 20-case vibe check on 200 golden dataset cases catches very different failure modes.

**No rollback test**
"We can roll back" means nothing unless you've done it. Run a rollback drill in staging. Time it. Make sure the person who would execute it in production knows the steps. Untested rollback plans fail at exactly the moment you need them.

**Skipping user testing because "it's obvious"**
Zone B confirmation UX that is obvious to the development team is frequently confusing to users who don't share the mental model. Minimum: 3 real users, realistic scenario, observe without guiding them.

## Related Skills

- `agent-eval-framework` — The eval infrastructure that feeds this checklist
- `trust-and-autonomy-tradeoff` — Zone A scope at launch is the primary autonomy decision
- `agent-boundary-design` — The Spec this review validates against
- `iteration-flywheel` — What happens after a successful launch
