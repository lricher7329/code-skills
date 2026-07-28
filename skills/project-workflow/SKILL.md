---
name: project-workflow
description: Guide work across Lawrence's software repositories using one consistent living plan (ROADMAP.md), a repo-tier git strategy (reviewed / gated / direct), local verification as the merge gate, plan-driven session re-entry, and a tier-appropriate landing flow. Use when entering or re-entering a repo, proposing next development steps, working in planning mode, starting a milestone/phase, updating project documentation, or completing implementation work that should be verified, committed, pushed, and landed on the default branch. This skill is kept identical across Codex and Claude Code.
---

# Project Workflow

## Purpose

Establish one consistent project lifecycle across every software repo — and identically
across Codex and Claude Code. Each repo has a single living plan, work is verified locally
before it lands, and every session re-entry starts from the plan. This skill governs *how a
project moves forward*; it is the counterpart to implementation skills (planning, audit,
debug, refactor, review), which govern *how individual changes are made*.

The pattern:

- Every repo has one living plan, `ROADMAP.md`, at its root — holding both the
  milestone/phase breakdown **and** the current development state.
- Every repo has a **tier** (A/B/C) that determines its git and CI strategy. Ceremony is
  proportional to blast radius, not applied uniformly.
- Re-entering a repo in a fresh session begins by reading `ROADMAP.md` and proposing the
  next step, or presenting a decision when the roadmap contains multiple viable next
  steps.
- Planning work edits `ROADMAP.md` in place, so the plan persists where it is found.
- Substantial work gets a branch; routine work does not, outside Tier A.
- **Local verification is the gate that lets work land.** Remote CI serves a different
  purpose per tier — merge gate (A), release gate (B), or packaging only (C).
- Completing a milestone updates the docs, then verifies → commits → lands by the tier's
  route, and ends by presenting the next roadmap step or decision.

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
"land it" / "ship it"          -> land a milestone
```

---

## Repository Tiers

Every repo sits in one of three tiers. The tier determines branching, what gates a merge,
and what remote CI is *for*. Read the tier before proposing any git action.

| | **Tier A — Reviewed** | **Tier B — Gated** | **Tier C — Direct** |
|---|---|---|---|
| **Applies to** | Multi-developer, or regulated code under formal change control | Solo, but deployed / shared-contract / handles participant data | Solo, self-contained: analysis, tooling, extensions, docs sites |
| **Branching** | Always `milestone/<slug>` | Branch for substantial work only | Default branch; branch only for risky or exploratory work |
| **Merge gate** | Remote CI + review, via PR | **Local verification** | Local checks proportional to risk |
| **Remote CI is** | The merge gate | A **release/deploy gate** — dispatch or tag | Packaging/publishing only |
| **PR** | Required, reviewed | Optional; if used, `--fill` + auto-merge, no review wait | Not used |
| **Push cadence** | Per milestone | Meaningful milestones, **daily minimum** | Daily minimum, for backup |

**Declaring a tier.** The tier is recorded in `ROADMAP.md` under `## Current State` as
`**Tier:** B — Gated`. That declaration wins. If a repo has no tier recorded, infer a
default from its character, state the inference, and offer to record it:

- **Tier A** — `catalyst/*`.
- **Tier B** — anything deployed, publishing a contract a sibling repo consumes, or
  touching participant/clinical data: `celeris/*`, `metricis`, `janus*`, `form-platform`,
  `autonometrics`, `platform-standards`.
- **Tier C** — everything else: `research/*`, `experiments/*`, `tools/*`, `extensions/*`,
  `mcp/*`, `web/*`.

When in doubt between two tiers, take the higher one and say why.

**Tier is a property of the repo, not the change.** A risky change in a Tier C repo gets a
branch and more testing; it does not get a PR. Escalating a single change to a higher tier's
ceremony is churn — raise the repo's tier if that need is recurring.

---

## Local Verification

In Tier B and C, local verification replaces the PR as the thing that makes work safe to
land. It is therefore not optional and not a matter of memory — it must be a command.

