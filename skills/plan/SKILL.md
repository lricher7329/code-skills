# Plan Skill

## Purpose

A high-rigor prompting approach for software planning, codebase review, and implementation. Produces grounded recommendations, concrete file-by-file edits, PR-sized work breakdown, determinism and test strategy, and alignment to referenced design documents.

## When to Use

Invoke this skill when the user requests:
- Architecture review or technology decision memo
- Codebase review and implementation plan
- Adding features to an existing system
- Schema/data model changes
- Deterministic simulation/synthetic data pipelines
- Evaluation frameworks and benchmarking
- Governance- or compliance-adjacent technical design

## Invocation

```
"use plan skill"
"full implementation plan"
"create plan for <feature>"
```

---

## Operating Principles (Non-Negotiable)

### 1. Read Before You Write
- If the user provides documents, plans, diagrams, YAML, config, or code: read them first.
- Explicitly reference relevant sections and explain how they constrain the implementation.

### 2. No Hand-Waving
Every recommendation must map to:
- A code location
- A schema field
- A config flag
- A test

### 3. Determinism by Design
- If randomness exists, seed handling must be explicit and reproducible.
- Prefer a single RNG object passed through layers; avoid global state.

### 4. Backward Compatibility and Safe Evolution
- Use schema versioning and defaults loaders.
- Avoid breaking existing data artifacts; use migration strategies.

### 5. Incremental Delivery
- Prefer small PR-sized steps over rewrites.
- Each PR must have entry/exit criteria and tests.

### 6. Feature-Flag Everything
- New logic or behavior changes must be toggleable.
- Preserve old behavior when flags are off.

### 7. Testing is Part of Implementation
- Provide unit, regression, and determinism tests.
- Map tests to acceptance criteria.

---

## Input Gathering (Before Starting)

Before producing output, identify or ask for:

### 1. References
Documents, code, YAML, or plans that must be read:
- Design docs, RFCs, architecture decisions
- Existing code files that will be modified
- Configuration or schema files

### 2. Objective
- **Primary objective**: What needs to be implemented
- **Out of scope**: What should explicitly not be done

### 3. Constraints
User-specified constraints plus defaults:
- Deterministic behavior under a seed
- Backward compatibility with schema versioning
- Minimal, incremental changes (no rewrites)
- Feature flags for behavior changes
- Tests required for new or modified behavior

### 4. Expected Deliverables
- Implementation plan with PR breakdown
- Code for new/modified modules
- Test plan

---

## Standard Workflow

### Step 1: Intake and Evidence

Identify and list all reference artifacts:
- Design docs, plans, RFCs, existing markdown, diagrams, PDFs, YAML, scripts

Extract from each:
- Constraints
- Invariants
- Expected outputs
- Planned sequencing
- Acceptance criteria

### Step 2: Codebase Reconnaissance

Identify:
- Entry points
- Data models
- Serialization boundaries
- Configuration mechanisms
- Randomness and seed propagation
- Output writers
- Test suite locations

Produce a concise **integration map** of modules and data flows.

### Step 3: Design and Implementation Plan

Provide:
- Updated data model/schema plan
- Configuration/feature flags
- Integration points (exact files/functions)
- Failure modes and guardrails
- Evaluation hooks (if relevant)

### Step 4: PR Plan

Break work into PR-sized chunks. Each PR includes:
- PR title
- Files touched
- What changes
- Tests
- Acceptance criteria

### Step 5: Code Output

Provide:
- Complete code for new modules
- Diffs or file-by-file replacement text for modifications
- Clear integration snippet(s)

### Step 6: Tests and Acceptance

Provide:
- Tests aligned to each PR
- Deterministic reproducibility tests
- Regression tests
- Schema migration/defaults tests

---

## Required Output Sections

Use these headings exactly:

```
## 1. References Read and Constraints Extracted

## 2. Repository Entry Points and Integration Map

## 3. Data Model / Schema Plan

## 4. Implementation Plan and PR Breakdown

## 5. Exact Code Changes (File-by-File)

## 6. Test Plan and Acceptance Criteria Mapping

## 7. Risks and Mitigations
```

---

## Output

Save the implementation plan to `.claude/plans/plan-YYYY-MM-DD.md` in the project root, where `YYYY-MM-DD` is the current date.

If multiple plans are created on the same day, append a sequence number: `plan-YYYY-MM-DD-2.md`.

---

## Reference Patterns

### Deterministic RNG Pattern

```python
# Create RNG once at pipeline start
rng = random.Random(seed)

# Pass rng through all generation functions
def generate_data(rng: random.Random, ...):
    value = rng.random()
    ...

# For per-entity determinism (optional):
# Derive local RNG seeds from stable IDs without global reseeding
entity_seed = hash(f"{entity_id}:{timestamp}") % (2**32)
entity_rng = random.Random(entity_seed)
```

### Schema Versioning Pattern

```python
CURRENT_SCHEMA_VERSION = "2.0"

def load_with_defaults(data: dict) -> dict:
    """Load data, adding missing fields with defaults."""
    version = data.get("schema_version", "1.0")

    # Add new fields with defaults
    if version < "2.0":
        data.setdefault("new_field", None)
        data.setdefault("feature_enabled", False)

    data["schema_version"] = CURRENT_SCHEMA_VERSION
    return data
```

### Feature Flag Pattern

```yaml
# config.yaml
features:
  new_generator: false
  enhanced_validation: true
  experimental_output: false
```

```python
def generate(config, data):
    if config.features.get("new_generator", False):
        return new_generator(data)
    return legacy_generator(data)
```

### PR Breakdown Pattern

```markdown
### PR 1: Add schema versioning support

**Scope**: Introduce schema_version field and defaults loader

**Files touched**:
- src/models/schema.py (new)
- src/loaders/data_loader.py (modify)
- tests/test_schema.py (new)

**Changes**:
1. Add SchemaVersion enum
2. Implement load_with_defaults()
3. Update DataLoader to use versioned loading

**Tests**:
- test_load_v1_data_with_defaults
- test_load_v2_data_unchanged
- test_version_upgrade_path

**Acceptance criteria**:
- [ ] V1 data loads with new defaults populated
- [ ] V2 data loads unchanged
- [ ] Version field present in all outputs
```

---

## Validation Checklist

Before final output, confirm:

- [ ] All reference documents were read and constraints extracted
- [ ] Integration points are named (file + function/class)
- [ ] Schema/data model plan exists with versioning/backward compatibility
- [ ] Changes are feature-flagged and default-safe
- [ ] Determinism is preserved and tested
- [ ] PR plan is incremental with tests per PR
- [ ] Concrete code is provided for new/modified modules
- [ ] Risks and mitigations are explicitly documented

---

## Execution Rules

- No generic architecture talk; map each change to a file/function
- If files differ from reference docs, note mismatch and propose minimal reconciliation
- Keep changes modular and feature-flagged
- Preserve behavior when new features are disabled

---

## Anti-Patterns (Never Do)

- Suggest vague changes without code or file references
- Skip reading provided artifacts
- Propose a rewrite unless explicitly requested
- Ignore backward compatibility and schema evolution
- Ignore determinism and testing
- Hand-wave with "you could also..." without concrete mapping

---

## Domain-Specific Patterns

For specialized domains, see the patterns directory:
- `patterns/clinical.md` - Healthcare, clinical trials, research data

---

## Template

For programmatic use with orchestrators, see `TEMPLATE.md` in this directory.
