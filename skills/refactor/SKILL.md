# Refactor Skill

## Purpose

Improve code structure, clarity, or performance **without changing behavior**. The defining constraint of refactoring is that all existing functionality must work exactly as before.

## When to Use

Use this skill when:
- Improving code organization or readability
- Reducing duplication
- Renaming for clarity
- Extracting functions/classes/modules
- Improving performance without changing API
- Preparing code for future changes (making it easier to extend)

## When NOT to Use

Use other skills instead if:
- You're fixing a bug → `debug`
- You're adding new functionality → `plan` or `plan-lite`
- Behavior will change in any way → `plan-lite`

## Invocation

```
"use refactor skill"
"refactor <component>"
"clean up <code>"
"improve structure of <module>"
```

---

## Core Principle: Behavior Preservation

**The refactored code must produce identical outputs for identical inputs.**

This means:
- All existing tests must pass unchanged
- No new parameters, return values, or side effects
- No changes to public APIs
- Error cases behave the same

If behavior needs to change, use `plan-lite` instead.

---

## Refactoring Types

### Extract
- Extract function from inline code
- Extract class from large class
- Extract module from tangled file
- Extract constant from magic value

### Rename
- Rename variable/function/class for clarity
- Rename file to match contents
- Rename to follow conventions

### Move
- Move function to better location
- Move class to appropriate module
- Colocate related code

### Simplify
- Remove dead code
- Simplify complex conditionals
- Replace nested logic with early returns
- Remove unnecessary abstractions

### Consolidate
- Merge duplicate functions
- Unify inconsistent patterns
- Create shared utilities from repeated code

---

## Workflow

### Step 1: Identify Scope
- What code will be refactored
- What the improvement goal is
- What tests cover this code

### Step 2: Verify Test Coverage
- Identify existing tests
- Note any gaps (flag but don't fix during refactor)
- Ensure tests are green before starting

### Step 3: Plan Changes
- List specific refactoring operations
- Order from safest to riskiest
- Keep each change small and atomic

### Step 4: Execute
- Make one refactoring operation at a time
- Run tests after each change
- If tests fail, the refactoring introduced a behavior change (fix or reconsider)

### Step 5: Verify
- All tests pass
- Manual spot-check of key behaviors
- Review the diff for unintended changes

---

## Required Output Sections

```markdown
## 1. Refactoring Goal
- What improvement is being made
- Why this improves the code

## 2. Scope
- Files affected
- Functions/classes changed
- What is NOT being changed

## 3. Test Coverage Check
- Existing tests that cover this code
- Coverage gaps (noted, not fixed)

## 4. Refactoring Steps
For each step:
- What operation (extract, rename, move, etc.)
- Before snippet
- After snippet
- Why this preserves behavior

## 5. Verification
- Tests to run
- Manual checks recommended
- How to confirm behavior unchanged
```

---

## Self-Check

Before completing, confirm:

- [ ] All existing tests pass
- [ ] No new functionality added
- [ ] No behavior changes (same outputs for same inputs)
- [ ] Public APIs unchanged
- [ ] Each change is independently verifiable
- [ ] Changes are reversible if needed

---

## Anti-Patterns (Never Do)

- Change behavior during refactoring
- Add features "while you're in there"
- Refactor untested code without acknowledging risk
- Make multiple unrelated changes in one step
- Skip running tests between changes
- Refactor and fix bugs simultaneously

---

## Relationship to Other Skills

| Scenario | Skill |
|----------|-------|
| Code is broken | `debug` first, then `refactor` |
| Need new behavior | `plan-lite` (not refactor) |
| Improving structure only | `refactor` |
| Large architectural change | `plan` |
