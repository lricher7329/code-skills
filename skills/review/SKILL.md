# Review Skill

## Purpose

Review a specific change (PR, diff, or proposed code) against quality principles. Produces structured feedback with approve/request-changes decision and concrete, actionable comments.

## When to Use

Use this skill when:
- Reviewing a pull request
- Evaluating proposed code changes before merge
- Assessing a diff for quality and correctness
- Checking if a change follows project standards

## When NOT to Use

Use other skills instead if:
- Reviewing entire codebase quality → `audit`
- Planning new implementation → `plan` or `plan-lite`
- Investigating a bug → `debug`

## Invocation

```
"use review skill"
"review this PR"
"review this diff"
"code review <change>"
```

---

## Review Dimensions

Evaluate every change against these criteria:

### 1. Correctness
- Does the change do what it claims?
- Are edge cases handled?
- Are error conditions managed properly?

### 2. Testing
- Are new behaviors tested?
- Are regression tests included?
- Is test coverage appropriate for the risk?

### 3. Backward Compatibility
- Does this break existing functionality?
- Are API changes backward-compatible?
- Is migration path clear if breaking?

### 4. Determinism
- Is randomness properly seeded?
- Are there hidden sources of nondeterminism?
- Will results be reproducible?

### 5. Code Quality
- Is the change minimal (no unnecessary changes)?
- Is naming clear?
- Is complexity appropriate?

### 6. Integration
- Is the integration point clear?
- Are dependencies appropriate?
- Does this fit the existing architecture?

---

## Workflow

### Step 1: Understand Intent
- Read the PR description / commit message
- Understand what problem this solves
- Identify the scope of changes

### Step 2: Review Changes
- Read each modified file
- Trace how changes connect
- Check for completeness

### Step 3: Evaluate
- Assess against each review dimension
- Note issues with specific file:line references
- Categorize issues by severity

### Step 4: Decision
- Approve: Ready to merge as-is
- Request Changes: Issues must be addressed
- Comment: Minor suggestions, can merge without

---

## Required Output Sections

```markdown
## 1. Change Summary
- What this change does
- Files modified
- Scope assessment (small/medium/large)

## 2. Review Findings

### Correctness
- [PASS/ISSUE] Finding with file:line reference
- ...

### Testing
- [PASS/ISSUE] Finding with file:line reference
- ...

### Backward Compatibility
- [PASS/ISSUE] Finding with file:line reference
- ...

### Determinism
- [PASS/ISSUE/N/A] Finding with file:line reference
- ...

### Code Quality
- [PASS/ISSUE/SUGGESTION] Finding with file:line reference
- ...

### Integration
- [PASS/ISSUE] Finding with file:line reference
- ...

## 3. Issues Summary

### Blocking (must fix)
- Issue with file:line and recommended fix

### Non-blocking (should fix)
- Issue with file:line and recommended fix

### Suggestions (optional)
- Improvement idea with file:line

## 4. Decision
- **APPROVE** / **REQUEST CHANGES** / **COMMENT**
- Rationale for decision
```

---

## Output

Save the review to `.claude/reviews/review-YYYY-MM-DD.md` in the project root, where `YYYY-MM-DD` is the current date.

If multiple reviews are run on the same day, append a sequence number: `review-YYYY-MM-DD-2.md`.

---

## Issue Severity Guide

**Blocking (must fix before merge):**
- Incorrect behavior
- Missing tests for new functionality
- Breaking changes without migration
- Security issues
- Data corruption risks

**Non-blocking (should fix, can merge):**
- Missing edge case handling
- Incomplete error messages
- Minor test gaps
- Unclear naming

**Suggestions (optional improvements):**
- Style improvements
- Alternative approaches
- Future considerations

---

## Self-Check

Before finalizing review:

- [ ] I understood the intent of the change
- [ ] I read all modified files
- [ ] I checked all six review dimensions
- [ ] Every issue references a specific file:line
- [ ] Every issue has a recommended fix
- [ ] Decision matches severity of findings

---

## Anti-Patterns (Never Do)

- Review without understanding intent
- Vague feedback ("this looks wrong")
- Stylistic nitpicks on existing code (not in diff)
- Block on suggestions/preferences
- Miss obvious correctness issues while focusing on style
- Approve without actually reviewing

---

## Feedback Style

**Good feedback:**
```
[ISSUE] src/loader.py:45 - Missing null check before accessing `data["field"]`.
If `data` is empty dict, this raises KeyError.
Fix: Add `if "field" in data:` guard or use `data.get("field")`.
```

**Bad feedback:**
```
This might break.
```
