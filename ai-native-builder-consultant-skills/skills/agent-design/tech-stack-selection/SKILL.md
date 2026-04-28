---
name: tech-stack-selection
description: Choose the right framework, model, and infrastructure for an AI Native product. Use when starting a new build, evaluating whether to switch stacks, or when current choices are creating latency, cost, or maintainability problems.
---

# Tech Stack Selection

## When to Use

- Starting a new AI Native product and need to make foundational technology choices
- Evaluating whether to switch from one framework or model to another
- Current stack is causing production problems (latency, cost, reliability) and you need to diagnose whether it's a stack problem or a design problem
- Client is asking "which LLM should we use" without a framework for answering it

## Ask These Questions First

1. What is the primary task — generation, retrieval, classification, orchestration, or a combination? The task type determines the model family before anything else.
2. Where does the product run — cloud, edge device, on-premise, or hybrid? Deployment constraint eliminates large parts of the option space immediately.
3. What are the hard constraints — latency (p95 requirement), cost per call, data residency, regulatory compliance?
4. What does the team already know? A stack the team can debug beats a theoretically superior stack they can't.
5. Is this a prototype/MVP or a production system? The right answer is different for each stage.

## Output Format

```markdown
## Tech Stack Decision

**Product:** [填入]  **Stage:** MVP / Production  **Date:** [填入]

### Hard Constraints (eliminates options before evaluation)
| Constraint | Value | Eliminates |
|-----------|-------|-----------|
| Latency (p95) | [e.g. <2s] | [models/frameworks that can't meet this] |
| Cost per call | [e.g. <¥0.01] | [models that exceed budget] |
| Deployment | [cloud/edge/on-prem] | [options incompatible with deployment] |
| Data residency | [e.g. China only] | [foreign-hosted options if applicable] |

### Model Selection

| Tier | Recommended | Why | When to use |
|------|------------|-----|-------------|
| Primary | [model] | [specific reason for this task] | [use case] |
| Fallback | [model] | [cost/speed tradeoff] | [use case] |
| Ruled out | [model] | [specific reason eliminated] | — |

### Framework Selection

| Layer | Choice | Why not alternatives |
|-------|--------|---------------------|
| Agent orchestration | [e.g. OpenClaw / LangGraph / custom] | [specific reason] |
| Retrieval | [e.g. Chroma / Milvus / pgvector] | [specific reason] |
| Serving | [e.g. FastAPI / vLLM] | [specific reason] |
| Deployment | [e.g. Jetson / Alibaba Cloud / self-hosted] | [specific reason] |

### Infrastructure

| Component | Choice | Scale trigger to revisit |
|-----------|--------|--------------------------|
| Vector store | [choice] | [when to upgrade] |
| Caching | [choice] | [when to add] |
| Queue | [choice] | [when to add] |

### Decision rationale (one paragraph)
[Why this combination, for this team, at this stage. What we're trading off and why that's acceptable.]

### Revisit conditions
- [ ] If latency p95 > [X]ms consistently → evaluate [alternative]
- [ ] If monthly inference cost > ¥[X] → evaluate [cheaper model/caching strategy]
- [ ] If team grows to [N] engineers → evaluate [more structured framework]
```

## 咨询链位置

**在新版 SOP 中：** 模块 4「技术决策与实施计划」的核心技能，在边界、知识和数据前提基本明确后使用。

**常见联动：**
- `knowledge-injection-strategy` — 技术栈决定知识架构的实现成本
- `llm-capability-boundaries` — 用能力边界校验模型选择是否现实
- `multi-agent-orchestration` — 当系统复杂度升高时决定是否需要多 Agent

---

## The Selection Framework

### Step 1: Lock constraints first, evaluate options second

Before comparing any two models or frameworks, write down the constraints that eliminate options outright. Latency requirements eliminate large models. Edge deployment eliminates cloud-only APIs. Data residency eliminates foreign-hosted providers. Cost ceilings eliminate frontier models for high-volume tasks.

Most "which model should I use" discussions skip this step and go straight to capability comparison. That's backwards. Constraints eliminate 70% of the option space before any capability evaluation is needed.

### Step 2: Match model to task, not to benchmark

Benchmarks measure general capability. Your task is specific. The right question is not "which model scores highest on MMLU" but "which model handles [your specific task type] reliably within your constraints."

**Task-to-model family mapping:**

