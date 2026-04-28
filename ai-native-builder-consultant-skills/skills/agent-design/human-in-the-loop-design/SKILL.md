---
name: human-in-the-loop-design
description: Design the human confirmation experience: which actions need approval, which pattern to use, and how to build trust over time. Use when designing Zone B interactions, building approval flows, or fixing low engagement with agent-driven actions.
---

# Human-in-the-Loop Design


## When to Use

- Designing the Zone B confirmation UX for an agent (defined in agent-boundary-design)
- Users are rubber-stamping confirmations without reading — or abandoning them entirely
- Building a Feishu / DingTalk / WeCom card-based approval flow
- Approval rate looks healthy but trust metrics (return rate, edit distance) are trending down

## Ask These Questions First

1. For each action requiring confirmation: is the human verifying correctness (Pattern 1), expected to modify (Pattern 2), or evaluating agent judgment (Pattern 3)?
2. How reversible is each action? How consequential if wrong?
3. What is the current trust level with users — week 1, month 1, or month 3+? (This determines how much to confirm vs. automate.)
4. For chat-embedded products: what is the IM platform, and what card/widget capabilities does it support?

## Output Format
```markdown
## HITL Design Spec

**产品：** [填入]  **版本：** v1.0

**信任阶段：** Week 1 / Month 1 / Month 3+（当前阶段影响确认模式的选择）

### 确认动作清单

| 动作 | 可逆性 | 出错代价 | 确认模式 | 展示内容 |
|------|--------|---------|---------|---------|
| [动作 1] | 高/中/低 | 高/中/低 | Preview+Confirm | [展示什么] |
| [动作 2] | 高/中/低 | 高/中/低 | Edit+Confirm | [展示什么] |
| [动作 3] | 高/中/低 | 高/中/低 | Explain+Confirm | [展示什么] |

### Zone B 通知设计

**通知渠道：** [HMI / IM 推送 / 邮件]
**等待策略：** [X 分钟无响应 → 升级 / 超时自动处理]
**操作选项：** [确认 / 取消 / 修改 / 重测]

### 信任校准计划

| 阶段 | 目标信号 | 升级条件 | 升级到 |
|------|---------|---------|-------|
| 上线初期 | 审批率 > 90% 连续 2 周 | 满足 | Zone A |
| 成熟期 | 编辑率 < 5% 连续 1 月 | 满足 | 自动执行 |

### 核心指标

| 指标 | 目标值 | 预警阈值 | 含义 |
|------|--------|---------|------|
| 审批率（按动作类型） | > 90% | < 80% | 低于预警说明推荐质量差 |
| 编辑距离 | < 20% | > 40% | 高于预警说明草稿偏差大 |
| 覆盖放行率 | < 1% | > 3% | 用户在绕过系统 |
| 决策耗时 | < 30s | > 2min | 信息展示太复杂 |
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 2「Agent 设计」里的 Zone B 体验设计，建立在 `agent-boundary-design` 之后。

**常见联动：**
- `agent-boundary-design` — 先明确哪些动作属于 Zone B
- `trust-and-autonomy-tradeoff` — 决定确认 UX 的严格程度和升级条件
- `agent-eval-framework` — 衡量 HITL 是否真的建立了信任

---

## 核心前提

Human-in-the-loop (HITL) is not a safety feature you add at the end. It is a **trust-building mechanism** you design from the beginning.

Most builders treat HITL as a speed bump — a necessary friction to satisfy compliance or prevent catastrophic errors. This is the wrong frame. HITL, designed well, is how users develop the confidence to eventually trust more automation. HITL designed badly is why users stop using the product.

The difference between the two comes down to three things:
1. **Which actions** require human confirmation (the boundary decision — see `agent-boundary-design`)
2. **What pattern** the confirmation takes (the design decision — this skill)
3. **What information** the human sees when confirming (the content decision — also this skill)

---

## The Three Confirmation Patterns

Every human-in-the-loop interaction uses one of these three patterns. The choice is not aesthetic — it depends on what the human actually needs to do.

### Pattern 1: Preview + Confirm

**Use when:** The action is clear, the human needs to verify it's correct, and editing is not necessary.

```
┌─────────────────────────────────────────┐
│  Agent is about to:                     │
│  Send follow-up message to [Client A]   │
│  Channel: [IM]  |  Time: Now            │
│                                         │
│  [Preview message ▼]                    │
│                                         │
│  [ Cancel ]        [ Send → ]           │
└─────────────────────────────────────────┘
```

**Key design principles:**
- The action must be fully specified — no ambiguity about what "Confirm" will do
- Show a real preview of the output, not a description of it
- The primary action button states the action ("Send →"), not just "Confirm"
- One-click approval — the human shouldn't need to think, just verify

**When this pattern fails:**
- The preview is too long to scan in 3 seconds (shorten it or use Pattern 2)
- The action has side effects not shown in the preview (name them explicitly)
- The human frequently wants to change something (use Pattern 2 instead)

---

### Pattern 2: Edit + Confirm

**Use when:** The agent produces a draft, but the human is expected (or likely) to modify it before approving.

```
┌─────────────────────────────────────────┐
│  Draft client message:                  │
│  ┌───────────────────────────────────┐  │
│  │ [Client A] 您好，                 │  │
│  │ 您的事项将于下月到期，建议您提前   │  │
│  │ 与我们确认下一步方案...           │  │
│  └───────────────────────────────────┘  │
│                                         │
│  [Regenerate ↺]  [Edit ✎]  [Send →]    │
└─────────────────────────────────────────┘
```

**Key design principles:**
- Make editing genuinely easy — inline editing is better than a separate editing interface
- "Regenerate" is as important as "Send" — give the human a path to a different draft without friction
- The edit affordance should be prominent, not hidden
- Track edit rate: if >60% of users edit, the agent's drafting quality needs work; if <5% edit, you might not need this pattern

**When this pattern fails:**
- The draft is so long that editing feels like starting over (agent output should be calibrated to what a human can review in under 30 seconds)
- There's no "Regenerate" option (the human has no recourse if the draft is wrong in a way they can't easily fix)

---

### Pattern 3: Explain + Confirm

**Use when:** The action has significant consequences, the human needs to understand *why* the agent is recommending it, and trust in the agent's judgment is still being established.

```
┌─────────────────────────────────────────┐
│  Recommended action:                    │
│  Flag this claim for manual review      │
│                                         │
│  Why: Three risk indicators present:   │
│  • Claim submitted 6 days after event  │
│  • Third claim this policy year         │
│  • Claimant address changed last week   │
│                                         │
│  Confidence: Medium (not high)          │
│                                         │
│  [ Override: Approve ] [ Flag → ]       │
└─────────────────────────────────────────┘
```

**Key design principles:**
- The explanation must be specific and verifiable — not "this looks risky" but "here are the specific signals"
- Express confidence honestly — "medium confidence" is more trustworthy than "recommended"
- The override option must be prominent and easy — if users feel trapped, they'll distrust the system
- Explain the "why" in the user's vocabulary, not the model's

**When this pattern fails:**
- The explanation is generic ("based on multiple factors...") — then it's not an explanation, it's noise
- Overrides are buried or require extra steps — this signals distrust of the human
- Every action uses this pattern — it creates explanation fatigue and users stop reading

---

## The Confirmation Content Framework

Regardless of pattern, every confirmation interaction should answer four questions for the human:

**1. What exactly will happen?**
State the action in specific, concrete terms. Not "send a message" — "send this specific message to [Client A] via [channel] at [time]."

**2. Is this reversible?**
If yes, say so (reduces anxiety, increases approval rate). If no, make it obvious (increases care, reduces errors).

**3. What should I be checking?**
Guide the human's attention. For a drafted message, the human should check the recipient and the tone. For a system action, they should check the scope. Don't make them figure this out themselves.

**4. What are my options?**
Always offer at least: [Approve] and [Cancel]. Often add: [Modify] or [Regenerate]. For consequential actions: [Override with reason].

---

## Designing for Chat-Embedded Confirmation

Most AI Native products in enterprise contexts live inside existing IM platforms — Feishu, DingTalk, WeCom, Slack. This changes the UX constraints significantly.

### The 3-Second Rule

In a chat stream, users will spend an average of 3 seconds deciding whether to engage with an interactive card. If your confirmation UI requires more cognitive load than that, users will ignore it, dismiss it, or develop a rubber-stamp habit (approving without reading).

Design for 3-second comprehension:
- Action in the card title (verb first: "Send renewal reminder to…")
- Key details visible without expanding
- Primary action button above the fold
- No more than 3 pieces of information before the action buttons

### Feishu Schema V2 Card Patterns

For Feishu-based products, the interactive card system (Schema V2) is the confirmation UI. Key patterns:

**Effective confirmation card structure:**
```
Header: [Action verb] + [Object] + [Channel/method]
Body:   [Preview or key details — max 3 lines visible]
        [Expandable: full preview or full explanation]
