# Debug Skill

## Purpose

Investigate bugs, trace root causes, and produce minimal, targeted fixes. This skill focuses on **understanding why something is broken** before proposing changes.

## When to Use

Use this skill when:
- Something is broken or behaving unexpectedly
- A test is failing and the cause is unclear
- Behavior differs between environments
- A regression has been introduced
- Performance has degraded unexpectedly

## When NOT to Use

Use other skills instead if:
- You're adding new functionality → `plan` or `plan-lite`
- You're improving structure without fixing a bug → `refactor`
- You're reviewing code quality broadly → `audit`

## Invocation

```
"use debug skill"
"debug this issue"
"investigate <problem>"
"why is <X> broken"
```

---

## Core Principles

### 1. Reproduce First
- Confirm the issue exists and is reproducible
- Identify the minimal reproduction case
- Document exact steps, inputs, and environment

### 2. Understand Before Fixing
- Trace the execution path
- Identify where expected behavior diverges from actual
- Find the root cause, not just the symptom

### 3. Minimal Fix
- Change only what's necessary to fix the issue
- Avoid "while I'm here" improvements
- Preserve existing behavior for unaffected paths

### 4. Validate the Fix
- Confirm the fix resolves the original issue
- Check for regressions in related functionality
- Add a test that would have caught this bug

---

## Workflow

### Step 1: Reproduction
- Confirm the issue is reproducible
- Identify minimal reproduction steps
- Document environment, inputs, expected vs actual output

### Step 2: Hypothesis
- Form hypotheses about possible causes
- Rank by likelihood
- Identify what evidence would confirm/refute each

### Step 3: Investigation
- Trace execution from entry point to failure
- Identify the exact location where behavior diverges
- Look for: wrong inputs, state corruption, race conditions, incorrect logic, missing error handling

### Step 4: Root Cause
- Distinguish symptom from cause
- Trace back: why did the immediate cause happen?
- Stop when you reach the actionable fix point

### Step 5: Fix
- Propose the minimal change that addresses root cause
- Show exact file(s) and code changes
- Explain why this fixes the issue

### Step 6: Validation
- How to verify the fix works
- What regression tests to run
- New test to prevent recurrence

---

## Required Output Sections

```markdown
## 1. Issue Summary
- What is broken
- How to reproduce
- Expected vs actual behavior

## 2. Investigation Trace
- Entry point examined
- Execution path followed
- Where behavior diverges from expected

## 3. Root Cause
- The underlying issue (not just the symptom)
- Why this causes the observed behavior
- How long this has likely existed (if determinable)

## 4. Fix
- File(s) to change
- Exact code changes
- Why this addresses the root cause

## 5. Validation
- How to verify the fix
- Regression risks
- Test to add

## 6. Prevention
- How to prevent similar issues
- Related code that might have the same problem
```

---

## Output

Save the debug report to `.claude/debug/debug-YYYY-MM-DD.md` in the project root, where `YYYY-MM-DD` is the current date.

If multiple debug sessions are run on the same day, append a sequence number: `debug-YYYY-MM-DD-2.md`.

---

## Investigation Techniques

### Trace Execution
- Follow data from input to output
- Log intermediate values at key points
- Compare working vs broken cases

### Bisect
- Find when the issue was introduced
- Compare before/after states
- Identify the change that caused regression

### Isolate
- Remove variables until minimal reproduction
- Test components independently
- Check if issue is in code, config, or environment

### Compare
- Working vs broken environment
- Expected vs actual state at each step
- Similar code that works vs code that doesn't

---

## Self-Check

Before proposing a fix, confirm:

- [ ] Issue is reproducible with documented steps
- [ ] Root cause is identified (not just symptom)
- [ ] Fix is minimal and targeted
- [ ] Fix doesn't introduce new issues
- [ ] Validation steps are clear
- [ ] Test would catch this bug in the future

---

## Anti-Patterns (Never Do)

- Propose a fix without understanding root cause
- Make unrelated changes "while fixing"
- Skip reproduction ("I think I know what's wrong")
- Fix the symptom instead of the cause
- Assume the first hypothesis is correct