| Task type | What to optimize for | Typical choice |
|-----------|---------------------|----------------|
| Structured output (JSON, extraction) | Format adherence, reliability | Smaller fine-tuned model often beats larger general model |
| Long-context reasoning | Context window, reasoning quality | Frontier models (Qwen-Long, Claude, GPT-4o) |
| High-volume classification | Speed, cost | Small distilled model, possibly fine-tuned |
| Creative generation | Output quality, diversity | Frontier models |
| Tool calling / agent actions | Tool use reliability | Models with strong function-calling training |
| Chinese-language tasks | Chinese corpus quality | Qwen family often outperforms non-Chinese-native models |

### Step 3: Choose the simplest framework that fits

Framework selection is where teams most often over-engineer. The rule: use the simplest thing that handles your actual requirements, not the most featureful thing available.

**Framework complexity ladder:**

```
Raw API calls          ← start here if task is simple
    ↓ when you need: prompt management, retry logic
LLM SDK (official)
    ↓ when you need: multi-step workflows, tool calling
Lightweight orchestration (custom or minimal framework)
    ↓ when you need: multi-agent, complex state, production observability
Full framework (LangGraph, OpenClaw, CrewAI)
    ↓ only when: team is large, workflow is genuinely complex, switching cost is acceptable
Managed platform
```

**The most common mistake:** jumping to a full framework at week one because it "seems more professional." Full frameworks add abstraction, debugging complexity, and upgrade dependencies. Pay that cost only when the alternative is worse.

### Step 4: Deployment constraint shapes everything

**Cloud (standard):** Full model size range available. Latency limited by network. Cost scales with tokens. Simplest to start.

**Edge (Jetson, mobile, embedded):** Model size hard-constrained by device TOPS. Quantization required (INT8/INT4). Frameworks must support ONNX or TensorRT export. Inference libraries: TensorRT-LLM, llama.cpp, MLC-LLM.

**On-premise (enterprise, compliance):** Self-hosted models only. Serving infrastructure (vLLM, Ollama, TGI) required. GPU procurement lead time is a project risk.

**China deployment specifics:**
- API providers: Alibaba Qwen (DashScope), Baidu ERNIE, Zhipu GLM, Moonshot, DeepSeek
- Data residency: all data stays in China, foreign-hosted APIs not viable for regulated industries
- ICP filing affects which cloud providers are options
- Feishu/DingTalk/WeCom integration favors Alibaba Cloud ecosystem for operational simplicity

### Step 5: Plan for the switching cost

Every stack choice creates switching cost. The question is not "will we ever switch" but "how painful will switching be, and does our current choice minimize that pain if we need to?"

**Lower switching cost:** Keep business logic separate from LLM calls. Use an abstraction layer (even a thin one) between your code and the model API. Treat model choice as configuration, not as hardcoded dependency.

**Higher switching cost:** Framework lock-in (if your orchestration logic is deeply tied to one framework's abstractions), fine-tuned models (retraining cost), vector store schema (migration complexity).

## Common Anti-Patterns

**"We'll use GPT-4 for everything"**
Frontier models are expensive and slow for high-volume, simple tasks. Use them for tasks that actually need their capability. Use smaller, faster, cheaper models for classification, routing, and structured extraction.

**"We need RAG, let's set up Milvus"**
Milvus is a production vector database. For a prototype with 1,000 documents, SQLite with pgvector or even in-memory FAISS is sufficient and requires zero operational overhead. Scale when you have the load, not in anticipation of it.

**"Let's use LangChain because everyone uses LangChain"**
LangChain is a general-purpose framework with a large API surface. Its abstractions often obscure what's actually happening in LLM calls, making debugging harder. For many production use cases, a thin custom wrapper around the official SDK is more maintainable. Evaluate whether the framework's features actually match your requirements.

**"The model we fine-tuned is now our stack"**
Fine-tuned models are expensive to retrain and can't easily incorporate new base model improvements. Use fine-tuning only when the behavior genuinely can't be achieved with prompting, and treat the fine-tuned model as a component that requires a maintenance plan.

## Related Skills

- `agent-boundary-design` — Scope defines what the stack needs to support
- `knowledge-injection-strategy` — RAG pipeline is part of the stack decision
- `llm-capability-boundaries` — Understand model limits before selecting
- `agent-eval-framework` — How to measure whether stack choices are working in production
