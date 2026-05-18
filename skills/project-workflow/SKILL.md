# Project Workflow

## Purpose

Establish one consistent project lifecycle across every software repo — and identically
across Codex and Claude Code. Each repo has a single living plan, work happens on
milestone-scoped branches, and every session re-entry starts from the plan. This skill
governs *how a project moves forward*; it is the counterpart to implementation skills
(planning, audit, debug, refactor, review), which govern *how individual changes are made*.

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
  *Re-entry* below).
- **When starting** a new milestone or phase of work.
- **When finishing** a milestone — to land the work.
- **When doing planning work** (including plan mode) — so the plan lands in `ROADMAP.md`.

Triggering is automatic: in Claude Code via the entry in `~/Code/CLAUDE.md`; in Codex via
this skill's `description`. It may also be invoked explicitly.

## Invocation

```
"use project-workflow skill"
"where did I leave off"        -> re-entry
"start a milestone"            -> start a milestone/phase
"complete this milestone"      -> land a milestone
```

---

## The Primary Plan: ROADMAP.md

One file per repo, named `ROADMAP.md`, at the repo root. It is the durable home of the
plan and the operational source of truth for current development state.

**Existing-convention clause:** if a repo already standardizes on a differently named
primary plan (e.g. `docs/project-plan.md`), respect it — treat that file as the `ROADMAP.md`
for all purposes rather than forcing a rename. New repos, and repos with no primary plan,
use `ROADMAP.md` at the root. If a repo has none, offer to scaffold one from
`ROADMAP-TEMPLATE.md` (in this skill's directory) before doing milestone work.

`ROADMAP.md` has four sections:

1. **`## Current State`** — active milestone, current branch, the next concrete step, and
   the date last updated. This is what a fresh session reads first.
2. **`## Milestones`** — each milestone (`### M<n> — <name>`) with a status tag and a
   checklist of phases. Completed milestones also record their branch, PR number, and
   merge date.
3. **`## Detail Documents`** — links to secondary docs that hold depth the roadmap should
   not carry inline: detailed specs, design notes, dated planning docs, archived history.
4. **`## Changelog`** — dated, one-line entries for milestone/phase status changes.

**Status legend:** `[ ]` planned · `[~]` in progress · `[x]` complete. Use it for both
milestone tags and phase checkboxes.

`ROADMAP.md` stays concise — it is an index and a status board for current state,
sequencing, active milestones, and known issues. Push detail into linked documents rather
than letting the roadmap grow unbounded. Do not treat archived or superseded plans as
current unless `ROADMAP.md` explicitly points to them for context.

---

## Workflow modes

### 1. Re-entry (fresh session)

1. Locate and read `ROADMAP.md`. If absent, offer to scaffold it from `ROADMAP-TEMPLATE.md`
   and stop.
2. Read `## Current State` — note the active milestone, expected branch, and next step.
   Read enough of `## Milestones` to see completed phases and active risks.
3. Check git: `git branch --show-current` and `git status --short`.
4. Inspect only the code/docs needed to verify the plan still matches reality.
5. Reconcile and **propose the next concrete step**:
   - If on the expected milestone branch with work in progress → propose continuing the
     next unchecked phase.
   - If on the default branch with no active milestone → propose starting the next
     planned milestone.
   - If `ROADMAP.md` and the code/git state disagree → say so explicitly and recommend
     resolving the drift (or updating the plan) before proposing new work.

### 2. Planning mode

Planning work — including plan mode — updates `ROADMAP.md` **in place**:

- Add or revise milestones and phases directly in `## Milestones` whenever planning
  decisions change scope, status, sequencing, risks, or the next step.
- Update `## Current State` to point at the new next step.
- Add a `## Changelog` entry.
- Add or update linked secondary documents when detail is too large for the hub plan.
- End planning work with an explicit proposed next development step.

`ROADMAP.md` is the durable home where decisions persist; dated or detailed planning
artifacts are *detail documents* linked from `## Detail Documents`.

### 3. Starting a milestone or phase

1. Confirm the milestone is defined in `ROADMAP.md`. If not, define it first (see
   *Planning mode*).
2. Inspect git state first. If the working tree has unrelated edits, preserve them and
   choose the least disruptive branch strategy — never overwrite or revert unrelated work.
3. Ensure the default branch is current: `git checkout <default> && git pull`.
4. Cut the branch: `git checkout -b milestone/<slug>`, where `<slug>` is a short,
   dash-separated name derived from the milestone (e.g. `milestone/auth-rework`,
   `milestone/mobile-readiness`).
5. In `ROADMAP.md`: tag the milestone `[~] in progress`, record `Branch: milestone/<slug>`
   under it, and update `## Current State` (active milestone, branch, next step, date).

**One branch per milestone.** The branch lives until the milestone *and* its documentation
updates are complete. Do not merge mid-milestone.

### 4. During implementation

Keep code and planning state aligned as work lands:

1. Implement the milestone using the repo's normal engineering patterns.
2. Update tests and validation proportional to the risk of the change.
3. Update `ROADMAP.md` as work lands — check off completed phases, record dates, move
   resolved issues, add newly discovered issues or risks, update the next step.
4. Update linked secondary docs when specs they hold have changed.
5. Prefer small, coherent batches reviewable as a milestone unit.

### 5. Completing a milestone

When the milestone's work **and** its documentation updates are done:

**Automatic — done without asking:**
1. Run relevant validation (tests, linters, type checks).
2. Update `ROADMAP.md`: tag the milestone `[x] complete`, check off its phases, advance
   `## Current State` to the next milestone/step, add a `## Changelog` entry.
3. `git add` the code and documentation together.
4. Draft a milestone-oriented commit message (summary + body).

**Pause for confirmation here.** Present the staged diff summary (`git diff --cached
--stat`) and the draft commit message. Then, only on approval, run each remote-touching
step:

5. `git commit`
6. `git push -u origin milestone/<slug>`
7. `gh pr create` — title and body from the milestone and its phases.
8. Monitor checks.
9. Merge the PR when checks and review conditions are satisfied.
10. `git checkout <default> && git pull` — return to the default branch, updated.
11. Record the PR number and merge date on the milestone in `ROADMAP.md`.

If any step needs credentials, external artifacts, or an explicit product decision, leave
it as an open item in `ROADMAP.md` and state the blocker clearly.

---

## Git autonomy

| Phase | Behavior |
|-------|----------|
| Run validation, update `ROADMAP.md`, `git add`, draft commit message | Automatic |
| `git commit`, `git push`, `gh pr create`, merge | **Pause for confirmation first** |

Confirmation is requested once, before the remote sequence; on approval the sequence runs
through (including monitoring checks and merging). Never push, open a PR, or merge without
that confirmation.

## Response Expectations

When asked to review the current development plan, lead with:

1. Current milestone/status from `ROADMAP.md`.
2. Any mismatch between the plan and the repository state.
3. The recommended next step.
4. Whether that next step is planning, implementation, validation, or release/PR work.

Keep updates concise, but always mention plan/documentation changes when they affect
project state.

## Relationship to other skills

- **Planning skills** (e.g. `plan`, `plan-lite`) produce detailed, often dated plans.
  Those become *detail documents* linked from `ROADMAP.md`; this skill owns the durable
  roadmap and the git lifecycle.
- **`audit` / `debug` / `refactor` / `review`** operate *within* a milestone branch on the
  current change; they do not manage the roadmap or the git lifecycle.

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
- Overwrite or revert unrelated working-tree changes when cutting a branch.
