---
name: ai-native-ux-patterns
description: Select and sequence AI Native UX patterns: progressive disclosure, ambient awareness, collaborative drafting, proactive initiation, state persistence, guided discovery. Use when designing beyond the chat box.
---

# AI Native UX Patterns


## When to Use

- Designing the user-facing layer of a new AI Native product beyond a basic chat interface
- Users engage once but don't return — the interaction isn't differentiated from a search box
- Choosing which AI Native patterns to build first (not all six are appropriate at launch)
- An agent is capable but users don't discover its proactive or ambient features

## Ask These Questions First

1. What is the established trust level with users — week 1 (output quality trust only) or month 3+ (judgment trust)?
2. Which patterns are feasible given the agent's current boundary and capabilities?
3. What is the primary user workflow — is it reactive (user initiates) or would proactive initiation add genuine value?
4. What is the IM or UI platform — and what interaction affordances does it support (cards, inline edit, push notifications)?

## Output Format

A **UX Pattern Roadmap** covering:
- Pattern selection and rationale: which of the six patterns to build and why
- Sequencing: v1 / v2 / v3 rollout with trust prerequisite for each
- Per-pattern design notes: trigger conditions, failure modes to design for, metrics to track
- Anti-pattern to avoid: specifically why "chat default" may or may not apply to this product

---

---

## The Core Distinction: Surface vs Substance

**AI-enhanced UX:** the interface is the same; AI makes it faster or smarter.
- Autocomplete that's better
- Search results that are ranked smarter
- A form that pre-fills intelligently

**AI Native UX:** the interface only exists because AI exists; without AI, this interaction is impossible.
- A conversation that adapts to what you reveal across 20 turns
- An agent that proactively identifies something you didn't ask about
- A workflow that reconfigures itself based on what the model infers about your intent

The test: if you replaced the AI component with a rule-based system or a better database, would the interaction still make sense? If yes — it's AI-enhanced. If no — it's AI Native.

Both can be valuable. But they require completely different UX design approaches.

---

## The Six AI Native UX Patterns

### Pattern 1: Progressive Disclosure

**What it is:** The interface reveals complexity gradually as the user demonstrates they're ready for it. The AI infers readiness from behavior, not from user settings.

**What makes it AI Native:** A rules-based system can't infer readiness. It can only ask ("are you a beginner or expert?"). The AI observes how you interact and adjusts without asking.

**When to use it:**
- Domain knowledge gap between novice and expert users is large
- Expert users would be frustrated by beginner UX; novice users would be overwhelmed by expert UX
- You don't want to segment users upfront

**How it works:**
```
Session 1, turn 1: User asks a basic question
→ Agent responds in plain language, offers to explain more

Session 1, turn 5: User uses domain terminology correctly  
→ Agent switches to peer-level language, stops explaining basics

Session 3: User asks an advanced question
→ Agent treats them as an expert from the start of the session
```

**The failure mode:** Revealing too much too fast (user feels the system is making assumptions) or too slow (expert user gives up before the interface adapts). The calibration needs testing with real users.

> **Production pattern:** In domains with a wide expertise gap between novice and experienced users, progressive disclosure resolves the UX dilemma without forcing upfront segmentation. Users who are new get fuller context and confirmation prompts; users with 50+ interactions get direct, peer-level responses with fewer caveats. The calibration is inferred from interaction history — not a setting. Key finding: expert users who encounter "beginner mode" before the system has calibrated to them disengage within two sessions. The first few interactions set the tone — calibrate fast.

---

### Pattern 2: Ambient Awareness

**What it is:** The agent notices things the user didn't explicitly ask about — and surfaces them at the right moment.

**What makes it AI Native:** This is the core behavior of a 搭档 (partner) vs 助理 (assistant). An assistant answers questions. A partner notices things.

**When to use it:**
- The user's domain has common errors or overlooked considerations that experts always catch
- The user may not know what they don't know
- False confidence in a decision has real costs

