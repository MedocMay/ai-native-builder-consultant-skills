# Terminology

This glossary standardizes the main consulting and delivery terms used in this repository.

---

## Core Terms

**AI Native**  
A product whose core value depends on AI behavior, not just AI augmentation of an existing workflow.

**AI-enhanced**  
A product where AI improves an existing workflow, but the product still fundamentally exists without the model.

**Agent**  
A system that perceives state, reasons over it, takes action, and observes outcomes over time.

**SOUL**  
The production system-prompt constitution for an agent. It defines role, objective, constraints, and behavior under pressure.

**HITL**  
Human-in-the-loop. The confirmation and intervention layer between agent behavior and consequential action.

**Zone A / Zone B / Zone C**  
The trust and action boundary model.
- Zone A: autonomous execution
- Zone B: human confirmation required
- Zone C: forced escalation or refusal

**Vibe Check**  
A small manual test set used to catch obvious regressions quickly.

**Golden Dataset**  
A curated eval set used for more stable regression testing and release decisions.

**Production Signals**  
Real-world behavioral metrics from live usage, such as override rate, trigger rate, error rate, and business outcomes.

**Iteration Flywheel**  
The ongoing loop of production data, review, root-cause analysis, validated change, deployment, and monitoring.

---

## Standard Deliverable Names

Use these names consistently across English and Chinese materials.

| English | Chinese |
|--------|---------|
| AI Positioning Statement | AI 产品定位声明 |
| Agent Boundary Spec | Agent Boundary Spec |
| SOUL.md | SOUL.md |
| HITL Design Spec | HITL Design Spec |
| Knowledge Architecture Decision | 知识架构决策 |
| Trust-Autonomy Calibration Plan | 信任-自治校准计划 |
| Data Foundation Plan | 数据基础计划 |
| Eval Plan + Production Review Protocol | Eval 计划 + 每周 Production Review 议程 |
| Tech Stack Decision | 技术栈决策 |
| Multi-Agent Architecture Spec | 多 Agent 架构 Spec |
| Build / Rollout Plan | 实施 / 推进计划 |
| Launch Risk Review | Launch Risk Review |
| Iteration Flywheel Design | 迭代飞轮设计 |
| Weekly Production Review | 每周 Production Review |

---

## Translation Guidance

These terms should usually remain in English:
- Agent
- SOUL
- HITL
- Zone A / B / C
- Eval
- Launch Risk Review
- Production Review

These terms can be translated naturally into Chinese:
- positioning
- boundary
- knowledge architecture
- rollout
- flywheel

When in doubt:
- keep the deliverable title stable
- explain it in the surrounding language
- avoid creating multiple Chinese names for the same artifact
