---
name: ai-native-vs-ai-enhanced
description: Classify a product as AI Native or AI-enhanced. Use when scoping a new AI product, evaluating whether agent architecture is needed, or diagnosing a 'chatbot wrapper' that isn't delivering value.
---

# AI Native vs AI-Enhanced


## When to Use

- Someone describes a new AI product idea and you need to understand what kind of AI integration is actually needed
- A product feels like "a chatbot bolted on" and users aren't engaging
- The team is debating whether to build agent architecture or a simpler model integration
- Scoping an MVP: deciding how much AI infrastructure is warranted

## Ask These Questions First

1. If you removed the AI component entirely, would the product still exist — just worse? Or would it cease to exist?
2. Can you replicate 80% of the value with a well-designed decision tree or rules engine?
3. Does the interaction require a genuinely new paradigm (ongoing dialogue, proactive initiation, behavioral inference) or does it fit an existing one (search, form, dashboard)?
4. Who is the moat: the AI behavior itself, or the data/network effects around it?

## Output Format
```markdown
## AI 产品定位声明

**产品名称：** [填入]
**核心价值主张：** [一句话——用户真正得到的，不是功能列表]

**AI 角色定位：**
- 分类：AI Native / AI-enhanced / 混合（哪些部分属于哪类）
- 判断依据：[四个测试的结论摘要]

**架构含义：**
- 需要 Agent 架构：是 / 否
- 理由：[一句话]

**护城河来源：**
- AI 行为本身是差异化：是 / 否
- 若否，护城河来自：[数据 / 网络效应 / 品牌]

**下一步：** 进入 `agent-boundary-design`，定义 Agent 的边界
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 1「定位诊断」的起点，用来判断项目到底是在做 AI Native、AI-enhanced，还是混合形态。

**常见联动：**
- `agent-loop-model` — 当客户对“Agent 到底是什么”没有共同心智模型时一起使用
- `agent-boundary-design` — 当确认需要 Agent 架构后，进入边界设计

---

## Why This Distinction Matters

The difference between AI Native and AI-enhanced is not a question of how much AI is in the product. It is a question of whether the product's core value proposition **depends on** AI — or merely benefits from it.

Getting this wrong at the start leads to two common failure modes:

**Over-engineering:** Building a full agent architecture for a product that just needs a smarter autocomplete. You pay in complexity, latency, and cost for something a rule-based system would have handled fine.

**Under-engineering:** Building a thin AI wrapper around an existing product and wondering why users don't engage. The interaction model was designed for a different paradigm. No amount of prompt tuning fixes a paradigm mismatch.

---

## The Definitions

**AI-enhanced:** AI makes an existing interaction faster, smarter, or easier. The core interaction exists without AI — AI is an accelerant.

Examples:
- Search that ranks results better because of embeddings
- A form that pre-fills intelligently based on past inputs
- A dashboard that surfaces anomalies you'd have found eventually anyway
- Autocomplete that finishes your sentence better than it used to

Remove the AI and the product still works — just slower or less conveniently.

**AI Native:** The core value proposition only exists because AI exists. The interaction cannot be replicated by a rule-based system, a better database, or a more experienced human doing the same task manually.

Examples:
- An agent that monitors a professional's workflow and proactively flags issues they didn't ask about
- A system that conducts a nuanced multi-turn conversation to elicit requirements a user couldn't have articulated in a form
- A product that adapts its entire interaction model to each user based on behavioral inference — not settings
- An orchestration layer that reasons across disparate data sources to synthesize a judgment that no single rule could produce

Remove the AI and the product doesn't just get worse — it ceases to exist.

---

## The Four Tests

Run these four tests on any product idea. The answers tell you what you're building.

### Test 1: The Rule-Based Replacement Test

> "Could you replicate 80% of this product's value with a sufficiently sophisticated decision tree or rules engine?"

- Yes → AI-enhanced. The AI is doing optimization, not transformation.
- No → AI Native candidate. Proceed to Test 2.

### Test 2: The Interaction Paradigm Test

> "Does this product require a fundamentally new interaction paradigm — one that didn't exist before LLMs — or does it fit an existing paradigm (search, form, dashboard, notification)?"

- Existing paradigm → AI-enhanced, however good the AI is.
- New paradigm (ongoing dialogue, proactive initiation, behavioral inference, judgment synthesis) → AI Native candidate.

### Test 3: The Marginal User Value Test

> "For a user who becomes expert at using this product, does the AI get *more* valuable over time — or does it fade into the background?"

- Fades into background → AI-enhanced (becomes infrastructure, which is fine)
- Grows more valuable → AI Native. The model's accumulation of context, pattern recognition, or judgment improves the value delivered.

### Test 4: The Defensibility Test

> "If the AI component were commoditized — same quality available to any competitor for free — would there still be a moat?"

- Yes → The moat is elsewhere (network effects, data, brand). AI is a feature.
- Barely / No → The AI behavior *is* the product. This is AI Native territory — the differentiation comes from how you've designed the AI's judgment, not just the underlying model.

---

## The Spectrum (Most Products Live Here)

Very few products are purely one or the other. Most sit on a spectrum:

```
AI-enhanced                                          AI Native
─────────────────────────────────────────────────────────────
Better      Smarter    AI-first    Agent-    Fully
autocomplete  search     UX       assisted  autonomous
                                  judgment    agent