**How it works:**
```
User: "Draft a follow-up message for the [Client X] case"
Agent: [drafts the message]
       [notices: a key deadline in this case is 6 days away,
        but the standard processing time is 8-12 days]
       "One thing — the deadline is in 6 days but standard
        processing runs 8-12 days. You may want to flag
        this today before drafting the follow-up."
```

**The failure mode:** Over-noticing. If the agent surfaces 3–4 "things to be aware of" per interaction, users learn to ignore them all. Ambient awareness is powerful precisely because it's rare. Reserve it for genuinely important signals.

**The rule:** Surface an ambient observation only when: (a) the user is unlikely to have noticed it, and (b) not noticing has a real cost. If both aren't true, stay quiet.

---

### Pattern 3: Collaborative Drafting

**What it is:** The agent produces drafts that expect human editing, and learns from the editing pattern over time.

**What makes it AI Native:** The draft is not the product — the edited draft is. The agent's job is to get the human to the right output faster than they could get there alone, not to produce a final output independently.

**When to use it:**
- The output quality bar is high and subjective (tone, relationship context, judgment calls)
- The user has strong personal style that needs to be preserved
- The output is something the user will put their name on

**How it works:**
```
Round 1: Agent drafts based on context
User: edits tone, adjusts one factual claim, sends

Round 5: Agent has observed: user always softens formal language,
         always adds the client's personal detail in the second paragraph
→ Agent incorporates these patterns into subsequent drafts
```

**The edit-to-learn loop:**
Track what users change. Not to train the model in real time, but to:
- Update the system prompt with user-specific behavioral rules
- Surface patterns: "I've noticed you often add [X] to messages of this type — should I start including it?"

**The failure mode:** Drafts that are too polished to edit. If the agent produces something that feels "complete," users will either send it without reading (rubber stamp) or throw it out entirely (too intimidating to edit). The ideal draft is 80% right with obvious gaps — it invites collaboration rather than passive acceptance or rejection.

---

### Pattern 4: Proactive Initiation

**What it is:** The agent takes action or surfaces information without being asked, triggered by events or conditions it monitors.

**What makes it AI Native:** A rule-based system can send scheduled notifications. An AI can notice that a situation has developed that the user probably cares about and articulate why.

**When to use it:**
- Time-sensitive situations the user may not be monitoring
- Pattern recognition across data the user can't process manually
- "Wouldn't it be useful if someone told you when X" — if yes, that's a proactive initiation opportunity

**How it works:**
```
[Monday 9am, agent monitoring pipeline runs]
Agent detects: 3 high-priority items in the user's pipeline
are approaching deadlines. No action initiated for any.

→ Proactive message:
"Morning. Three items need attention this week —
[Item A] in 8 days, [Item B] in 10 days, [Item C] in 12 days.
[Item A] is most urgent given current processing timelines.
Want me to draft follow-ups for all three?"
```

**The failure mode:** Alert fatigue. If the agent initiates contact 8 times a day, users mute it. Define proactive initiation triggers narrowly: only high-signal, time-sensitive, actionable events. Everything else stays passive — available when the user asks, not pushed when they didn't.

**The trust threshold:** Proactive initiation requires higher established trust than reactive response. Don't build it into v1. Build it in v2, after users have developed confidence in the agent's reactive responses.

---

### Pattern 5: State Persistence and Resumption

**What it is:** The agent maintains context across sessions and can resume complex workflows mid-flight, even across days or weeks.

**What makes it AI Native:** A rule-based workflow system can save state. But only an AI can reconstruct the *reasoning context* — what was being considered, what was deferred, what information was still outstanding — and resume a nuanced conversation naturally.

**When to use it:**
- Workflows span multiple days or require information gathering over time
- Users regularly need to "pick up where they left off"
- The context of a situation changes between sessions (new information, time passed)

