# Audit Skill

## Purpose

Perform a **codebase-wide quality, correctness, and consistency audit**. Produces a **concrete, actionable, evidence-backed report** that identifies gaps, inconsistencies, risks, and technical debt.

This is a **read-only review**. No fixes, refactors, or tests are implemented unless explicitly requested.

## When to Use

Use this skill when:
- Starting work on an unfamiliar codebase
- Preparing for a major refactor or feature addition
- Assessing technical debt before a release
- Evaluating code quality after rapid development
- Onboarding to understand architectural boundaries and risks
- Auditing an AI/agent system prior to scale, deployment, or evaluation

## When NOT to Use

Use other skills instead if:
- Reviewing a specific PR or diff → `review`
- Fixing a known bug → `debug`
- Planning new implementation → `plan` or `plan-lite`
- Improving structure without changing behavior → `refactor`

## Invocation

```
"use audit skill"
"audit this codebase"
"quality review"
"codebase consistency check"
```

---

## Preflight (Required)

Before beginning the audit, perform a **Preflight assessment** and report it explicitly in the output.

### Preflight Checklist

- Repository size and structure (top-level directories and purpose)
- Primary languages, frameworks, and runtimes
- Entry points (CLI, API, workers, scripts)
- Test framework(s) and CI presence
- Configuration mechanisms (env vars, config files, defaults)
- AI/ML stack (models, providers, vector stores, prompt templates)
- Constraints or access limits (permissions, time, or missing dependencies)
- Anything that could not be inspected or verified (with reason)

**The audit is not considered complete without a documented Preflight section.**

## Definition of Done

An audit is considered complete only if:
- Repository structure has been scanned and summarized
- At least one end-to-end execution path has been traced
- Representative files from each major subsystem were reviewed
- Configuration, schema, and testing strategies were inspected
- Findings are supported by concrete file-level evidence
- All issues include severity, confidence, and remediation guidance
- Output follows the Required Output Format exactly

---

## Review Dimensions

Evaluate the codebase across **all dimensions below**.

### 1. Testing & Quality Gates
- Are new features consistently accompanied by tests?
- Are tests deterministic, isolated, and named consistently?
- Are regression tests present where randomness or heuristics exist?
- Are CI, linting, formatting, or pre-commit hooks enforcing quality?
- Are there orphan, flaky, or dead tests?

### 2. Schema & Model Consistency
- How are schemas defined (Pydantic, dataclasses, TypedDicts, plain dicts)?
- Are the same entities defined differently across layers?
- Are serialization/deserialization boundaries explicit?
- Are defaults, optional fields, and nullability handled uniformly?
- Are output schemas enforced for AI/model responses?

### 3. Configuration & Feature Flags
- Are feature flags used consistently and intentionally?
- Are there hard-coded behaviors that should be configurable?
- Are configs validated or assumed?
- Are environment-specific behaviors explicit?
- Is configuration drift detectable?

### 4. Cross-Module References
- Do modules agree on field names, types, and semantics?
- Are constants duplicated across files?
- Are imports and dependencies clean or tangled?
- Are there implicit contracts not captured in code?

### 5. Determinism & Reproducibility
- Is randomness centrally managed or scattered?
- Are seeds passed explicitly and logged?
- Are timestamps, global state, or mutable defaults leaking nondeterminism?
- Can an execution be reproduced from logs/config alone?

### 6. Ownership & Responsibility Boundaries
- Which layer owns "truth" vs derived data?
- Who owns schema definition, validation, persistence?
- Are responsibilities leaking across layers?
- Are boundaries explicit or implicit?

### 7. Observability & Debuggability
- Is logging structured, leveled, and contextual?
- Are correlation IDs or request IDs propagated?
- Are metrics collected for latency, errors, throughput, cost?
- Is there sufficient context to reproduce failures?

### 8. Security & Privacy
- Are secrets handled securely (no hard-coded credentials)?
- Are PII/PHI handling rules explicit and enforced?
- Are access controls and audit logs present where needed?
- Are dependencies pinned and vulnerabilities considered?

### 9. Performance & Cost Controls
- Are hot paths bounded or profiled?
- Are batch vs per-item operations used appropriately?
- Is caching used intentionally and invalidated safely?
- Are cost controls present (token limits, retries, backoff)?

### 10. AI-Specific Interfaces & Guardrails
- Is there a single abstraction for LLM/model calls?
- Are prompts versioned, validated, and testable?
- Are model outputs schema-validated and repairable?
- Are RAG boundaries (chunking, retrieval, ranking) explicit?
- Are guardrails, timeouts, and failure modes defined?

---

## Severity and Confidence Model

Every issue **must** include both:

### Severity
- **S0 – Blocking:** correctness, data-loss, security, or regulatory risk
- **S1 – High:** likely production failure or major developer friction
- **S2 – Medium:** scalable defect or maintainability risk
- **S3 – Low:** localized or opportunistic technical debt