Footer: [Secondary action]    [Primary action →]
```

**HMAC signing:** Every action button in a confirmation card should be signed. This prevents replay attacks (user clicks "Send" twice, or the callback is triggered maliciously) and provides an audit trail. The signature should include: user ID, action ID, timestamp, and a hash of the action payload. Expire after 5 minutes.

**Callback design:** The callback handler must be idempotent. If the user double-clicks or the card is rendered twice, the action should execute exactly once. Design this from the start — retrofitting idempotency is painful.

**Timeout behavior:** If a confirmation card times out (user doesn't respond in X minutes), define the explicit behavior: does the action queue, expire, or escalate? Never leave this undefined — undefined timeout behavior is a production incident waiting to happen.

> **Founder pattern from a production payment flow:** A payment confirmation card had a subtle HITL problem: after the user paid, the "success" state of the card remained actionable, so users could tap the payment button again. We added server-side state tracking and made the callback check state before acting. Then we updated the card to a static "已支付" state on success. The fix was small. The bug had been causing duplicate payment attempts for weeks before we caught it. **Idempotency and state feedback are not optional.**

---

## The Trust Calibration Curve

HITL is not a static design — it should change as the user's trust in the agent grows.

```
High │                              ╭──── Full autopilot
     │                         ╭───╯     (agent acts, user notified)
     │                    ╭────╯