**How it works:**
```
[Session 3, two days after Session 2]
User: "Where did we leave off on the [Project X] case?"

Agent: "Last time we were working through [Project X].
You were waiting on a piece of information from [Party]
before confirming the next step. That hasn't come in yet —
it's been 48 hours. The window closes in 6 days.
Do you want me to draft a follow-up?"
```

**State management considerations:**
- What to persist: decision context, outstanding items, explicitly noted preferences
- What not to persist: every message (noise), personally identifiable information beyond what's necessary
- How to surface: brief, scannable summary at session start — not a full replay

---

### Pattern 6: Guided Discovery

**What it is:** The agent leads the user through a structured exploration or decision process, asking questions to surface what the user actually needs (which is often different from what they asked for).

**What makes it AI Native:** This is Socratic method at scale. A rule-based decision tree can ask branching questions. An AI can hold the entire context of the conversation, notice contradictions, adjust the question sequence dynamically, and arrive at a nuanced conclusion.

**When to use it:**
- Users often don't know what they need (or think they know, but are wrong)
- The right answer depends on constraints the user hasn't articulated yet
- The domain has non-obvious interdependencies that matter for the recommendation

**How it works:**
```
User: "I need a recommendation for my client — they have some specific constraints."

Agent: "Happy to help narrow that down. A few quick questions:
Is this for one person or multiple?
[User: two people]
What are their key profile details?
[User: mid-40s, no major health history]
Any specific coverage focus, or broad protection?
[User: broad — they have a family history concern in one area]
And is there a budget range?
[User: moderate, fixed annual amount]

Okay — with those constraints, I'm ruling out [X] because 
[it doesn't fit the profile], 
and [Y] because [budget doesn't work for two]. 
That leaves three options worth looking at..."
```

**The failure mode:** Interrogation mode. If the agent asks 6 questions before giving any value, users disengage. The rule: ask one question at a time, give partial value as soon as you have enough to say something useful, and frame questions as collaborative ("this will help me rule out options") not bureaucratic ("I need this information before I can proceed").

---

## Choosing the Right Pattern

These patterns aren't mutually exclusive. Most mature AI Native products use 3–4. But starting with all six is a mistake — each adds design and engineering complexity, and some require trust that hasn't been built yet.

**Sequencing recommendation:**

| Stage | Patterns to build | Why |
|-------|------------------|-----|
| v1 (launch) | Collaborative Drafting + Guided Discovery | Immediate value, low trust required |
| v2 (1 month) | Progressive Disclosure + State Persistence | Rewards returning users |
| v3 (3 months) | Ambient Awareness + Proactive Initiation | Requires established trust |

**The trust requirement rule:**
- Patterns 1–3 (disclosure, awareness, drafting): trust in *output quality*
- Patterns 4–6 (initiation, persistence, discovery): trust in *agent judgment*

Don't build Pattern 4 (proactive initiation) until users trust Pattern 2 (ambient awareness). The sequencing matters.

---

## The One Anti-Pattern to Avoid Above All

**The Chat Default**

When in doubt, teams build a chat interface. "It's AI, so it should be conversational."

Chat is one modality. It is appropriate for:
- Open-ended exploration (Guided Discovery)
- Complex back-and-forth (Collaborative Drafting)
- Situations where the right action is unclear

Chat is inappropriate for:
- Routine actions with known structure (use a UI with AI assist)
- High-frequency, low-complexity interactions (latency kills adoption)
- Situations where the user knows exactly what they want (chat forces them to describe it)

The question is never "chat or not chat." The question is: what does the user need to accomplish, and what interface gets them there fastest while building trust?

Sometimes that's chat. Often it isn't.

---

## Related Skills

- `human-in-the-loop-design` — The confirmation mechanics inside several of these patterns
- `agent-boundary-design` — Defines what the agent can do, which constrains which patterns are possible
- `system-prompt-engineering` — Each pattern requires specific behavioral rules in the SOUL
- `agent-eval-framework` — How to measure whether your chosen patterns are working
