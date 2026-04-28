---
name: multi-agent-orchestration
description: Design router/worker multi-agent architecture with typed handoff contracts and failure isolation. Use when splitting a single agent, designing agent-to-agent communication, or debugging cascading failures in a multi-agent system.
---

# Multi-Agent Orchestration


## When to Use

- Designing a system with more than one agent and need to define how they communicate
- A single agent is hitting complexity limits — system prompt bloat, mixed trust zones, unclear failure modes
- Multi-agent system is producing inconsistent or cascading failures in production
- Need to define the handoff contract between a router and worker agents before building either

## Ask These Questions First

1. Run the split test: does each proposed agent have independent failure, independent deployment, and independent trust boundary?
2. What is the router's job — classification only, or does it also do work?
3. For each agent-to-agent handoff: what data is passed, in what schema, and what happens on failure?
4. What is the maximum call depth, and is there any risk of cycles in the call graph?

## Output Format
```markdown
## Multi-Agent Architecture Spec

**系统名称：** [填入]  **版本：** v1.0

### 拆分决策

| 测试 | 结论 | 说明 |
|------|------|------|
| 独立失效性 | 是 / 否 | [说明] |
| 独立部署性 | 是 / 否 | [说明] |
| 独立信任边界 | 是 / 否 | [说明] |

**决策：** 拆分 / 不拆分（保持单 Agent + 工具调用）

### Agent 清单

| Agent | 职责（一句话 Job Story） | 信任级别 |
|-------|------------------------|---------|
| [Router] | [分类意图，路由到对应 Worker] | 无需用户信任 |
| [Worker A] | [具体职责] | Zone A/B/C |
| [Worker B] | [具体职责] | Zone A/B/C |

### Handoff 契约

**Router → Worker A：**
```json
{
  "request_id": "string",
  "intent": "enum([意图列表])",
  "context_summary": "string (≤200字)",
  "constraints": { "[字段]": "[类型]" }
}
```

**Worker A → 结果：**
```json
{
  "request_id": "string",
  "status": "success | partial | not_found | error",
  "result": "[结果内容]",
  "confidence": "high | medium | low",
  "error_message": "string | null"
}
```

### 调用图（防止循环）

```
Router → Worker A（单向）
Router → Worker B（单向）
最大调用深度：[填入]
```

### 失败隔离

| Agent | 失败行为 | 降级策略 |
|-------|---------|---------|
| [Worker A] | [失败条件] | [降级到什么] |
| [Worker B] | [失败条件] | [降级到什么] |

### 可观测性

- 追踪粒度：每次 Handoff 记录 request_id + 输入摘要 + 输出摘要 + 延迟
- 告警：任意 Agent 错误率 > [填入]% 触发告警
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 4「技术决策与实施计划」中的条件技能，只在单 Agent 已经不能清晰承载职责时启用。

**常见联动：**
- `tech-stack-selection` — 先确认系统复杂度是否真的需要多 Agent
- `agent-eval-framework` — 多 Agent 系统需要更严格的验证
- `launch-risk-review` — 上线前需要重点审查 handoff 和失败隔离

---

## 核心前提

Multi-agent systems are the right answer to exactly one problem: a task that genuinely requires independent, composable units of capability.

They are the wrong answer to: complexity that feels cleaner when split up, a desire for parallelism, wanting to use different models for different parts, or general architectural ambition.

This skill helps you build multi-agent systems that work — and helps you avoid building them when you don't need to.

---

## Before You Start: The Split Test

Run this before designing any multi-agent architecture.

For each proposed agent split, answer:

**1. Independent failure?**
If Agent B fails completely, can Agent A still deliver value to the user? If no — they're coupled enough that splitting adds complexity without resilience.

**2. Independent deployment?**
Can you update Agent B's system prompt or model without touching Agent A? If no — you don't have two agents, you have one agent with a confusing call structure.

**3. Independent trust boundary?**
Do users grant different levels of trust to what each agent does? (See `agent-boundary-design`.) If no — the split is architectural, not product-meaningful.

If you can't answer yes to at least two of three: keep it as one agent with tool-calling. The complexity cost of multi-agent is real. Pay it only when you get something real back.

---

## The Router / Worker Pattern

The most reliable multi-agent architecture for product use cases. Everything else is a variation on this.

```
User message
     │
     ▼