Trust│               ╭────╯         ← trust is earned, not assumed
     │          ╭────╯
     │     ╭────╯
Low  │─────╯ Preview+Confirm all actions
     └──────────────────────────────────
          Week 1    Month 1    Month 3+
```

**Week 1:** Confirm everything consequential. The goal is not efficiency — it's calibration. Users are learning what the agent does; the agent (in aggregate) is revealing whether it can be trusted.

**Month 1:** Selectively promote to Edit+Confirm for actions where the approval rate is >90% and the edit rate is <10%. These are actions the agent has demonstrated it handles well.

**Month 3+:** For actions with >95% approval, >6 months of history, and reversible consequences — consider moving to autopilot with notification. The human should be able to review and reverse, but doesn't need to confirm in advance.

**How to measure trust calibration:**
- Approval rate by action type (low approval = wrong action or wrong framing)
- Edit rate by action type (high edit = agent quality issue)
- Override rate by action type (high override = agent judgment problem)
- Time-to-decision by action type (high latency = explanation is too complex or action is too scary)

> **Founder pattern from Shrimper:** We tracked approval rates by action type from day one. The most useful finding: "Schedule meeting" had 94% approval but 60% of the time the user changed the time. The agent was getting the participants and agenda right, but consistently suggesting morning slots for a user who prefers afternoons. One behavioral rule fix (infer preferred time from past confirmed meetings) brought the edit rate to <15%. The approval rate told us the agent was trusted; the edit rate told us where the quality gap was. Both metrics matter.

---

## Anti-Patterns

**The Rubber Stamp**
Confirmations happen so frequently, for actions so low-stakes, that users learn to approve without reading. Then one genuinely important confirmation gets the same treatment.

Fix: Confirmation should be rare enough that it gets attention. Move low-stakes actions to full autopilot (with notification). Reserve confirmation for actions that genuinely warrant human judgment.

**The Explanation Wall**
The agent provides a thorough explanation of its reasoning before every action. Users stop reading after week two.

Fix: Use Explain+Confirm only for high-stakes or novel actions. For routine actions, trust that the user has developed familiarity and use Preview+Confirm instead.

**The One-Button Trap**
The confirmation UI offers only [Confirm] and [Cancel]. The user who wants a slightly different action has no path forward.

Fix: Every Pattern 1 card should have [Cancel]. Every Pattern 2 card should have [Regenerate] and [Edit]. Every Pattern 3 card should have [Override]. Giving the human a path forward other than "accept or reject" is always right.

**The Silent Override**
Users override the agent's recommendation, but there's no way for the system to learn from this. The agent keeps making the same recommendation indefinitely.

Fix: Override events should feed back into the system — at minimum logged, ideally used to update agent behavior. An agent that makes the same wrong recommendation 50 times has a data problem and a product problem. The data problem is usually easier to fix.

**The Phantom HITL**
The product has confirmation UX, but the agent has already executed the consequential action before showing the confirmation. (The message was drafted *and* queued; the "Send" button just triggers delivery.) Users think they're in control; they're not.

Fix: The agent state must be serializable and reversible up to the confirmation point. "Pending confirmation" is a real state in your data model, not a UX fiction.

---

## Related Skills

- `agent-boundary-design` — Decide which actions need HITL before designing the UX
- `system-prompt-engineering` — Encode HITL trigger conditions in the agent's behavioral rules
- `ai-native-ux-patterns` — HITL is one pattern in the broader AI Native UX vocabulary
- `agent-eval-framework` — Measure whether your HITL design is building trust or creating friction