```

The useful question is not "which end of the spectrum?" but **"where on the spectrum does the core value proposition live?"**

A CRM with AI-powered lead scoring sits left of center — the CRM exists without AI; the scoring is an enhancement.

A system where a professional's entire daily workflow is mediated through an agent that proactively manages their task queue, drafts their communications, and surfaces decision-relevant information without being asked — that sits right of center. The value is the agent behavior, not the underlying data management.

---

## Why This Shapes Everything Downstream

The AI Native / AI-enhanced distinction is not just semantic. It determines:

**Architecture:** AI Native products need agent architecture, SOUL files, trust zone design, eval frameworks. AI-enhanced products need model integration, good prompting, caching. These are different engineering problems.

**UX paradigm:** AI Native UX is built around the six patterns in `ai-native-ux-patterns` — progressive disclosure, ambient awareness, collaborative drafting, proactive initiation. AI-enhanced UX mostly adapts existing UI patterns with smarter backends. Applying AI Native UX patterns to an AI-enhanced product creates confusion (users don't expect the product to have opinions). Applying AI-enhanced UX to an AI Native product produces a chatbot wrapper (users don't get the full value).

**Defensibility strategy:** AI-enhanced products are defensible through data, switching costs, and brand — the AI is table stakes. AI Native products need to build moats in the AI behavior itself: the quality of the agent's judgment, the depth of the system prompt, the quality of the feedback loop from production data.

**Go-to-market:** AI-enhanced products can often sell to existing buyers of the category ("it's your existing tool, but smarter"). AI Native products often need to create a new buying category, which is harder but, when it works, more defensible.

---

## The Most Common Mistake

**Describing an AI Native vision but building AI-enhanced infrastructure.**

The pitch is: "an agent that proactively manages the professional's entire workflow." The build is: a chat interface on top of existing data, with a prompt that says "be proactive."

These are incompatible. The vision requires:
- Event-driven architecture (agent is triggered by conditions, not just user messages)
- State persistence across sessions
- Ambient awareness logic (what should the agent monitor and when should it surface something?)
- Trust zone design (what can it do autonomously vs. what needs confirmation?)

None of that comes from making the chat interface better. The architecture has to match the paradigm.

**The diagnostic question:** "Is the agent sitting and waiting for the user to say something, or does it have a persistent monitoring and reasoning loop?" If it's waiting — you've built an AI-enhanced chat interface. If it has a loop — you're building AI Native.

---

## Output: The Positioning Statement

After running the four tests, produce this:

```markdown
## Product AI Positioning

**Core value proposition:** [one sentence — what does this product do for the user?]

**AI role:** [AI Native / AI-enhanced / hybrid — with which parts in each category]

**The rule-based replacement test:** [what a rules engine could and couldn't replicate]

**Interaction paradigm:** [existing paradigm adapted / genuinely new paradigm]

**AI behavior is the moat:** [yes / no / partially — explanation]

**Architecture implication:** [agent architecture needed / model integration sufficient]
```

This one-pager forces clarity before any design or engineering work begins. The most expensive time to discover you've misclassified the product is six months into development.

---

## Related Skills

- `agent-loop-model` — Once you know you're building AI Native, understand the mechanical model your agent runs on
- `agent-boundary-design` — The first design decision for any AI Native product
- `ai-native-ux-patterns` — The UX patterns that are only appropriate for AI Native products
