# Plan-Lite Skill

## Purpose

A lightweight, focused prompting approach for small or scoped tasks. Delivers clear, concrete, and reliable outputs without losing rigor or determinism.

## When to Use

Use this skill when the task is:
- One feature or module
- A focused code change
- A small architectural decision
- A config/schema update
- A targeted improvement within a larger plan
- A bug fix or refactor

## When NOT to Use

Use the full `plan` skill instead if:
- The task spans multiple subsystems
- A full architecture or roadmap is requested
- Governance or evaluation frameworks are involved
- The user asks for a full implementation plan

## Invocation

```
"use plan-lite skill"
"quick plan"
"focused change for <feature>"
```

---

## Core Rules (Always Follow)

### 1. Read First
- If the user provides code, markdown, YAML, or plans: read them before responding.
- Explicitly respect constraints from provided artifacts.

### 2. Be Concrete
- Map every recommendation to a file, function, schema field, or config key.
- Avoid abstract or generic advice.

### 3. Preserve Determinism
- If randomness exists, require explicit seed handling.
- Never introduce hidden or global randomness.

### 4. Don't Break Existing Behavior
- Prefer additive changes.
- Use defaults and feature flags when behavior changes.

### 5. Small Changes, Clear Steps
- Favor minimal diffs and PR-sized increments.
- Avoid rewrites unless explicitly requested.

---

## Standard Response Structure

Use this structure unless the user explicitly asks otherwise:

```markdown
## 1. Context & Constraints
- What was read
- What constraints matter

## 2. Proposed Change
- What will change and why

## 3. Exact Implementation
- File(s) to edit (exact paths)
- Code snippets or diffs
- Config/schema changes (if any)
- Integration point (where to call it)

## 4. Tests / Validation
- Tests to add (at least one smoke test + regression/determinism where relevant)
- How to run them
- How to verify correctness

## 5. Risks / Notes
- Edge cases or follow-ups (brief)
```

---

## Output

Save the plan to `.claude/plans/plan-lite-YYYY-MM-DD.md` in the project root, where `YYYY-MM-DD` is the current date.

If multiple plans are created on the same day, append a sequence number: `plan-lite-YYYY-MM-DD-2.md`.

---

## Definition of Done

Output is complete when:
- Code compiles/runs with minimal edits
- Integration point is explicit (where to call it)
- Tests included or clear validation steps provided
- Behavior unchanged when feature flag is off (if applicable)

---

## Quality Bar (Self-Check)

Before responding, confirm:

- [ ] I read the provided artifacts
- [ ] I referenced the actual code/schema
- [ ] I proposed a minimal, concrete change
- [ ] I didn't invent abstractions or rewrite unnecessarily
- [ ] I explained how to test or validate the change
- [ ] Integration point is clear

---

## Anti-Patterns (Never Do)

- Propose changes without reading provided files
- Give abstract advice without file/function references
- Introduce unnecessary abstractions
- Rewrite when a small edit suffices
- Skip validation/testing guidance
