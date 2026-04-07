# Contributing to AI Native Builder Consultant Skills

**Maintainer:** [Medoc May](https://github.com/MedocMay/ai-native-builder-consultant-skills)


## The Bar

Every skill in this repo is grounded in production experience. The contribution bar reflects that:

- **Must come from production** — things you built, shipped, and observed breaking or working in front of real users. Research papers and second-hand accounts don't qualify.
- **Must answer a specific decision** — not "explain a concept" but "help someone decide X." Every skill is a decision framework, not a knowledge article.
- **Must have seen the failure modes** — if you've only seen the success case, the skill is incomplete. What goes wrong, under what conditions, and how do you catch it?

## Skill Format

Every skill must follow the standard structure:

```markdown
---
name: skill-name-in-kebab-case
description: One sentence, under 250 characters, front-loaded with trigger keywords.
---

# Skill Title

## When to Use
[Specific situations / trigger phrases that indicate this skill is needed]

## Ask These Questions First
[3–5 questions to gather context before giving recommendations]

## Decision Framework
[The actual framework — scored rubrics, decision trees, trade-off tables]

## Output Format
[What the skill produces — a spec, a scored table, a one-pager, a recommendation]

---
[Supporting content: worked examples, anti-patterns, related skills]
```

**Description field:** Under 250 characters. Claude uses this to decide whether to trigger the skill — longer descriptions get truncated. Front-load the trigger keywords (the words someone would say when they need this skill).

## What We Don't Accept

- Generic "best practices" with no production grounding
- Concept explainers with no decision output
- Skill descriptions over 250 characters
- Skills that duplicate existing coverage without meaningfully extending it
- Content that reveals confidential product data, user data, or internal metrics from any specific company

## Process

1. **Open an issue first** for new skills — describe the decision it answers and the production experience behind it. Wait for a thumbs-up before writing the full skill.
2. **PRs for fixes** (typos, factual corrections, format issues) — open directly, no issue needed.
3. **One skill per PR.** Keep changes focused.
4. **Run a self-check** before submitting:
   - Does it have all four required sections?
   - Is the description under 250 characters?
   - Does it say something that can't be found in a generic AI article?
   - Has someone actually built this and seen it fail?

## Format Validation

Check your YAML frontmatter is valid:

```bash
# Quick check — should print name and description cleanly
head -5 your-skill/SKILL.md
```

The `name` field must match the directory name exactly.

## License

By contributing, you agree your contribution is licensed under Apache 2.0 and that Medoc May retains the right to include your contribution in this repository. See [LICENSE](LICENSE).
