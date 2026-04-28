---
name: llm-capability-boundaries
description: Map what LLMs do reliably, unreliably, and cannot do. Use before scoping agent capabilities, when debugging surprising model behavior, or deciding whether a task needs a tool vs. the model alone.
---

# LLM Capability Boundaries


## When to Use

- Deciding whether to trust the model with a specific task before building it
- Debugging a production failure and need to distinguish model problem from product design problem
- Scoping agent capabilities for a new product and need a realistic picture of what to promise users
- Choosing between a pure LLM approach and a hybrid approach (LLM + deterministic tool)

## Ask These Questions First

1. What is the specific task — generation, extraction, classification, reasoning, recall, or arithmetic?
2. What happens when the model is wrong — is the error visible, auditable, or silent?
3. Is this task high-frequency (many calls, failure rate matters) or low-frequency (acceptable to review manually)?
4. Does the task require current or specialized information the model likely doesn't have in training?

## Output Format

A **Capability Assessment** per task type:
- Reliability rating: high / medium / low
- Primary failure mode (if any)
- Recommended design approach: use freely / validate output / route to tool / require human review
- Verdict: model problem or product design problem (with diagnostic reasoning)

---

## Why This Matters for Product Design

The most expensive mistakes in AI Native product development come from one of two mismatches:

**Overestimating the model:** Trusting the LLM with a task it cannot reliably do — high-precision arithmetic, reasoning over very long contexts, perfectly consistent behavior across thousands of users. The product ships, fails in production, and erodes trust.

**Underestimating the model:** Building brittle rule-based systems for tasks the model handles gracefully — nuanced language understanding, adapting tone to context, recognizing intent across varied phrasings. Unnecessary complexity, slower iteration, worse user experience.

Neither comes from the model being "good" or "bad." Both come from not knowing where the boundary is.

---

## What LLMs Do Reliably Well

These are the capabilities you can build product features on with high confidence:

### Language Understanding and Generation
- Understanding intent across varied phrasings, typos, mixed languages, incomplete sentences
- Generating fluent, contextually appropriate text in a given style or register
- Translating between languages while preserving nuance
- Summarizing long documents into concise, accurate abstracts
- Reformatting content (structured → prose, prose → structured, formal → informal)

**Design implication:** Use the model freely for any task where the core value is language fluency. Don't build rule-based parsers for things the model understands naturally.

### In-Context Reasoning
- Following multi-step logical instructions given in the prompt
- Applying a provided framework to a new situation
- Identifying contradictions or gaps in a given set of information
- Explaining its reasoning step by step when asked

**Design implication:** You can teach the model complex decision logic via the system prompt and few-shot examples. You don't need fine-tuning for most behavioral customization.

### Pattern Recognition Across Provided Context
- Identifying themes, sentiments, or patterns in a set of documents
- Classifying inputs according to a taxonomy you've defined
- Extracting structured data from unstructured text (with accuracy caveats — see below)
- Comparing options against criteria and recommending

**Design implication:** The model can act as a sophisticated pattern recognizer on the context you provide. Quality depends heavily on the quality and relevance of what you inject.

### Adaptive Communication
- Calibrating tone and complexity to an inferred audience
- Maintaining a specified persona consistently within a session
- Adjusting communication style based on prior turns in the conversation

**Design implication:** The model can carry and maintain a behavioral identity (see `system-prompt-engineering`) without explicit rule-matching for every scenario.

---

## What LLMs Do Unreliably

These are the capabilities where the model will sometimes work and sometimes fail — often without obvious signals about which it's doing. Product designs that depend on these capabilities reliably will have production problems.

### Precise Arithmetic and Counting
The model can do mental math and get approximate answers. It cannot reliably do multi-step precise arithmetic — especially with large numbers, percentages compounded over many steps, or counting items in long lists.

**The failure pattern:** The model produces a plausible-looking number with confidence. The user assumes it's correct. It's off by a non-trivial amount.

**Design fix:** Never use the model for calculations you'd use a calculator for. Route arithmetic to a code interpreter tool or a deterministic function. The model can set up the calculation; a tool should execute it.

### Factual Recall for Long-Tail or Recent Information
The model has strong recall of high-frequency information in its training data. It has poor recall of obscure facts, recent events (post knowledge cutoff), and information that was rare or inconsistently represented in training data.

**The failure pattern:** The model produces a confident, plausible-sounding answer that is factually wrong. For common topics this is rare; for niche domains it can be frequent.

**Design fix:** Don't rely on the model's parametric memory for domain-specific facts. Use RAG to provide the facts; use the model to reason about them. See `knowledge-injection-strategy`.

### Consistent Behavior at Scale
A single LLM call behaves predictably. One thousand LLM calls on similar inputs will show variance — some will behave differently, especially on edge cases, adversarial inputs, or situations where the system prompt is ambiguous.

**The failure pattern:** The product works in testing (small N) and develops inconsistent behavior in production (large N). The inconsistency is rarely catastrophic — it's usually "occasionally worse" rather than "sometimes totally wrong" — but it accumulates into trust erosion.