1. **Find the repo's check command** before landing anything. In order of preference: a
   `make ci` target (present in `celeris/core`, `celeris/match`, `celeris/flow`), a
   `.pre-commit-config.yaml`, or the lint/typecheck/test commands the repo's CI workflow
   runs. If the repo has none, say so and offer to add a `make ci` target — a repo that
   cannot be verified locally cannot be Tier B.
2. **Run it before every landing**, not just at milestone end. Report failures with output;
   never land on a red check.
3. **Confirm nothing is uncommitted:** `git status --porcelain` must be empty before the
   work is called done. A passing local suite over a dirty tree proves nothing about what
   you are about to push.

**What local checks cannot catch.** Your working copy is macOS (case-insensitive) with a
warm environment. A forgotten `git add`, a dependency missing from the lockfile, and a
case-mismatched import all pass locally and fail on a clean Linux checkout. Remote CI is
the only thing that catches this class, so it stays in the loop at every tier:

- **Tier B** — the release/deploy gate is a clean checkout by construction. Run it before
  any release, deploy, pilot, or demo, and after any dependency change.
- **Tier C** — where a packaging or publishing workflow exists, a red build is a real
  signal. Treat it as a follow-up task rather than a blocker on unrelated work.

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

1. **`## Current State`** — repo tier, active milestone, current branch, the next concrete
   step, and the date last updated. This is what a fresh session reads first.
2. **`## Milestones`** — each milestone (`### M<n> — <name>`) with a status tag and a
   checklist of phases. Completed milestones also record their branch, PR number where one
   exists, and merge date.
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

## Next-Step Handoffs

At both lifecycle boundaries — fresh-session re-entry and after landing a milestone or
development phase — end by presenting what `ROADMAP.md` says should happen next.

1. Re-read `ROADMAP.md` after any merge and final default-branch pull, so the handoff uses
   the merged state.
2. Derive the next action from `## Current State`, then confirm it against the next
   unchecked milestone or phase in `## Milestones`.
3. If there is exactly one viable next step, present it as the recommended next step and
   identify whether it is planning, implementation, validation, or release work.
4. If more than one viable next step is present — such as parallel planned milestones,
   competing unchecked phases, an explicit open decision, or plan/git ambiguity — present
   the options as a decision for the user. Each option should name the roadmap item and
   the immediate action it would start. Do not silently choose between viable options
   unless `ROADMAP.md` explicitly prioritizes one.
5. If no concrete next step can be derived, say that the roadmap needs an update and
   recommend making `## Current State` explicit before starting more work.

---

## Workflow modes

### 1. Re-entry (fresh session)

1. Locate and read `ROADMAP.md`. If absent, offer to scaffold it from `ROADMAP-TEMPLATE.md`
   and stop.
2. Read `## Current State` — note the **tier**, active milestone, expected branch, and next
   step. Read enough of `## Milestones` to see completed phases and active risks.
3. Check git: `git branch --show-current` and `git status --short`.
4. Inspect only the code/docs needed to verify the plan still matches reality.
5. Reconcile and apply **Next-Step Handoffs**:
   - If on the expected branch with work in progress → propose continuing the next
     unchecked phase.
   - If on the default branch with no active milestone → propose starting the next
     planned milestone.
   - If multiple viable roadmap steps are available → present the options as a decision
     and pause for the user's choice before starting new work.
   - If `ROADMAP.md` and the code/git state disagree → say so explicitly and recommend
     resolving the drift (or updating the plan) before proposing new work.
6. If there are unpushed commits older than a day, say so — the daily-push floor exists so
   that local-only work is never the sole copy.

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
3. Ensure the default branch is current: `git switch <default> && git pull --ff-only --prune`.
4. **Decide whether to branch**, by tier:
   - **Tier A** — always branch.
   - **Tier B** — branch when the work is substantial: spans more than one session, touches
     a schema/contract/migration, or would leave the default branch broken partway through.
     Otherwise work directly on the default branch in small increments.
   - **Tier C** — work on the default branch unless the change is exploratory or likely to
     be abandoned.
