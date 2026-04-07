# Consulting Workflow

This document explains how to run an AI Native consulting engagement using this repository.

The repository has two layers:

- `SOP`: the project-level operating system
- `Skills`: the decision modules used at each key step

The rule is simple:

1. Move the project forward by module
2. Invoke the relevant skill at each decision point
3. Turn the result into a named deliverable
4. Use that deliverable as an input to later decisions

---

## The Six Modules

### Module 1: Positioning

Goal:
- Decide whether the product is AI Native, AI-enhanced, or hybrid

Primary skills:
- `ai-native-vs-ai-enhanced`
- `agent-loop-model`

Primary deliverable:
- `AI Positioning Statement`

Go / No-Go:
- Go if the client agrees on what they are actually building
- No-Go if the team still lacks product-definition clarity

### Module 2: Agent Design

Goal:
- Define the agent's boundary, identity, HITL design, knowledge architecture, and trust progression

Primary skills:
- `agent-boundary-design`
- `system-prompt-engineering`
- `human-in-the-loop-design`
- `knowledge-injection-strategy`
- `trust-and-autonomy-tradeoff`
- `ai-native-ux-patterns`

Primary deliverables:
- `Agent Boundary Spec`
- `SOUL.md`
- `HITL Design Spec`
- `Knowledge Architecture Decision`
- `Trust-Autonomy Calibration Plan`

Go / No-Go:
- Go if the boundary is accepted and Zone C is explicit
- No-Go if the project is still trying to automate everything without a trust model

### Module 3: Data and Eval Foundation

Goal:
- Establish the data, labels, eval set, and production signals that make the system measurable

Primary skills:
- `data-foundation`
- `agent-eval-framework`

Primary deliverables:
- `Data Foundation Plan`
- `Eval Plan + Production Review Protocol`

Go / No-Go:
- Go if data can be exported, entities can be aligned, and eval is defined
- No-Go if data access or entity structure is too broken to validate the system

### Module 4: Technical Decision and Rollout Planning

Goal:
- Select the simplest viable stack and define the implementation plan

Primary skills:
- `tech-stack-selection`
- `multi-agent-orchestration` (if needed)
- `llm-capability-boundaries`

Primary deliverables:
- `Tech Stack Decision`
- `Multi-Agent Architecture Spec` (conditional)
- `Build / Rollout Plan`

Go / No-Go:
- Go if the stack supports the already-signed design decisions
- No-Go if technical enthusiasm is redefining product boundaries

### Module 5: Launch Readiness

Goal:
- Decide whether the system is ready for real users

Primary skills:
- `launch-risk-review`

Primary deliverables:
- `Launch Risk Review`
- `Rollout Strategy`

Go / No-Go:
- Go if eval, rollback, monitoring, and user readiness are all in place
- No-Go if any critical launch gate is still unproven

### Module 6: Iteration Flywheel

Goal:
- Convert production data into a stable weekly improvement process

Primary skills:
- `iteration-flywheel`

Primary deliverables:
- `Iteration Flywheel Design`
- `Weekly Production Review`

Go / No-Go:
- Go if there is clear ownership, cadence, and data capture
- No-Go if changes are still being made by intuition alone

---

## Dependency Rules

Hard dependencies:
- No `AI Positioning Statement` → do not begin agent design
- No `Agent Boundary Spec` → do not finalize SOUL, HITL, or autonomy design
- No `Data Foundation Plan` → do not claim formal eval readiness
- No `Launch Risk Review` pass → do not launch

Soft dependencies:
- `system-prompt-engineering`, `human-in-the-loop-design`, and `knowledge-injection-strategy` can be refined in parallel once the boundary is stable
- `tech-stack-selection` and implementation planning can progress together
- A demo can help alignment, but it never replaces data, eval, or launch review

---

## Recommended Order

```text
1. AI Positioning Statement
2. Agent Boundary Spec
3. SOUL.md
4. HITL Design Spec
5. Knowledge Architecture Decision
6. Trust-Autonomy Calibration Plan
7. Data Foundation Plan
8. Eval Plan + Production Review Protocol
9. Tech Stack Decision
10. Multi-Agent Architecture Spec (if needed)
11. Build / Rollout Plan
12. Launch Risk Review
13. Iteration Flywheel Design
```

Not every project needs every document. But every major decision should become a document the client can review, sign off on, and operate from.