┌─────────────┐
│   Router    │  ← stateless, fast, cheap model
│   Agent     │    classifies intent → selects worker
└──────┬──────┘
       │
  ┌────┴─────┬──────────┬──────────┐
  ▼          ▼          ▼          ▼
Worker A   Worker B   Worker C   Fallback
(lookup)  (drafting) (analysis)  (human)
```

**The router's job is narrow:** classify the incoming request and route it to the right worker. That's it. The router should not do any work itself, should not have domain knowledge, and should not make judgment calls.

**Worker agents are specialists:** each has deep context for its domain and no knowledge of the others. Workers should never call other workers directly — all inter-agent communication goes through the router or a shared state store.

**The fallback is not optional:** every router must have an explicit path for "none of the above." What happens when the request doesn't match any worker? If you don't design this, the system will route ambiguous requests to the closest-matching worker, which will try to answer something it shouldn't, and produce confident wrong output.

---

## Handoff Contracts

The most common source of multi-agent failures in production is an implicit handoff contract — Agent A passes data to Agent B in a format Agent B didn't expect, and Agent B either silently fails or hallucinates to fill the gaps.

**A handoff contract is a typed data schema.** Write it before you build either agent.

```typescript
// Example: Router → Lookup Worker handoff contract
interface LookupHandoff {
  // What the router passes
  request_id: string;           // for tracing
  user_id: string;              // for personalization / audit
  intent: "product_lookup"      // confirmed by router
        | "policy_lookup"
        | "coverage_check";
  query: string;                // normalized query text
  constraints: {                // extracted structured constraints
    product_type?: string;
    age_range?: [number, number];
    budget_max?: number;
  };
  context_summary: string;      // ≤200 word conversation summary
                                // agent doesn't get full history
}