5. When branching: `git switch -c milestone/<slug>`, where `<slug>` is a short,
   dash-separated name derived from the milestone (e.g. `milestone/auth-rework`).
6. In `ROADMAP.md`: tag the milestone `[~] in progress`, record the branch under it (or
   `Branch: <default>` when working directly), and update `## Current State` (active
   milestone, branch, next step, date).

**A branch should be short-lived.** It exists to keep the default branch working, not to
batch up a release. If a branch has been open long enough that its merge will be hard to
reason about, land the safe parts and re-cut — a big-bang merge is worse solo than on a
team, because no reviewer is coming to justify the batch.

### 4. During implementation

Keep code and planning state aligned as work lands:

1. Implement the milestone using the repo's normal engineering patterns.
2. **Commit in small, coherent increments** — each one a self-contained, working change with
   a real message. This is what makes a branch optional in Tiers B and C.
3. Update tests and validation proportional to the risk of the change.
4. Update `ROADMAP.md` as work lands — check off completed phases, record dates, move
   resolved issues, add newly discovered issues or risks, update the next step.
5. Update linked secondary docs when specs they hold have changed.
6. **Push at meaningful milestones, and at least daily.** Pushing is backup, not a
   ceremony — it does not require the completion-step confirmation on an already-pushed
   branch, and it does not wait for the milestone to be finished.

### 5. Landing a milestone

When the milestone's work **and** its documentation updates are done:

**Automatic — done without asking:**
1. Run **Local Verification** — the repo's check command, plus a clean
   `git status --porcelain`. Report failures with output; do not proceed on red.
2. Update `ROADMAP.md`: tag the milestone `[x] complete`, check off its phases, advance
   `## Current State` to the next milestone/step when unambiguous, or record that a
   next-step decision is needed when multiple roadmap paths are viable; add a
   `## Changelog` entry.
3. `git add` the code and documentation together.
4. Draft a milestone-oriented commit message (summary + body).

**Cross-repo contract check (before calling it done).** If the milestone changed any
interface shared with a sibling repo or a downstream consumer — a data contract, schema,
attribute catalog/snapshot, API shape, or a version-gated allowlist — the work is **not
complete until both sides are reconciled**. A change that bumps a contract version on one
side silently breaks every consumer that gates on the old version. So: (a) propagate the
change to the sibling repo, or flag it explicitly to that repo's owner with the exact
edit owed; and (b) register/accept the new version in every consumer that validates it.
Check the sibling repo directly when you can (read its contract/snapshot/version files)
rather than assuming — the two sides drift precisely because each is updated alone. Record
the reconciliation (the owed registration, the version, the verified value parity) in the
commit and the handoff doc.

**Pause for confirmation here.** Present the staged diff summary (`git diff --cached
--stat`) and the draft commit message. Then, only on approval, run the tier's landing
sequence:

**Tier A — Reviewed**
5. `git commit`
6. `git push -u origin milestone/<slug>`
7. `gh pr create` — title and body from the milestone and its phases.
8. Monitor checks.
9. Merge the PR when checks and review conditions are satisfied.

**Tier B — Gated**
5. `git commit`
6. If on a branch: merge to the default branch locally
   (`git switch <default> && git merge --no-ff milestone/<slug>`), then push. If already on
   the default branch: push.
7. Optionally open a PR for the record — `gh pr create --fill && gh pr merge --auto --squash`.
   Do not wait on review; there is none. Use this when the repo's remote CI runs on PRs and
   a green check is worth having on the change.
8. If the milestone is a release, deploy, pilot, or partner demo — **or changed
   dependencies** — trigger the release gate now: dispatch the repo's gated workflow with
   the readiness reason, or push the release tag. Monitor it, and treat a failure as
   unfinished work rather than a separate problem.

**Tier C — Direct**
5. `git commit`
6. Push to the default branch (merging the branch down first if one was used).
7. If a packaging/publishing workflow exists and this change could affect it, check its
   result and record any failure as a roadmap item.

