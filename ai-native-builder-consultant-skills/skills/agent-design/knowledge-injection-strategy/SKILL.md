---
name: knowledge-injection-strategy
description: Choose between RAG, fine-tuning, and context engineering for domain knowledge. Use when deciding how your agent will 'know' things, when responses feel generic despite good prompting, or scaling from MVP to production.
---

# Knowledge Injection Strategy


## When to Use

- Deciding how an agent will access domain knowledge — before building any retrieval or fine-tuning pipeline
- Agent responses feel generic or factually unreliable despite a good system prompt
- Scaling from MVP (context engineering) and need a sustainable knowledge architecture for production
- A RAG system is producing wrong answers and you need to diagnose whether it's a retrieval or model problem

## Ask These Questions First

1. Is the knowledge factual (product specs, policies, records) or behavioral (tone, reasoning patterns, domain judgment)?
2. How large is the knowledge base — fits in a context window (<50K words), or larger?
3. How frequently does the knowledge change — daily, weekly, monthly, or rarely?
4. Do you have labeled (input, ideal output) pairs for fine-tuning, and if so how many?

## Output Format
```markdown
## 知识架构决策

**产品：** [填入]  **日期：** [填入]

### 知识分类

| 知识域 | 类型 | 规模 | 更新频率 | 推荐方案 |
|--------|------|------|---------|---------|
| [知识域 1] | 事实型/行为型 | <50K / >50K 词 | 日/周/月 | 上下文工程 / RAG / Fine-tuning |
| [知识域 2] | 事实型/行为型 | <50K / >50K 词 | 日/周/月 | 上下文工程 / RAG / Fine-tuning |

### 三层架构分配

**Layer 1 — System Prompt（稳定知识）：**
- [永远不变的规则和行为模式]

**Layer 2 — 动态上下文（RAG / 实时注入）：**
- [按查询检索的知识域]
- 检索策略：[语义检索 / 结构化查询 / 混合]

**Layer 3 — 模型权重（Fine-tuning，如适用）：**
- [需要 Fine-tuning 的行为模式及原因]
- 训练数据规模：[当前 / 目标]

### RAG 关键设计（如适用）

| 维度 | 决策 |
|------|------|
| 分块策略 | 语义分块 / 结构化提取 / 固定窗口 |
| 检索方式 | 纯语义 / 元数据过滤 + 语义 / 精确匹配 |
| 已知失效风险 | [填入] |

### 评估计划

- 测试查询集：[20 条代表性问题]
- 准确率判断标准：[填入]
- 延迟要求：[p50 / p95 目标值]
```

---

## 咨询链位置

**在新版 SOP 中：** 模块 2「Agent 设计」和模块 4「技术决策」之间的桥梁，用来决定知识分层和实现方式。

**常见联动：**
- `system-prompt-engineering` — 决定哪些稳定规则进入 SOUL
- `data-foundation` — 数据盘点决定 RAG 或 fine-tuning 的可行性
- `agent-eval-framework` — 验证知识策略是否真的工作

---

---

## The Three Approaches

### Context Engineering (Prompt Stuffing)

Put the knowledge directly in the prompt — either in the system prompt (static knowledge) or assembled at request time (dynamic knowledge).

**How it works:**
- Static: facts, rules, and domain knowledge written directly into the system prompt
- Dynamic: relevant information retrieved and inserted into each message (this is the retrieval part of RAG, used without a vector database)

**Cost model:** Cheap to build, expensive to operate at scale (tokens per request × request volume)

**When it's right:**
- Knowledge set is small (< ~50,000 words fits comfortably in context)
- Knowledge is relatively static (changes weekly or monthly, not hourly)
- You're in MVP/prototype stage
- The knowledge is procedural ("here's how we do things") rather than factual ("here's our product catalog")

