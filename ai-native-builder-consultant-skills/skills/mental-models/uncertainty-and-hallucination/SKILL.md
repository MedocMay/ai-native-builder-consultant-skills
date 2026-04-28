---
name: uncertainty-and-hallucination
description: Diagnose why an agent produces confident wrong output and apply the right fix. Use when an agent is hallucinating, when designing a fact-sensitive feature, or auditing a product's risk profile.
---

# Uncertainty and Hallucination


## When to Use

- An agent is producing confident wrong answers and you need to know why and what to fix
- Designing a feature that surfaces facts, recommendations, or analysis to users
- Assessing the risk profile of a proposed AI feature before building
- A RAG system is giving wrong answers and you need to distinguish retrieval failure from model hallucination

## Ask These Questions First

1. What type of wrong answer are you seeing — fabricated facts, wrong reasoning, format errors, or misattribution?
2. Does the agent have access to the correct information in its context, or is it relying on parametric memory?
3. What is the cost of a confident wrong answer in this product — minor annoyance, trust erosion, or serious harm?
4. Is the failure consistent (same input always fails) or intermittent (same input sometimes fails)?

## Output Format

A **Hallucination Diagnosis** covering:
- Root cause: parametric memory gap / instruction-following failure / context confusion
- Severity: frequency estimate + cost per failure
- Recommended fix from the five patterns: retrieval grounding / uncertainty disclosure / source attribution / scope limiting / human review
- For RAG systems: retrieval failure vs. model hallucination verdict with diagnostic evidence

---

## The Core Problem

LLMs don't know when they don't know.

This is not a bug that will be fixed in the next model version. It is a structural property of how these models work: they generate the most statistically plausible continuation of the input, which is often correct and occasionally confident-and-wrong. The model has no reliable internal signal for "I'm guessing" vs "I know this."

The practical consequence: the model will produce fluent, confident, detailed wrong answers on some fraction of queries. That fraction varies by domain (lower for common knowledge, higher for specialized or recent information), by query type (lower for "explain this" questions, higher for "what is the exact value of X" questions), and by context quality (lower when relevant context is provided, higher when the model is relying on parametric memory alone).

Product design that ignores this will produce trust-ending incidents. Product design that works with it can build highly reliable systems.

---

## The Three Root Causes

Understanding why hallucinations happen tells you which design intervention to apply.

### Cause 1: Parametric Memory Gaps

The model's training data didn't contain reliable information about this topic — either because the topic is obscure, recent, specialized, or inconsistently represented.

When asked, the model doesn't say "I don't know." It pattern-matches to the closest thing it does know and generates a plausible-sounding answer.

**Signature:** The wrong answer is coherent and plausible. It sounds like it could be true. It often mixes correct and incorrect information, making it hard to spot.

**Example failure:** "What is the exact regulatory requirement for X in province Y?" — the model knows the general regulatory landscape but not the specific rule. It generates a specific-sounding answer that's based on similar (but different) regulations.

**Fix:** Retrieval. The model should never be asked to recall specific facts from parametric memory for high-stakes queries. Provide the facts; ask the model to reason about them.

### Cause 2: Instruction-Following Under Pressure

The model is instructed to be helpful and complete. When it encounters a gap in its knowledge, the helpfulness imperative can override the honesty imperative — especially if the system prompt doesn't explicitly encode uncertainty disclosure as a priority.

**Signature:** The model produces a confident answer when a more honest response would be "I'm not sure about this specific detail." The failure is a prioritization failure, not a knowledge failure.

**Example failure:** A user asks a specific question. The model has partial knowledge. A well-designed system would say "I have general knowledge about this but not the specific detail — here's what I do know, and here's where to verify." An under-designed system produces the partial knowledge as if it were complete.

**Fix:** System prompt design. Explicitly encode uncertainty disclosure as a first-class behavior: "When uncertain about a specific fact or detail, say so before providing the information, not after. Uncertainty disclosure precedes the uncertain information." See `system-prompt-engineering`, Layer 5 (Behavioral Rules).

### Cause 3: Context Confusion

The model has access to multiple pieces of information — some retrieved, some from the conversation history, some from its training — and they conflict, or the model misattributes which source to use for a given claim.

**Signature:** The answer is partially correct (the parts the model retrieved) and partially wrong (the parts it filled in from training or misattributed). Often harder to detect because the correct parts lend credibility to the wrong parts.

**Example failure:** A RAG system retrieves three product documents. The model correctly answers one part of the query from Document A, but then fills in a detail from its training data (not from the retrieved documents) when the retrieved documents don't contain that specific detail.

**Fix:** Grounding instructions. The system prompt should specify the model's epistemic hierarchy: "Answer using only the information provided in the retrieved context. If the retrieved context does not contain the answer to a specific question, say so explicitly. Do not supplement with information from outside the provided context."

---

## The Five Design Patterns for Uncertainty Management

### Pattern 1: Retrieval Grounding

Don't ask the model to recall facts. Provide facts; ask the model to reason about them.

This is the most powerful intervention. A model reasoning over accurate retrieved context is substantially more reliable than a model reasoning from parametric memory alone.

Implementation: Any query that requires specific factual claims should trigger retrieval before generation. The model's job shifts from "know the answer" to "find the right answer in what I've been given and reason about it."

Limitation: Retrieval grounding only works when the retrieval is accurate. A RAG system with poor retrieval quality substitutes one failure mode (model hallucination) for another (confident wrong answers based on wrong retrieved content). The retrieval pipeline is not a free pass — see `knowledge-injection-strategy` for retrieval quality design.