**Design fix:** Design explicit handling for the tails of the distribution. What happens when the model does something unexpected? Eval frameworks (see `agent-eval-framework`) catch these issues systematically. Behavioral rules in the SOUL narrow the variance.

### Exact Format Compliance Under Pressure
The model can produce structured output (JSON, XML, specific templates) reliably in normal conditions. Under edge cases — unusually long inputs, complex nested structures, contradictory instructions — it will sometimes deviate.

**The failure pattern:** The output parser breaks because the model produced nearly-correct JSON with one malformed field. The agent errors out silently.

**Design fix:** Always validate structured output. Use output parsers with repair logic (try to fix minor formatting errors before erroring). Design for graceful degradation when parsing fails.

### Sustained Attention Across Very Long Contexts
Models have a documented "lost in the middle" problem — information in the middle of a very long context receives less attention than information at the beginning and end. In practical terms: important instructions buried in a 50,000-token context may be partially ignored.

**The failure pattern:** The system prompt has twenty important rules. The model follows the first five and the last two reliably. Rules in the middle are followed inconsistently.

**Design fix:** Put the most critical instructions at the top of the system prompt and reinforce them at the bottom. Keep system prompts focused — every token added competes for attention. Use the five-layer SOUL structure from `system-prompt-engineering` to prioritize what lives where.

---

## What LLMs Cannot Do

These are hard limits — not "do unreliably" but "cannot do by design."

### Access Real-Time or Post-Training Information
The model's knowledge is frozen at the training cutoff. It cannot know about events, data, or facts that occurred after training without being explicitly told in the context.

**Design implication:** Any task requiring current information requires retrieval. This is not a limitation to work around — it's a design constraint to design with.

### Maintain State Between Sessions
Unless you explicitly persist and inject state, the model starts each session with no memory of previous ones. The context window is ephemeral.

**Design implication:** Persistent memory is always a product feature you build, not a model capability you get for free. See the memory architecture in `agent-loop-model`.

### Guarantee Determinism
The same input can produce different outputs on different calls (especially with temperature > 0). This is by design — it enables creative and varied outputs — but it means you cannot guarantee exact reproducibility.

**Design implication:** If your product requires deterministic outputs (audit logs, compliance records, reproducible reports), generate the output once, store it, and serve the stored version. Don't regenerate.

### Reason About Its Own Uncertainty Reliably
The model can express uncertainty ("I'm not sure about this") but cannot reliably self-assess when it should. It will sometimes express high confidence about wrong answers and sometimes express unnecessary uncertainty about correct ones.

**Design implication:** Don't design products where the model's self-expressed confidence is the primary signal for user trust. Build external validation: eval frameworks, retrieval grounding, human review for high-stakes outputs.

---

## The "Model Problem" vs "Product Design Problem" Diagnostic

When an agent behaves badly in production, there are two explanations:

**Model problem:** The model genuinely cannot do this task reliably — it's outside the reliable capability boundary. Solution: route to a different tool, add human review, or remove the feature.

**Product design problem:** The model could do this task, but the context is wrong, the instructions are ambiguous, the task is too large for one call, or the failure handling is missing. Solution: fix the design.

Most production failures are product design problems, not model problems. Before concluding the model "can't do this," run through:

1. **Context check:** Does the model have the information it needs? Is anything important missing or truncated?
2. **Instruction check:** Is the task unambiguous in the system prompt? Are there conflicting instructions?
3. **Scope check:** Is the task too large for a single LLM call? Would breaking it into steps improve reliability?
4. **Failure handling check:** When the model produces a wrong answer, does the system catch it and recover, or does it silently propagate?

If all four check out and the model still fails — then it's a model boundary problem, and you need a different architecture for that task.

---

## The Calibration Table

A practical reference for product design decisions:

| Task type | Reliability | Design approach |
|-----------|-------------|-----------------|
| Natural language understanding | High | Use freely, minimal validation |
| Text generation (fluency, tone) | High | Use freely, human review for high-stakes |
| Classification with defined categories | High | Use with validation, spot-check sample |
| Extraction from structured text | Medium-high | Validate output format, spot-check facts |
| Multi-step reasoning (in-context) | Medium | Break into steps, validate intermediate outputs |
| Arithmetic / precise calculation | Low | Route to code execution tool |
| Factual recall (niche / recent) | Low | Use RAG, never rely on parametric memory alone |
| Consistent behavior at scale | Medium | Eval framework, behavioral rules in SOUL |
| Self-assessment of uncertainty | Low | External validation, don't rely on model's confidence signal |

---

## Related Skills

- `uncertainty-and-hallucination` — The specific failure mode where the model produces confident wrong output — causes, patterns, and fixes
- `agent-loop-model` — How capability limits interact with the agent architecture
- `knowledge-injection-strategy` — How to compensate for factual recall limits with RAG
- `agent-eval-framework` — How to measure where your specific agent is hitting capability limits