// What the Lookup Worker returns
interface LookupResult {
  request_id: string;           // echo back for tracing
  status: "success" | "partial" | "not_found" | "error";
  results: ProductRecord[];
  confidence: "high" | "medium" | "low";
  missing_constraints: string[]; // what it needed but didn't get
  error_message?: string;        // if status === "error"
}
```

**Why this level of formality?**

Because in six months, when something breaks at 2am, the contract is the thing that tells you which agent is at fault and what data was in flight. Without it, you're reading LLM logs trying to reverse-engineer what was passed and why.

> **Founder pattern from Shrimper:** The agent-registry routing layer was built with an informal handoff — the router passed raw message payloads, workers assumed structured context objects. Everything worked in testing because the test messages happened to match the structure workers expected. In production, messages from Feishu had a different schema than messages from Telegram (different metadata fields, different user ID formats). Workers failed silently — returning empty results rather than errors — because they could parse *something* from the payload, just not the right fields.
>
> The fix was a normalization layer between router and workers: every incoming message, regardless of channel, is transformed into a canonical `AgentContext` object before routing. The workers now have a contract they can rely on. **Write the contract before writing either agent. The normalization layer is cheaper to build up front than to retrofit.**

---

## The Call Graph Rules

Three rules that prevent the most common multi-agent failure modes:

**Rule 1: No cycles.**
Agent A must never call Agent B if Agent B can call Agent A (directly or transitively). Draw the call graph. If there's a cycle, one of those agents needs to be redesigned.

How to check: number your agents 1..N. Agent 1 can only call agents 2..N. Agent 2 can only call agents 3..N. Etc. If any agent calls one with a lower number, you have a cycle.

**Rule 2: Bounded depth.**
Set a maximum call depth (3 is usually right; 5 is the hard limit). A chain of Agent A → Agent B → Agent C → Agent D has three handoff points, each of which can fail and each of which adds latency. If you find yourself needing depth > 3, the task scope is too broad for a single user interaction — split it into multiple turns.

**Rule 3: No shared mutable state between workers.**
Workers should read from shared state (user profile, conversation history, retrieved documents) but never write to state that another worker also writes. If two workers need to update the same record, that update goes through the router or a dedicated state manager — never directly.

---

## Failure Isolation

In a multi-agent system, failures must not cascade. Define failure behavior for each agent before building.

**The three failure modes and their responses:**

**Graceful degradation:** Agent B fails, Agent A continues with reduced capability.
```
Lookup Agent fails → Drafting Agent uses only the user's 
message context, produces a draft that acknowledges 
"I don't have the specific product details right now"
```

**Explicit error propagation:** Agent B fails with a typed error, Agent A handles it.
```typescript
if (lookupResult.status === "error") {
  // Don't try to draft with incomplete data
  // Route to fallback handler
  return router.escalate(request_id, lookupResult.error_message);
}
```

**Timeout with partial result:** Agent B takes too long, Agent A proceeds with what it has.
```
If Lookup Agent doesn't respond within 3s:
→ proceed with partial context
→ flag response as "based on incomplete data"
→ log timeout for monitoring
```

**Never:** silently swallow errors and proceed as if they didn't happen. This is how multi-agent systems produce confident wrong output.

---

## Context Passing: The Minimal Principle

Every handoff is a decision about how much context to pass. The instinct is to pass everything — the full conversation history, all retrieved documents, all user metadata. This is almost always wrong.

**The minimal context principle:** pass only what the receiving agent needs to do its specific job. Nothing more.

Why it matters:
- Long contexts increase latency and cost
- Long contexts dilute the relevant signal (the model attends to everything; important things get lost)
- Long contexts make debugging harder (what was the agent actually using?)

**How to define minimal context:**
For each handoff, ask: if the receiving agent had *only* this data and nothing else, could it do its job well? If yes, that's the right context. If no, add the minimum missing piece and ask again.

**The context summary pattern:**
Instead of passing full conversation history, pass a structured summary:
```
context_summary: "A domain professional's client: couple, mid-40s, 
no major pre-existing conditions, moderate budget. 
Previous messages: focused on a specific coverage type."
```

The summarizer can be the router (cheap model, runs fast) or a dedicated summarization step. Either way: structured summaries beat raw history every time for worker agent performance.

---

## Observability: The Non-Negotiable

You cannot debug a multi-agent system you can't observe. Before going to production, instrument these three things:

**1. Request tracing**
Every request gets a `request_id` at the router. Every agent logs: `{request_id, agent_name, input_hash, output_hash, latency_ms, status}`. You can now reconstruct any request's full path through the system.

**2. Handoff logging**
Log the full handoff payload (input and output) for every agent transition, with the `request_id`. When something goes wrong, you have the exact data that was in flight.

**3. Failure rate by agent**
Track `error_count / total_requests` per agent, per time window. Alert when any agent's error rate exceeds threshold. This is how you find the weakest link.

> **Founder pattern:** When we instrumented Shrimper's worker pool properly, the most surprising finding was that 23% of failures originated not in the workers, but in the normalization layer between the channel adapters and the router — bad Feishu webhook payloads that the normalizer wasn't handling. Without per-step tracing, we would have blamed the workers. Observability tells you *where* to look; logs tell you *what* happened. You need both.

---

## Worked Example: domain professional Multi-Agent

**The task:** A domain professional asks for a recommendation for a specific client profile.

**The wrong design (single complex agent):**
One agent receives the request, does product lookup, applies underwriting logic, generates recommendation, drafts client communication. When it's wrong, you don't know which step failed.

**The right design (router/worker):**

```
Router (intent classification)
  ├─ intent: "recommendation_request"
  └─ routes to → Recommendation Pipeline

Recommendation Pipeline (orchestrator, not a model):
  1. Call Lookup Worker: get candidate products
  2. Call Filter Worker: apply client's constraints
  3. Call Ranking Worker: score by fit
  4. Call Drafting Worker: write recommendation

Each step: typed input/output contract, explicit failure handling
```

**Handoff contract (Router → Lookup Worker):**
```json
{
  "request_id": "req_20250315_001",
  "intent": "recommendation_request",
  "client_profile": {
    "age": 45, "smoker": false,
    "pre_existing": [], "budget_annual": 8000
  },
  "query_focus": "critical_illness",
  "context_summary": "A domain professional's client context..."
}
```

**What this design enables:**
- Lookup Worker fails → skip to fallback with "limited product data" caveat
- Filter Worker produces zero results → explicit "no products match" response, not hallucination
- Each worker's performance tracked independently
- Prompt changes to Drafting Worker don't require retesting Lookup Worker

---

## Related Skills

- `agent-boundary-design` — The split decision that determines whether multi-agent is right at all
- `agent-eval-framework` — How to test a multi-agent system (harder than single-agent eval)
- `system-prompt-engineering` — Each worker needs its own well-designed SOUL