**All tiers, after landing:**
9. `git switch <default> && git pull --ff-only --prune` — return to the default branch,
   fast-forwarded, with stale remote-tracking refs pruned.
10. Delete the merged branch locally and remotely when it is safe to do so. Preserve any
    local branch with unreconciled commits that are not represented on the default branch.
11. Record the merge date on the milestone in `ROADMAP.md`, and the PR number where one
    exists.
12. Re-read `ROADMAP.md` and apply **Next-Step Handoffs**. The final user-facing response
    must include the next roadmap step, or a decision with options if more than one step
    is viable.

If any step needs credentials, external artifacts, or an explicit product decision, leave
it as an open item in `ROADMAP.md` and state the blocker clearly.

---

## Git autonomy

| Phase | Behavior |
|-------|----------|
| Run local verification, update `ROADMAP.md`, `git add`, draft commit message | Automatic |
| Push an existing branch as backup (no new remote refs, no merge) | Automatic |
| `git commit`, first `git push` of a branch, `gh pr create`, merge to default branch, release-gate dispatch or tag | **Pause for confirmation first** |

Confirmation is requested once, before the landing sequence; on approval the sequence runs
through, including monitoring checks and merging. Never merge to a default branch, open a
PR, or trigger a release gate without that confirmation.

## Response Expectations

When asked to review the current development plan, lead with:

1. Repo tier, and current milestone/status from `ROADMAP.md`.
2. Any mismatch between the plan and the repository state.
3. The recommended next step.
4. Whether that next step is planning, implementation, validation, or release work.

If multiple roadmap steps are viable, replace the single recommendation with a concise
decision: list the options, the immediate action for each, and ask the user to choose
before starting implementation.

Keep updates concise, but always mention plan/documentation changes when they affect
project state.

## Relationship to other skills

- **Planning skills** (e.g. `plan`, `plan-lite`) produce detailed, often dated plans.
  Those become *detail documents* linked from `ROADMAP.md`; this skill owns the durable
  roadmap and the git lifecycle.
- **`audit` / `debug` / `refactor` / `review`** operate *within* the current change; they
  do not manage the roadmap or the git lifecycle. In Tiers B and C they are how a change
  gets scrutiny, since no PR review will supply it.

## Definition of Done

- `ROADMAP.md` exists, is current, records the repo's tier, and `## Current State` matches
  the actual git state.
- Local verification passed and the working tree is clean.
- The work landed by its tier's route — Tier A merged via reviewed PR; Tier B merged behind
  a green local check, with the release gate run if it was a release; Tier C pushed to the
  default branch.
- Completed milestones are tagged `[x]` with merge date recorded, and PR number where one
  exists.
- Nothing was merged to a default branch before its documentation updates were complete.
- Any contract/schema/catalog shared with a sibling repo or downstream consumer was
  reconciled on both sides (or the owed edit was explicitly flagged to its owner) before
  the milestone was called done — no side left ahead of the other.

## Anti-Patterns (Never Do)

- Land work in Tier B or C without running the repo's check command — the local gate is the
  only gate those tiers have.
- Declare a milestone done with a dirty working tree.
- Apply Tier A ceremony to a Tier C repo, or skip a Tier A PR because the change looks
  small.
- Do milestone work directly on the default branch in Tier A.
- Let a branch stay open across many sessions and land as one big-bang merge.
- Treat remote CI as a substitute for review, or review as a substitute for remote CI —
  in Tiers B and C nobody is reviewing, so scrutiny comes from tests and the `review`/`audit`
  skills.
- Release, deploy, or bump dependencies in Tier B without running the clean-checkout
  release gate.
- Let `ROADMAP.md` and the active git branch drift out of sync without flagging it.
- Merge to a default branch, open a PR, or trigger a release gate without the
  completion-step confirmation.
- Let `ROADMAP.md` accumulate detail that belongs in a linked document.
- Start a second milestone while another is still open.
- Overwrite or revert unrelated working-tree changes when cutting a branch.