### Confidence
- **High:** confirmed via direct code path inspection
- **Medium:** strongly inferred with partial confirmation
- **Low:** plausible but not fully traceable

## Issue ID Prefixes

Use these category prefixes for all Issue IDs:

| Prefix | Category |
|--------|----------|
| TST-### | Testing & Quality Gate Issues |
| SCH-### | Schema & Model Inconsistencies |
| CFG-### | Configuration & Feature Flag Issues |
| XMOD-### | Cross-Module Inconsistencies |
| DET-### | Determinism & Reproducibility Risks |
| OWN-### | Ownership & Boundary Issues |
| OBS-### | Observability & Debuggability Issues |
| SEC-### | Security & Privacy Risks |
| PERF-### | Performance & Cost Risks |
| AI-### | AI Interface & Guardrail Issues |

Use a three-digit numeric suffix (e.g., `TST-001`) and increment within each category.

---

## Execution Rules

### Must Do
- Read the code before drawing conclusions
- Reference **specific files and symbols** for every issue
- Assign **Severity and Confidence** to every issue
- Describe the **failure mode** for all S0/S1 issues
- Include a **recommended remediation approach**
- Prefer incremental, PR-sized fixes
- State explicitly what could not be verified

### Must Not Do
- Rewrite code
- Refactor or add tests
- Generalize without concrete examples
- Assume intent; infer only from code
- Propose large rewrites unless unavoidable
- Introduce new frameworks without justification
- Include stylistic nitpicks

## Language Style

Be precise and evidence-based.

**Correct:**
> File `schemas/user.py` defines `UserModel` using Pydantic with optional `email`, while `services/auth.py` assumes `email` is always present and non-null.

**Incorrect:**
> Schemas are inconsistent.

---

## Required Output Format

Use these headings **exactly**:

```markdown
## 0. Preflight Summary

## 1. Audit Scope and Method
- Subsystems reviewed
- Sampling strategy
- Execution paths traced

## 2. Summary of Key Findings
- High-level themes
- Areas of greatest risk

## 3. Detailed Findings by Category

### 3.1 Testing & Quality Gate Issues
- Issue ID (e.g., TST-001)
- Severity / Confidence
- Issue description
- File(s) involved
- Evidence (file:line and symbol)
- Why this is a problem
- Recommended remediation

### 3.2 Schema & Model Inconsistencies
(same structure)

### 3.3 Configuration & Feature Flag Issues
(same structure)

### 3.4 Cross-Module Inconsistencies
(same structure)

### 3.5 Determinism & Reproducibility Risks
(same structure)

### 3.6 Ownership & Boundary Issues
(same structure)

### 3.7 Observability & Debuggability Issues
(same structure)

### 3.8 Security & Privacy Risks
(same structure)

### 3.9 Performance & Cost Risks
(same structure)

### 3.10 AI Interface & Guardrail Issues
(same structure)

## 4. Suggested Remediation Plan
- Group issues into PR-sized tasks
- Order by severity (S0/S1 → S2 → S3)
- Identify safe vs high-risk changes

## 5. Risks of Not Addressing These Issues
- Impact on correctness
- Impact on evaluation or benchmarking
- Impact on maintainability and scale
```

---

## Output

Save the audit report to `.claude/audits/audit-YYYY-MM-DD.md` in the project root, where `YYYY-MM-DD` is the current date.

If multiple audits are run on the same day, append a sequence number: `audit-YYYY-MM-DD-2.md`.

---

## Workflow

1. Perform Preflight assessment
2. Scan repository structure and identify subsystems
3. Review representative files per subsystem
4. Trace at least one end-to-end execution path
5. Validate findings against actual usage
6. Produce audit report using required format
7. Save report to `.claude/audits/`

---

## Relationship to Other Skills

| Skill | Purpose |
|-------|---------|
| audit | Read-only quality and risk assessment |
| plan | Full remediation planning after audit |
| plan-lite | Targeted fixes for individual findings |

**Always run audit first** to understand the system state before planning changes.

---

## Self-Check

Before finalizing the audit report:

- [ ] I completed and documented the Preflight Summary
- [ ] I traced at least one end-to-end execution path
- [ ] I reviewed representative files in each major subsystem
- [ ] Every issue has an Issue ID, Severity, and Confidence
- [ ] Every issue cites concrete evidence (file:line and symbol)
- [ ] Every issue includes a failure mode (for S0/S1) and a remediation approach
- [ ] The output follows the Required Output Format exactly

---

## Anti-Patterns (Never Do)

- Audit without reading actual code
- Generalize findings without specific file/line evidence
- Skip the Preflight assessment
- Propose fixes (this is read-only)
- Include stylistic opinions as findings
- Report issues without severity/confidence ratings
