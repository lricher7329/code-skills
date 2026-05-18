# Project-Workflow Skill

## Purpose

Establish one consistent project lifecycle across every software repo: a single living plan
document, milestone-scoped branches, and plan-driven session re-entry. This skill governs
*how a project moves forward* — it is the counterpart to the implementation skills (`plan`,
`plan-lite`, `audit`, `debug`, `refactor`, `review`), which govern *how individual changes
are made*.

The pattern:

- Every repo has one living plan, `ROADMAP.md`, at its root — holding both the
  milestone/phase breakdown **and** the current development state.
- Re-entering a repo in a fresh session begins by reading `ROADMAP.md` and proposing the
  next step.
- Planning work edits `ROADMAP.md` in place, so the plan persists where it is found.
- Starting a milestone cuts a fresh `milestone/<slug>` branch.
- Completing a milestone updates the docs, then commits → pushes → PRs → merges.

## When to Use

- **At the start of any session** in a software repo — to orient via `ROADMAP.md` (see
  *Re-entry* below). Under `~/Code/`, this is triggered automatically by `~/Code/CLAUDE.md`.
- **When starting** a new milestone or phase of work.
- **When finishing** a milestone — to land the work.
- **When doing planning work** (including plan mode) — so the plan lands in `ROADMAP.md`.

## Invocation

```
"use project-workflow skill"
"where did I leave off"        -> re-entry
"start a milestone"            -> start a milestone/phase
"complete this milestone"      -> land a milestone
```

---

## The ROADMAP.md document

One file per repo, named `ROADMAP.md`, at the repo root. It is the durable home of the
plan and the single source of truth for current development state.

If a repo has no `ROADMAP.md`, offer to scaffold one from `ROADMAP-TEMPLATE.md` (in this
skill's directory) before doing milestone work.

`ROADMAP.md` has four sections:

1. **`## Current State`** — active milestone, current branch, the next concrete step, and
   the date last updated. This is what a fresh session reads first.
2. **`## Milestones`** — each milestone (`### M<n> — <name>`) with a status tag and a
   checklist of phases. Completed milestones also record their branch, PR number, and
   merge date.
3. **`## Detail Documents`** — links to secondary docs that hold depth the roadmap should
   not carry inline: dated plans under `.claude/plans/`, design docs under `docs/`, etc.
4. **`## Changelog`** — dated, one-line entries for milestone/phase status changes.

**Status legend:** `[ ]` planned · `[~]` in progress · `[x]` complete. Use it for both
milestone tags and phase checkboxes.

`ROADMAP.md` stays concise — it is an index and a status board. Push detail into linked
documents rather than letting the roadmap grow unbounded.

---

## Workflow modes

### 1. Re-entry (fresh session)

1. Read `ROADMAP.md`. If absent, offer to scaffold it from `ROADMAP-TEMPLATE.md` and stop.
2. Read `## Current State` — note the active milestone, expected branch, and next step.
3. Check git: `git branch --show-current` and `git status --short`.
4. Reconcile and **propose the next concrete step**:
   - If on the expected milestone branch with work in progress → propose continuing the
     next unchecked phase.
   - If on the default branch with no active milestone → propose starting the next
     planned milestone.
   - If git state and `## Current State` disagree → surface the discrepancy before
     proposing anything.

### 2. Planning mode

Planning work — including Claude Code plan mode — updates `ROADMAP.md` **in place**:

- Add or revise milestones and phases directly in `## Milestones`.
- Update `## Current State` to point at the new next step.
- Add a `## Changelog` entry.

Deep planning detail goes into linked secondary documents, not inline. The `plan` and
`plan-lite` skills continue to write their dated files to `.claude/plans/`; treat those as
**detail documents** and link them from `ROADMAP.md`'s `## Detail Documents` section.
`ROADMAP.md` is the durable home; dated plan files are attachments to it.

### 3. Starting a milestone or phase

1. Confirm the milestone is defined in `ROADMAP.md`. If not, define it first (see
   *Planning mode*).
2. Ensure the default branch is current:
   `git checkout <default> && git pull`.
3. Cut the branch: `git checkout -b milestone/<slug>`, where `<slug>` is a short,
   dash-separated name derived from the milestone (e.g. `milestone/auth-rework`).
4. In `ROADMAP.md`: tag the milestone `[~] in progress`, record `Branch: milestone/<slug>`
   under it, and update `## Current State` (active milestone, branch, next step, date).

**One branch per milestone.** The branch lives until the milestone *and* its documentation
updates are complete. Do not merge mid-milestone.

### 4. Completing a milestone

When the milestone's work **and** its documentation updates are done:

**Automatic — done without asking:**
1. Update `ROADMAP.md`: tag the milestone `[x] complete`, check off its phases, advance
   `## Current State` to the next milestone/step, add a `## Changelog` entry.
2. `git add` the changed files.
3. Draft a commit message (summary + body describing the milestone).

**Pause for confirmation here.** Present the staged diff summary and the draft commit
message. Then, only on approval, run each remote-touching step:

4. `git commit`
5. `git push -u origin milestone/<slug>`
6. `gh pr create` — title and body from the milestone and its phases.
7. Merge the PR.
8. `git checkout <default> && git pull` — return to the default branch, updated.
9. Record the PR number and merge date on the milestone in `ROADMAP.md`.

---

## Git autonomy

| Phase | Behavior |
|-------|----------|
| Update `ROADMAP.md`, `git add`, draft commit message | Automatic |
| `git commit`, `git push`, `gh pr create`, merge | **Pause for confirmation first** |

Confirmation is requested once, before the remote sequence; on approval the sequence runs
through. Never push, open a PR, or merge without that confirmation.

## Relationship to other skills

- **`plan` / `plan-lite`** — produce dated implementation plans in `.claude/plans/`. Those
  become *detail documents* linked from `ROADMAP.md`; this skill owns the durable roadmap.
- **`audit` / `debug` / `refactor` / `review`** — operate *within* a milestone branch on
  the current change; they do not manage the roadmap or the git lifecycle.

## Definition of Done

- `ROADMAP.md` exists, is current, and `## Current State` matches the actual git state.
- Each milestone's work was done on its own `milestone/<slug>` branch.
- Completed milestones are tagged `[x]` with PR number and merge date recorded.
- No milestone branch was merged before its documentation updates were complete.

## Anti-Patterns (Never Do)

- Do milestone work directly on the default branch.
- Let `ROADMAP.md` and the active git branch drift out of sync without flagging it.
- Push, open a PR, or merge without the completion-step confirmation.
- Let `ROADMAP.md` accumulate detail that belongs in a linked document.
- Start a second milestone branch while another milestone is still open.
