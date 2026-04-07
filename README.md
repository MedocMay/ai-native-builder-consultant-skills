# AI Native Builder Consultant Skills

> The skills marketplace for people **building** AI Native products — not just using them.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)
[![Author](https://img.shields.io/badge/Author-Medoc%20May-teal?style=flat-square)](https://github.com/MedocMay/ai-native-builder-consultant-skills)

---

## Start Here

This repository is a **consulting system for AI Native product building**.

It gives you three things:

- `Skills` for making hard product decisions
- `Templates` for turning those decisions into client-ready deliverables
- `Workflow` docs for running a full consulting engagement from positioning to launch and iteration

If you're new here, use this order:

1. Read [CONSULTING-WORKFLOW.md](CONSULTING-WORKFLOW.md)
2. Skim [TERMINOLOGY.md](TERMINOLOGY.md)
3. Start with `agent-boundary-design`
4. Use the [`templates/`](templates/) folder to capture outputs as formal deliverables

### Core Outputs

By the end of a full engagement, you should have most of these artifacts:

- `AI Positioning Statement`
- `Agent Boundary Spec`
- `SOUL.md`
- `HITL Design Spec`
- `Knowledge Architecture Decision`
- `Trust-Autonomy Calibration Plan`
- `Data Foundation Plan`
- `Eval Plan + Production Review Protocol`
- `Tech Stack Decision`
- `Launch Risk Review`
- `Iteration Flywheel Design`

---

## Why This Repo Exists

There are great Skills repos for *doing PM work with AI*. There are great Skills repos for *writing code with AI*.

There is nothing for the person sitting in front of a blank whiteboard trying to answer: **"How do I actually design and build this AI Native thing?"**

Not "how do I prompt better." Not "what's an LLM." Not "here's a PRD template."

The hard questions. The ones that matter.

- Where does my agent end and the human begin?
- Should I use RAG or fine-tuning? How do I decide?
- Why does my agent feel impressive in demos but break in production?
- How do I design trust into an AI product, not just bolt it on?
- What does "AI Native UX" actually mean in practice?

These aren't questions you find answers to in a book. They're questions you answer by building, shipping, breaking things, and building again.

This repo is the distillation of that process.

---

## Who this is for

**You're the right person if:**

- You've decided to build an AI Native product and need to make real architectural decisions
- You're a technical founder or PM who works directly on the product — not someone who delegates to "the AI team"
- You want decision frameworks, not generic advice
- You're comfortable with Claude Code, OpenClaw, or similar agent environments

**You're not the right person if:**

- You're looking for prompting tips for everyday tasks
- You want templates to fill in
- You haven't yet decided to build something — this repo assumes you're already building

---

## What's inside

Skills are organized into three layers, each building on the previous:

### Layer 1 — Mental Models
*The concepts you need before any decision makes sense.*

| Skill | What it answers |
|-------|----------------|
| `ai-native-vs-ai-enhanced` | Is my product actually AI Native, or am I just adding a chatbot? |
| `agent-loop-model` | What does an agent actually do, at the mechanical level? |
| `llm-capability-boundaries` | What can I rely on the model for, and what can't I? |
| `uncertainty-and-hallucination` | Why does my agent lie, and what do I do about it? |

### Layer 2 — Design Decisions
*The choices that determine whether your product works.*

| Skill | What it answers |
|-------|----------------|
| **`agent-boundary-design`** ← start here | Where does my agent end? What should it own vs delegate? |
| `system-prompt-engineering` | How do I write a system prompt that holds up in production? |
| `knowledge-injection-strategy` | RAG, fine-tuning, or context engineering — how do I choose? |
| `human-in-the-loop-design` | How do I design the human confirmation experience? |
| `ai-native-ux-patterns` | What does AI Native UX look like beyond the chat box? |
| `tech-stack-selection` | How do I choose a framework, model, and infrastructure? |
| `trust-and-autonomy-tradeoff` | How much should my agent do vs ask? |

### Layer 3 — Build Skills
*The craft of actually building it well.*

| Skill | What it answers |
|-------|----------------|
| `data-foundation` | Where does the data come from, how is it labeled, and how do we build an eval set? |
| `agent-eval-framework` | How do I know if my agent is actually working? |
| `multi-agent-orchestration` | How do I design agents that hand off to each other reliably? |
| `launch-risk-review` | What do I check before putting this in front of real users? |
| `iteration-flywheel` | How do I build the data → model → product feedback loop? |

---

## Quick start

This repository is published as a **portable skills library**. You can use it in three ways:

### 1. Read and copy a single skill

Open any `SKILL.md` file and paste it into your AI workflow as context.

### 2. Import into a local skills directory

If your agent environment supports local skills folders, copy the contents of the relevant `skills/` directory into your local skills path.

```bash
# Example: copy all skills from one collection
cp -R agent-design/skills/* /path/to/your/local/skills/
```

### 3. Use as a reference repo during product design

Work through the skills in sequence and turn each output into a decision artifact for your product team.

If you want the project-level method first, start with [CONSULTING-WORKFLOW.md](CONSULTING-WORKFLOW.md).  
If you want standardized artifact names, see [TERMINOLOGY.md](TERMINOLOGY.md).  
If you want ready-to-fill deliverables, open the [`templates/`](templates/) folder.

---

## Recommended Path

If you want the shortest practical route through the repository:

```text
ai-native-vs-ai-enhanced
  → agent-boundary-design
  → system-prompt-engineering
  → human-in-the-loop-design
  → knowledge-injection-strategy
  → data-foundation
  → agent-eval-framework
  → tech-stack-selection
  → launch-risk-review
  → iteration-flywheel
```

Use `multi-agent-orchestration` only when system complexity justifies it.

---

## How to use these skills

In tools that support local or imported skills, relevant skills may load automatically. You can also invoke them explicitly:

```
Use the agent-boundary-design skill to help me design the scope of my document processing agent.
```

**Skills chain naturally.** A typical design session might flow:

```
agent-boundary-design → human-in-the-loop-design → system-prompt-engineering → agent-eval-framework
```

**Skills are designed for dialogue, not monologue.** Each skill will ask you clarifying questions — about your specific product, users, and constraints — before giving recommendations. Generic advice is useless. These skills are built to give you specific answers to your specific situation.

---

## What makes this different

Most skills repos encode *frameworks from books*. This one encodes *decisions made while shipping*.

Every skill in this repo is grounded in at least one real product decision with a real consequence:

- A vertical-domain professional copilot built on OpenClaw-style workflows — shaped by thousands of real user interactions
- The multi-channel agent platform (Shrimper) — connecting Feishu, DingTalk, WeCom, Telegram, Discord
- Client projects: education data agents, digital human systems, enterprise SaaS pilots

When a skill says "this approach breaks in production," it's because it actually broke in production. When it says "this pattern builds user trust," it's because we measured it.

**This is not "best practices from the internet." This is what we learned the hard way.**

---

## The philosophy

Three principles guide every skill in this repo:

**1. Decision first, explanation second.**
Every skill starts from a decision you need to make. The framework exists to help you make that decision — not to teach you the topic comprehensively. If you want a textbook, there are better resources.

**2. Specific beats general.**
Skills ask about your product, your users, your constraints. The output should be a recommendation for *your situation*, not a generic framework you still have to translate.

**3. Production is the test.**
Demos lie. Skills are calibrated against production behavior — what actually breaks, what actually builds trust, what actually scales. If a pattern looks great in a demo but breaks in production, we say so explicitly.

---

## 中文支持 | Chinese

本 repo 的所有核心 Skills 提供中英双语版本。

中文 AI 创业者面临的挑战和英文世界有交叉，但也有独特之处：合规要求（ICP、等保、数据本地化）、主流平台（飞书、钉钉、微信生态）、用户信任曲线（中国用户对 AI 产品的接受度和信任建立方式不同）。

中文版 Skills 会在相关决策点融入这些本土化考量，而不是机械翻译。

---

## Repository Map

- `mental-models/`: foundational framing and capability boundaries
- `agent-design/`: the core product and agent-design decisions
- `build-skills/`: validation, launch, and iteration practices
- `templates/`: ready-to-fill consulting deliverables
- `CONSULTING-WORKFLOW.md`: project-level consulting sequence
- `TERMINOLOGY.md`: naming and translation standard

---

## Status

This repo is early and growing. Current state:

**Layer 2 — Design Decisions (agent-design plugin)**
- [x] `agent-boundary-design` — complete
- [x] `system-prompt-engineering` — complete
- [x] `human-in-the-loop-design` — complete
- [x] `knowledge-injection-strategy` — complete
- [x] `ai-native-ux-patterns` — complete
- [x] `tech-stack-selection` — complete
- [x] `trust-and-autonomy-tradeoff` — complete

**Layer 3 — Build Skills (build-skills plugin)**
- [x] `data-foundation` — complete
- [x] `agent-eval-framework` — complete
- [x] `multi-agent-orchestration` — complete
- [x] `launch-risk-review` — complete
- [x] `iteration-flywheel` — complete

**Layer 1 — Mental Models (mental-models plugin)**
- [x] `ai-native-vs-ai-enhanced` — complete
- [x] `agent-loop-model` — complete
- [x] `llm-capability-boundaries` — complete
- [x] `uncertainty-and-hallucination` — complete

**The plan:** Each skill produces a named, signable deliverable — not a knowledge summary. In practice, the consulting system runs as a decision chain:

```
Positioning:
ai-native-vs-ai-enhanced     → AI Positioning Statement

Agent design:
agent-boundary-design        → Agent Boundary Spec  ← start here
system-prompt-engineering    → SOUL.md
human-in-the-loop-design     → HITL Design Spec
knowledge-injection-strategy → Knowledge Architecture Decision
trust-and-autonomy-tradeoff  → Trust-Autonomy Calibration Plan

Data and eval foundation:
data-foundation              → Data Foundation Plan
agent-eval-framework         → Eval Plan + Production Review Protocol

Technical decision:
tech-stack-selection         → Tech Stack Decision
multi-agent-orchestration    → Multi-Agent Architecture Spec (if needed)

Launch and iteration:
launch-risk-review           → Launch Risk Review
iteration-flywheel           → Flywheel Design (post-launch)
```

Some deliverables are mandatory, some are conditional, but the rule is consistent: each major decision becomes a document the client can review, sign off on, and operate from.

You can find starter versions of those deliverables in the [`templates/`](templates/) folder.

---

## Contributing

If you've built an AI Native product and have a decision framework worth sharing — open an issue. The bar is: must be grounded in production experience, must answer a specific decision (not explain a concept), must have seen the failure modes as well as the successes.

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

## License

Apache 2.0. Use freely. Attribution appreciated.

---

*Built by [Medoc May](https://github.com/MedocMay/ai-native-builder-consultant-skills) — a founder who spent two years making every mistake in this repo so you don't have to.*
