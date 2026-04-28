---
name: agent-loop-model
description: Explain the perceive-reason-act-observe loop that every agent runs. Use when designing agent architecture for the first time, debugging unexpected agent behavior, or explaining 'what is an agent' to a stakeholder.
---

# Agent Loop Model


## When to Use

- Designing agent architecture for the first time and unclear what "agent" means precisely
- Debugging why an agent loops, stalls, or produces inconsistent results
- Explaining to a stakeholder or engineer what an agent actually does vs. a simple LLM call
- Deciding whether a product needs a reactive loop, a proactive loop, or both

## Ask These Questions First

1. Is the agent triggered by user input (reactive) or by events/schedules (proactive), or both?
2. What tools does the agent have access to in the ACT phase, and which require human confirmation?
3. What state needs to persist across sessions, and what can be ephemeral?
4. What is the stopping condition — how does the agent know it's done?

## Output Format

A **Loop Architecture Summary** covering:
- Trigger model: reactive / proactive / hybrid
- PERCEIVE inputs: what's in context at each iteration (system prompt, history, retrieved docs, injected state)
- ACT inventory: retrieval actions, state-writing actions, external-trigger actions, response actions — and their trust zones
- Memory architecture: working / session / persistent — what goes where
- Stopping conditions: done / blocked / error / safety-stop

---

---

## The Core Loop

Every agent, regardless of framework or complexity, executes some version of this loop:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐     │
│   │ PERCEIVE │ → │  REASON  │ → │   ACT    │     │
│   └──────────┘    └──────────┘    └──────────┘     │
│        ↑                                  │        │
│        └──────────────────────────────────┘        │
│                   OBSERVE                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**PERCEIVE:** The agent takes in its current context — a user message, a tool result, an event trigger, or a combination.

**REASON:** The agent (the LLM) processes this context and decides what to do next — respond, call a tool, ask a clarifying question, or terminate.

**ACT:** The agent executes the decision — sends a response, calls a tool, writes to memory, or triggers an external action.

**OBSERVE:** The agent takes in the result of its action — the tool's return value, the user's reply, an API response — and this becomes the new input to the next PERCEIVE phase.

This loop runs until the agent decides it's done (or a stopping condition is met).

---

## How This Differs From a Stateless LLM Call

A single LLM call is a one-shot transform:

```
Input → [LLM] → Output
```

The LLM has no memory of what happened before this call. It cannot take actions. It cannot observe results. It produces text and stops.

An agent runs the loop multiple times:

```
Input → [LLM] → Action → Result → [LLM] → Action → Result → [LLM] → Response
```

The critical differences:

| | Stateless LLM call | Agent loop |
|---|---|---|
| **Memory** | Only what's in the current prompt | Accumulates across iterations |
| **Actions** | None — produces text only | Can call tools, write to state, trigger external systems |
| **Iteration** | One pass | Multiple passes until done |
| **Termination** | Always after one call | Agent decides when it's done |
| **Error handling** | Caller handles errors | Agent can observe errors and try alternatives |

---

## The Four Phases in Detail

### PERCEIVE: What the Agent Sees

At each iteration, the agent's context window contains some combination of:

- **System prompt (SOUL):** The agent's identity, capabilities, constraints, and behavioral rules — the stable part that doesn't change iteration to iteration
- **Conversation history:** The accumulated messages from the current session
- **Tool results:** The outputs of any tools called in the previous ACT phase
- **Injected context:** Retrieved documents, user profile data, session state — added dynamically based on the current situation
- **Current input:** The user's latest message, or the event that triggered this iteration

**The design implication:** The agent can only reason about what's in its context window. If important information isn't there — whether because it was never retrieved, the context is too long, or it was truncated — the agent will either hallucinate or make a wrong decision. Context management is not an implementation detail; it is a core product design decision.

### REASON: What the LLM Decides

This is the one part of the loop that is genuinely non-deterministic and hard to fully control. The LLM reads everything in the context window and decides:

1. **Is this task complete?** If yes, produce a final response and stop.
2. **Do I need more information?** If yes, either ask the user or call a retrieval tool.
3. **Do I need to take an action?** If yes, which tool to call, with which parameters.
4. **Should I reason step by step before deciding?** (Chain-of-thought, often triggered by instruction or by complexity)

**The design implication:** The quality of the REASON phase depends almost entirely on the quality of the PERCEIVE phase. A model that's reasoning over incomplete, ambiguous, or contradictory context will produce unreliable decisions — no matter how good the underlying model is. Fix perception problems before blaming the model.

### ACT: What the Agent Does

Actions fall into four categories:

**Retrieval actions:** Fetch information from external sources — search, database lookup, API call, file read. The agent doesn't change anything; it just collects information to reason with.

**State-writing actions:** Update persistent state — write to memory, update a database record, log an audit event. These are often irreversible and should be in Zone B (confirmation required) or Zone A (automatic but audited).