**When it breaks:**
- Knowledge grows beyond context window limits
- Knowledge updates frequently and you can't rebuild the prompt on every change
- Different users need different knowledge subsets (can't personalize easily)

---

### Retrieval-Augmented Generation (RAG)

Build an index of your domain knowledge. At query time, retrieve the most relevant chunks and inject them into the context.

**How it works:**
1. Chunk your knowledge into retrievable units
2. Embed each chunk into a vector space
3. At query time: embed the user's query, find nearest chunks, inject into context
4. Model answers using retrieved context + its own knowledge

**Cost model:** Moderate build cost (embedding pipeline, vector store), moderate query cost (embedding + retrieval + LLM), high maintenance cost if done poorly

**When it's right:**
- Large knowledge base (product catalog, policy library, documentation corpus)
- Knowledge updates frequently (retrieve fresh content on each query)
- Different queries need different knowledge subsets (retrieval handles this naturally)
- You need to cite sources or show the model's knowledge provenance

**When it breaks:**
- Retrieval quality is poor (wrong chunks retrieved → wrong answers with confidence)
- Knowledge requires reasoning across multiple chunks (retrieval is good at finding, bad at reasoning over a distributed set)
- Query-to-chunk semantic mismatch (user asks "what's the cheapest option?" — the relevant chunk says "Plan A costs ¥299/month" — the embedding similarity may be low because the question and answer don't share vocabulary)

> **Founder pattern from a constraint-heavy product catalog:** The first version of the product knowledge layer used naive RAG: embed product descriptions, retrieve by cosine similarity. It worked for specific product lookups ("tell me about option X") and failed for comparison queries ("which option is best for a user with constraint Y?"). The failure mode was consistent: retrieval surfaced the top few products by embedding similarity, but the model had no way to reason about the interaction between the user's profile and the domain-specific eligibility rules, because those rules lived in different chunks and were not retrieved together.
>
> The fix was a hybrid: structured metadata on each product chunk (age range, health conditions, key exclusions) + filtered retrieval (filter first by hard constraints, then rank by semantic similarity). Structured filtering > pure embedding similarity for constraint-heavy domains.

---

### Fine-Tuning

Train the model on examples from your domain to internalize knowledge and behavioral patterns.

**How it works:**
- Prepare a dataset of (input, ideal output) pairs from your domain
- Train the model to update its weights toward those examples
- Deploy the fine-tuned model as your agent's backbone

**Cost model:** High build cost (dataset curation + training compute), zero marginal query cost for the knowledge itself, high maintenance cost if domain changes

**When it's right:**
- Behavioral pattern is consistent and hard to capture in a prompt (the model needs to *be* a certain way, not just *follow rules* about being a certain way)
- Response format or style is highly specific and consistent (structured outputs, domain-specific formatting)
- Latency is critical and context window usage must be minimized
- You have thousands of high-quality labeled examples
- The domain is stable enough that retraining every 3–6 months is acceptable

**When it breaks:**
- Your dataset is small (< ~1,000 high-quality examples — fine-tuning with insufficient data is worse than not fine-tuning)
- Domain knowledge changes frequently (fine-tuned knowledge is stale the day after training)
- You don't have labeled examples — only documents (RAG is better for document-based knowledge)
- You're trying to teach facts, not behaviors (models memorize facts poorly from fine-tuning; RAG is better for factual knowledge)

---

## The Decision Framework

Answer these questions in order:

### Q1: Is your knowledge factual or behavioral?

**Factual knowledge:** Things the agent needs to *know* — product specs, policies, customer records, documentation. The ground truth exists in a document or database somewhere.

→ Factual knowledge belongs in RAG or context engineering, not fine-tuning.

**Behavioral knowledge:** How the agent should *act* — tone, reasoning patterns, domain-specific judgment, response format. "An experienced domain professional would ask clarifying questions about the client before making a recommendation."

→ Behavioral knowledge belongs in fine-tuning or detailed system prompt examples, not RAG.

Most products need both. Separate the architectural decisions for each.

---

### Q2: How large and how dynamic is your factual knowledge base?

| | Small (< 50K words) | Large (> 50K words) |
|---|---|---|
| **Static** (changes monthly or less) | Context engineering (system prompt) | RAG with infrequent reindex |
| **Dynamic** (changes daily or more) | Dynamic context insertion | RAG with continuous ingestion pipeline |

---

### Q3: Does your behavioral knowledge need to generalize to unseen inputs, or just handle known patterns?

**Known patterns:** "Format policy summaries like this. Handle renewal objections like this. Escalate compliance questions like this." — These can be captured in system prompt examples. Fine-tuning is not necessary.

**Generalization needed:** "The agent should reason like a senior underwriter across any case it encounters, even novel ones." — This requires either fine-tuning on diverse, representative examples or a very carefully designed chain-of-thought system prompt.

If you're uncertain, start with system prompt examples. Fine-tune only when you can measure that system prompt examples aren't generalizing well enough.

---

### Q4: What's your data situation?

Run through this checklist:

```
Do you have labeled (input, ideal output) pairs?
├─ No → Fine-tuning is not ready. Use RAG + context engineering.
└─ Yes → How many?
          ├─ < 500 → Too few. Use as system prompt examples instead.
          ├─ 500–2,000 → Marginal. Can fine-tune for format/style; 
          │              not for substantive knowledge.
          └─ > 2,000 → Fine-tuning is viable for the covered patterns.

Do you have a document corpus (policies, product docs, etc.)?
├─ No → Context engineering with manually curated knowledge.
└─ Yes → How large?
          ├─ < 100 documents → Context engineering or simple RAG.
          └─ > 100 documents → Full RAG pipeline warranted.
```

---

## Designing a Hybrid Strategy

Most production AI Native products end up with a hybrid architecture. Here's how to think about the layers:

```
┌─────────────────────────────────────────────────┐
│  System Prompt (Layer 1)                        │
│  • Agent identity and behavioral rules          │
│  • Static domain rules that never change        │
│  • Few-shot examples of correct reasoning       │
├─────────────────────────────────────────────────┤
│  Dynamic Context (Layer 2)                      │
│  • Retrieved documents (RAG)                    │
│  • User-specific context (profile, history)     │
│  • Session state                                │
├─────────────────────────────────────────────────┤
│  Model Weights (Layer 3)                        │
│  • Base model capabilities                      │
│  • Fine-tuned behavioral patterns (if any)      │
└─────────────────────────────────────────────────┘
```

**The principle:** Push knowledge to the highest layer possible.

Layer 1 is cheapest to query, most reliable, easiest to update (edit a text file). Use it for everything that fits.

Layer 2 is flexible, scalable, and fresh. Use it for factual knowledge that's too large for Layer 1 or changes frequently.

Layer 3 is expensive to change. Use it only for behaviors that can't be expressed in layers 1 or 2.

---

## RAG Quality: The Two Problems Nobody Talks About

RAG tutorials focus on embedding and retrieval. Production failures usually come from these two things instead:

### Problem 1: Chunking strategy

The model can only answer questions from the chunks it retrieves. If your chunking strategy breaks information across chunk boundaries, the model will never have access to the complete information it needs.

**Naive chunking:** Split document every N characters. Fast, simple, breaks logical units constantly.

**Better approaches:**
- Semantic chunking: split at logical boundaries (sections, paragraphs, list items)
- Hierarchical chunking: chunk at multiple granularities, retrieve at the right level for the query
- Structured extraction: for structured data (tables, product specs), extract into structured formats rather than free text chunks

> **Founder pattern:** Insurance product sheets have a specific structure — coverage details, exclusions, pricing, underwriting requirements. Naive chunking would put half the exclusions in one chunk and half in another. We switched to template-based extraction: parse each product sheet into a structured JSON object with explicit fields. Retrieval became exact lookup by field rather than semantic search. Zero false positives. The lesson: for structured knowledge, structure the extraction, don't just chunk the text.

### Problem 2: Query-document semantic mismatch

Embedding similarity measures the semantic distance between a query and a document chunk. But user queries are often in the vocabulary of questions ("what's the cheapest?"), while documents are in the vocabulary of answers ("Plan A: ¥299/month"). These can be semantically distant even when they're informationally matched.

**Fixes:**
- Hypothetical document embedding (HyDE): generate a hypothetical answer to the query, embed the answer, retrieve documents similar to the hypothetical answer rather than the raw query
- Metadata filtering: add hard-constraint filters (e.g., "only retrieve documents tagged for this user's risk profile") before semantic ranking
- Query expansion: rewrite the query in multiple forms before retrieval (increases recall at the cost of compute)

---

## Maturity-Stage Recommendations

**Stage 1: MVP (0–100 users)**

Use context engineering only. Write your domain knowledge into the system prompt. This forces you to be explicit about what the agent knows, is fast to iterate, and doesn't require infrastructure.

You will likely hit the limits of this approach. That's intentional — hitting the limits tells you exactly what to build next.

**Stage 2: Early product (100–1,000 users)**

Add RAG for the specific knowledge domains where context engineering is breaking. Build the simplest retrieval pipeline that works — a flat vector store with good chunking is usually enough.

Don't fine-tune yet. You don't have enough production data to build a good training set.

**Stage 3: Scaling (1,000+ users)**

Now you have production data. Review the logs. Where is the agent consistently wrong? Where do users edit or override most frequently? These are your fine-tuning candidates — if they're behavioral patterns, not factual gaps.

Build a continuous ingestion pipeline for your RAG if knowledge changes frequently. Invest in retrieval quality (chunking strategy, metadata, query expansion) rather than just retrieval volume.

---

## The Evaluation Test

Before choosing an approach, define how you'll measure whether it's working.

```markdown
## Knowledge evaluation spec

Test query set: [20–50 representative questions covering edge cases]

For each approach, measure:
- Answer accuracy: [how you'll judge correct vs incorrect]
- Failure mode profile: [what kinds of errors does each approach make?]
- Latency: [p50 and p95 response time]
- Cost per query: [token count × rate for context approaches]

Decision criteria: [the specific accuracy/latency/cost tradeoff you'll optimize for]
```

Don't choose an approach before you can answer: "How will I know, in production, whether this is working?"

---

## Related Skills

- `agent-boundary-design` — Scope what the agent needs to know before designing how it knows it
- `system-prompt-engineering` — Layer 1 of the hybrid knowledge architecture
- `agent-eval-framework` — How to measure whether your knowledge strategy is actually working