### Pattern 2: Explicit Uncertainty Disclosure

Instruct the model to surface uncertainty before providing uncertain information, not as a post-hoc disclaimer.

The reason this matters: disclaimers after the fact don't work psychologically. "Here is the answer [detailed confident-sounding answer]. Note that you should verify this." — the user has already formed the belief. The disclaimer doesn't undo it.

Uncertainty disclosure before the uncertain information changes the cognitive frame: "I have general knowledge about this but I'm not certain about this specific detail — here's what I do know: [information]. You should verify [specific claim] with [specific source]."

This requires explicit encoding in the system prompt. Without it, the model's default behavior is to produce confident output (because that's what training data looks like — confident output gets positive feedback).

### Pattern 3: Source Attribution

When possible, have the model cite the specific source for each factual claim — either a retrieved document or an explicit acknowledgment that it's reasoning from general knowledge.

This serves two functions:
- Users can verify claims that matter
- The model itself is forced to be explicit about the basis for each claim, which reduces confident confabulation (it's harder to confabulate confidently when you have to name the source)

Implementation: Instruct the model to cite sources inline: "According to [document name/section], [claim]." For claims not from retrieved context: "Based on general knowledge (not from a specific document), [claim]."

### Pattern 4: Scope Limiting

Design the agent to have a narrow epistemic scope — it knows specific things about a specific domain, and it explicitly does not know other things.

A narrowly scoped agent is less likely to hallucinate than a broadly scoped one, because the narrow scope reduces the surface area where parametric memory gaps can surface. It also makes it easier to encode clear uncertainty disclosure: "For questions outside [specific domain], I don't have reliable information and I'll say so."

This is the "explicit ignorance" principle from `system-prompt-engineering` Layer 3: state what the agent doesn't know as clearly as what it does know.

### Pattern 5: Human Review for High-Stakes Claims

For outputs where confident wrong answers have serious consequences — medical, legal, financial, compliance — design human review into the workflow rather than treating model output as final.

This is not a failure of AI design; it's appropriate calibration of human oversight to stakes. An agent that drafts a compliance analysis for human legal review is more useful than one that produces the same draft as a final answer, because the draft plus review is more reliable than the draft alone.

Design the handoff explicitly: what triggers human review, who reviews it, what they're checking for, and how the reviewed output feeds back.

---

## Distinguishing Hallucination From Retrieval Failure

In RAG-based systems, wrong answers come from two different sources that require different fixes:

**Model hallucination:** The model had the retrieved context but ignored it or supplemented it with wrong parametric memory.

**Retrieval failure:** The retrieved context was wrong, incomplete, or irrelevant, and the model reasoned correctly over bad inputs.

These look the same to the user (wrong answer with confidence) but require completely different fixes.

**How to tell them apart:**
1. Inspect the retrieved context for the query that produced the wrong answer
2. If the retrieved context contains the correct information → model hallucination. Fix: grounding instructions, check for context confusion
3. If the retrieved context doesn't contain the correct information → retrieval failure. Fix: chunking strategy, query expansion, metadata filtering. See `knowledge-injection-strategy`
4. If the retrieved context is ambiguous or partially relevant → fix both retrieval and grounding

This diagnostic requires observability — you need to log what was retrieved for each query. If you can't see the retrieved context, you can't diagnose the failure. See `agent-eval-framework` for production logging design.

---

## The Trust Calibration Problem

Hallucination is especially dangerous in expert domains (medical, legal, financial, specialized technical) for one reason: users in these domains often have enough expertise to trust confident-sounding output, but not enough to catch the specific errors.

A doctor using an AI medical assistant may recognize a plausible-sounding treatment recommendation but not have the specialized knowledge to recognize a subtle error in drug interaction information. The confidence of the output overrides their uncertainty.

This is the "confident-and-wrong is worse than uncertain-and-right" problem. A system that says "I'm not sure about this specific interaction — please verify with [authoritative source]" is more useful than one that produces a detailed but incorrect answer confidently.

**Design principle for expert-domain products:** The goal is not to maximize the number of questions the agent answers definitively. It is to maximize the number of questions the agent answers correctly — even if that means answering some questions with "I need to verify this" rather than a confident response.

---

## What Hallucination Is Not

**It is not always wrong.** The model's parametric memory is correct for the vast majority of common-knowledge queries. Hallucination is a specific failure mode for specific query types — not a general unreliability.

**It is not random.** Hallucination follows predictable patterns (parametric memory gaps, specific domain weaknesses, query types that invite confabulation). These patterns can be characterized and designed around.

**It is not solely a model quality problem.** Better models hallucinate less, but even the best current models hallucinate in predictable ways. Product design that compensates for these failure modes produces more reliable systems than better models alone.

**It is not fixed by telling the model not to hallucinate.** "Never hallucinate" in the system prompt is wishful instruction. It doesn't change the underlying statistical generation process. What works: retrieval grounding, uncertainty disclosure, scope limiting, source attribution.

---

## Related Skills

- `llm-capability-boundaries` — The broader landscape of what models can and can't do
- `knowledge-injection-strategy` — Retrieval grounding as the primary anti-hallucination technique
- `agent-eval-framework` — How to measure hallucination rates in production and catch regressions
- `system-prompt-engineering` — Encoding uncertainty disclosure as a first-class behavioral rule