**External-trigger actions:** Cause something to happen in the world — send a message, schedule a meeting, make a payment, submit a form. These are the highest-stakes actions because they cross a product boundary and may be irreversible.

**Response actions:** Produce text for the user — a reply, a draft, an explanation, a recommendation.

**The design implication:** The action type determines the trust zone (see `agent-boundary-design`). Retrieval actions can almost always be Zone A (automatic). External-trigger actions usually need to be Zone B (confirmation) until trust is established. State-writing actions depend on reversibility.

### OBSERVE: What the Agent Learns From the Action

After every action, the agent receives a result:

- Tool calls return a response (success, failure, data)
- External actions return a status (sent, failed, pending)
- User messages return the user's next input

This result is appended to the context window, and the loop continues.

**The design implication:** Agents that don't handle tool failures gracefully will either loop indefinitely or produce wrong output with confidence. Every tool call should have an explicit failure path. "The tool returned an error" is valid context; the agent should be able to reason about it and either retry, use an alternative, or surface the error to the user.

---

## The Memory Problem

The agent loop has a fundamental architectural tension: the model has a finite context window, but loops can run many iterations and accumulate a lot of history.

This creates three memory problems:

**Working memory (in-context):** Everything in the current context window. Fast to access, limited in size, lost when the session ends.

**Short-term memory (session state):** Information that needs to persist across a few turns but not across sessions. Typically stored in a session store (Redis, in-memory cache) and injected into context as needed.

**Long-term memory (persistent state):** Information that needs to persist across sessions — user preferences, past decisions, accumulated domain knowledge about this user's situation. Stored in a database, retrieved selectively, injected in summarized form.

**The practical implication for product design:** Decide what each memory layer holds before you build. The most common mistake is putting everything in working memory (bloated context, high cost, eventual truncation) or putting nothing in persistent state (every session starts from zero, no personalization, no continuity).

---

## Loop Termination: When Does the Agent Stop?

Agents terminate on one of four conditions:

1. **Task complete:** The agent determines it has accomplished the goal and produces a final response
2. **Blocked:** The agent needs input it can't get autonomously (user clarification, human approval, missing tool)
3. **Error:** An unrecoverable error occurred and the agent surfaces it rather than continuing
4. **Safety stop:** A constraint or boundary condition was triggered

**The most common production failure:** The agent reaches condition 2 (blocked) but doesn't recognize it — so it loops, trying variations of the same blocked action, consuming tokens and time, and eventually either timing out or producing a wrong answer from frustration.

Design explicit blocked-state recognition: "If I've tried this action twice and failed, I should surface this to the user rather than trying a third time."

---

## The Trigger Model: Reactive vs. Proactive Agents

Most introductory agent descriptions assume a **reactive trigger** — the user says something, the loop runs, the agent responds.

AI Native products often need **proactive triggers** — the agent is triggered by an event, a condition, or a schedule, not by user input.

```
Reactive:  User message → Agent loop → Response to user
Proactive: Scheduled event / condition met → Agent loop → Action (possibly including message to user)
```

Proactive agents require:
- An event bus or scheduler that can trigger the agent loop
- A persistent monitoring state (what is the agent tracking, and when should it fire?)
- Explicit design of the "what to do when triggered" logic — this lives in the SOUL, not just in user-facing prompts

This is the architectural distinction between an agent that waits to be asked and one that proactively surfaces insights. If you're building the second kind, you need both a reactive loop (for user interactions) and a proactive loop (for monitoring). These are separate architectural components that share the same underlying agent.

---

## Common Misconceptions

**"The agent remembers everything from past sessions."**
It doesn't unless you explicitly build persistent memory and inject it. A new session starts with only what's in the system prompt plus whatever you retrieve and inject. Design memory architecture explicitly.

**"More agent iterations = better output."**
Not necessarily. Each iteration adds latency and cost. An agent that takes 8 tool calls to answer a question a good system prompt could have answered in 1 is not more capable — it's poorly designed. Minimize iterations for routine tasks; reserve deep iteration for genuinely complex ones.

**"I can fix bad reasoning by adding more instructions to the system prompt."**
Sometimes. But often bad reasoning is a perception problem (wrong context) or a task problem (the task is genuinely too ambiguous for the current tool set). More instructions can increase confusion when they interact with existing instructions in unexpected ways.

**"The agent is autonomous — it decides everything."**
The agent decides within the constraints you've designed. The system prompt, the available tools, the trust zones, and the stopping conditions are all design decisions. "Autonomous" means the agent handles variation within the designed space, not that it escapes the designed space.

---

## Related Skills

- `ai-native-vs-ai-enhanced` — Before building an agent loop, confirm you actually need one
- `agent-boundary-design` — Design what the agent can do in the ACT phase and when it needs human confirmation
- `system-prompt-engineering` — Design the SOUL that shapes REASON across all loop iterations
- `llm-capability-boundaries` — Understand what the REASON phase can and can't reliably do
