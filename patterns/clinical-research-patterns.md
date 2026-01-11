# Clinical/Research Domain Patterns

Domain-specific patterns for healthcare, clinical trials, and research data systems.

---

## Quick Reference

| Building... | Key Sections |
|-------------|--------------|
| Synthetic clinical data | [Synthetic Data Generation](#synthetic-data-generation), [Measurement Uncertainty](#measurement-uncertainty), [Temporal Constraints](#temporal-constraint-framework) |
| Clinical trial design | [Outcome Specification](#clinical-trial-outcome-specification), [Regulatory References](#regulatory-reference-mapping) |
| Clinical NLP/extraction | [Multi-Stage Extraction](#multi-stage-extraction-pipelines), [Evidence Spans](#evidence-spans), [Verification Failure Reasons](#verification-failure-reasons), [Adjudication Workflow](#adjudication-workflow) |
| Research data platform | [Trial-Level Data](#trial-level-data-preservation), [Visit Scheduling](#visit-scheduling-patterns), [Consent Versioning](#consent-versioning) |
| Integration layer | [Watermark Sync](#watermark-based-incremental-sync), [Field Mapping](#external-system-field-mapping), [Sync Logging](#sync-logging) |
| Offline/mobile collection | [Offline-First](#offline-first-with-sync-queue) |
| Multi-site research | [Multi-Site/Multi-Tenant](#multi-site-and-multi-tenant-patterns), [Role-Based Access](#role-based-access-control) |
| Security & compliance | [Audit Logging](#audit-logging), [PHI Handling](#phi-handling-patterns), [Feature Flags](#feature-flags-for-clinical-features) |
| Any clinical system | [Truth vs Docs](#truth-vs-documentation-separation), [Provenance](#provenance), [Quality Flags](#quality-flags-and-risk-assessment) |

---

## Truth vs Documentation Separation

In clinical and research contexts, maintain separation between:
- **Ground truth**: The actual data values, measurements, provenance
- **Documentation/narrative**: Human-readable reports, summaries, interpretations
- **Derived artifacts**: AI-generated content, computed scores, extracted entities

### Why This Matters
- Evaluations and benchmarks must compare against ground truth
- Narrative documents may contain ambiguity, interpretation, or simplification
- Audit trails require traceable provenance to source data
- AI outputs should never overwrite canonical data

### Pattern
```python
class ClinicalRecord:
    # Truth layer - immutable source data
    truth: TruthLayer

    # Derived/narrative layer - can be regenerated
    narrative: NarrativeLayer

    def regenerate_narrative(self):
        """Narrative is derived from truth, not vice versa."""
        self.narrative = generate_from_truth(self.truth)
```

### AI Artifact Separation
```python
@dataclass
class AIArtifact:
    """AI outputs stored separately with full provenance."""
    entity_type: str           # PATIENT, ENCOUNTER, NOTE
    entity_id: str
    artifact_type: str         # EXTRACTION, SUMMARY, EMBEDDING
    model_name: str
    model_version: str
    prompt_template_id: str
    input_hash: str            # SHA256 of inputs for cache invalidation
    confidence_score: float
    output_json: dict
    created_at: datetime
    expires_at: datetime | None

# AI is always advisory - human confirmation required for high-impact decisions
# Never: automatic approvals, eligibility determinations, diagnoses
```

---

## Per-Entity Deterministic Generation

When generating synthetic clinical data, ensure reproducibility at the entity level.

### Pattern
```python
# Derive local RNG seeds from stable identifiers
# This ensures the same patient always generates the same data
entity_seed = hash(f"{patient_id}:{visit_date}") % (2**32)
entity_rng = random.Random(entity_seed)

# Use entity_rng for all generation related to this entity
lab_value = generate_lab_result(entity_rng, test_type)
```

### LLM Determinism
```python
# For reproducible LLM extraction, use zero temperature
llm_config = {
    "model": "claude-sonnet-4-20250514",
    "temperature": 0.0,  # Deterministic extraction
    "max_tokens": 4096,
}

# Store versioned prompt templates
@dataclass
class PromptTemplate:
    name: str
    version: str              # Semantic versioning
    system_prompt: str
    user_prompt_template: str
    output_schema_json: dict
    created_at: datetime      # Immutable after creation
```

### Why This Matters
- Regression testing: same patient should produce same synthetic data
- Debugging: can reproduce specific patient scenarios
- Evaluation: consistent test sets across runs
- Audit: can verify what model/prompt produced an output

---

## Synthetic Data Generation

For generating realistic synthetic clinical data at scale.

### Scenario-Based Generation
```python
@dataclass
class ScenarioDefinition:
    """Combinable clinical scenarios for realistic patient profiles."""
    scenario_id: str
    scenario_name: str
    category: str              # disease_course|treatment_response|access|comorbidity

    clinical_criteria: dict    # {"max_edss_10_years": 3.0, "progression_rate": "BENIGN"}
    age_eligibility: dict      # {"min": 18, "max": 65}
    gender_requirement: str | None
    incompatibilities: list[str]  # Scenario IDs that conflict
    probability_weight: float

# Example scenarios (can be combined)
SCENARIOS = {
    # Disease course
    "benign_ms": {"max_edss_10_years": 3.0, "progression_rate": "BENIGN"},
    "aggressive_ms": {"min_edss_5_years": 6.0, "progression_rate": "AGGRESSIVE"},
    "pediatric_onset": {"age_at_onset": {"max": 18}, "relapse_frequency": "high"},

    # Treatment response
    "high_responder": {"neda_status": True},
    "treatment_intolerant": {"discontinuation_reason": "side_effects"},

    # Access/social
    "rural_low_access": {"distance_to_clinic_km": {"min": 200}},
    "disability_pension": {"disability_status": "AISH"},
}

# Patients can have multiple non-conflicting scenarios
patient.scenarios = ["benign_ms", "high_responder", "rural_low_access"]
```

### Measurement Uncertainty
```python
@dataclass
class MeasurementWithUncertainty:
    """Separate true value from documented value."""
    true_value: float              # Ground truth (for evaluation)
    documented_value: float        # What was recorded (may differ)
    confidence: str                # HIGH|MODERATE|LOW|UNCERTAIN
    measurement_modality: str      # IN_PERSON|TELEHEALTH|PHONE

# Modality-specific noise models
MEASUREMENT_NOISE = {
    "IN_PERSON": {"std": 0.0, "max_discrepancy": 0.0},
    "TELEHEALTH_VIDEO": {"std": 0.5, "max_discrepancy": 1.0},
    "TELEHEALTH_PHONE": {"std": 1.0, "max_discrepancy": 1.5},
}

# Component omission rates (what clinicians forget to document)
OMISSION_RATES = {
    "cognition": 0.50,     # 50% omission rate
    "fatigue": 0.35,
    "bladder_function": 0.40,
}
```

### Provider Style Variation
```python
# Different clinicians document differently
PROVIDER_STYLES = {
    "JM": {"style": "detailed_systematic", "avg_word_count": 600},
    "NN": {"style": "concise_technical", "avg_word_count": 300},
    "PS": {"style": "narrative_comprehensive", "avg_word_count": 500},
}

# Generate notes with provider-specific variation
def generate_note(patient, visit, provider_id):
    style = PROVIDER_STYLES[provider_id]
    # Apply style-specific templates and verbosity
```

---

## Temporal Constraint Framework

For longitudinal synthetic data, enforce temporal coherence.

### Pattern
```python
@dataclass
class TemporalConstraints:
    """Control temporal aspects of data generation."""
    current_date: date              # "Today" for masking future data
    respect_historical_timeline: bool = True  # e.g., medications only when available
    include_deceased_notes: bool = True
    max_notes_per_patient: int | None = None
    start_date: date | None = None  # Only generate recent notes
    end_date: date | None = None

# Historical availability (medications, tests, procedures)
AVAILABILITY_TIMELINE = {
    "Betaseron": date(1995, 7, 1),
    "Tysabri": date(2006, 11, 1),
    "Ocrevus": date(2017, 8, 1),
    "Kesimpta": date(2021, 1, 1),
}

# Temporal coherence rules
TEMPORAL_RULES = {
    "edss_progression": {"max_increase_per_year": 1.0},
    "medication_washout": {
        "Tysabri": 28,      # days
        "Gilenya": 60,
        "Ocrevus": 180,
    },
    "visit_intervals": {
        "routine": 180,     # days
        "relapse": 14,
        "medication_change": 30,
    },
}
```

### Longitudinal Visit Generation
```python
def generate_visit_timeline(patient, constraints: TemporalConstraints):
    """Generate visits respecting temporal rules."""
    visits = []
    current_date = patient.diagnosis_date

    while current_date <= constraints.current_date:
        visit_type = determine_visit_type(patient, current_date)
        interval = TEMPORAL_RULES["visit_intervals"][visit_type]

        # Enforce constraints
        if patient.current_medication:
            assert is_medication_available(patient.current_medication, current_date)

        visits.append(generate_visit(patient, current_date, visit_type))
        current_date += timedelta(days=interval)

    return visits
```

---

## Clinical Trial Outcome Specification

For trial design and simulation, fully specify outcomes.

### Pattern
```python
@dataclass
class OutcomeSpecification:
    """Complete specification for a trial outcome."""
    name: str                      # "TMT B/A Ratio"
    description: str

    # Population parameters (for simulation)
    population_mean: float
    population_sd: float

    # Clinical interpretation
    mcid: float                    # Minimum Clinically Important Difference
    range: tuple[float, float]     # Valid measurement range
    direction: str                 # "lower_better" | "higher_better"

    # Baseline adjustment (for heterogeneous populations)
    baseline_adjustment: float     # Expected deviation in target population

# Example: Long COVID trial outcomes
OUTCOMES = {
    "tmt": OutcomeSpecification(
        name="TMT B/A Ratio",
        description="Trail Making Test cognitive assessment",
        population_mean=2.22,
        population_sd=1.07,
        mcid=0.5,
        range=(0.9, 5.0),
        direction="lower_better",
        baseline_adjustment=0.2,
    ),
    "mfis": OutcomeSpecification(
        name="MFIS",
        description="Modified Fatigue Impact Scale",
        population_mean=23.7,
        population_sd=21.1,
        mcid=10,
        range=(0, 84),
        direction="lower_better",
        baseline_adjustment=10,
    ),
}
```

### Co-Primary Outcomes
```python
@dataclass
class CoPrimaryDesign:
    """Design with multiple primary outcomes (AND logic)."""
    outcomes: list[str]            # ["tmt", "mfis"]
    success_threshold: float       # 0.95 (posterior probability)
    success_logic: str             # "AND" - both must succeed

    def evaluate_success(self, posterior_probs: dict) -> bool:
        """Success requires ALL outcomes to exceed threshold."""
        return all(
            posterior_probs[outcome] >= self.success_threshold
            for outcome in self.outcomes
        )

# Decision rule
design = CoPrimaryDesign(
    outcomes=["tmt", "mfis"],
    success_threshold=0.95,
    success_logic="AND",
)
```

---

## Regulatory Reference Mapping

For protocol elements and compliance tracking.

### Pattern
```python
@dataclass
class RegulatoryReference:
    """Link content to regulatory authorities."""
    source_code: str              # ICH_E6R3, TCPS2, HC_DIV5
    source_name: str
    version: str
    section: str | None           # "4.2.1"
    url: str | None

@dataclass
class ProtocolElement:
    """Reusable, regulation-compliant protocol language."""
    element_id: str               # "ae_definition_standard"
    name: str
    category: str                 # adverse_events|consent|eligibility|statistical
    status: str                   # draft|approved|deprecated

    content: str                  # Markdown formatted
    customization_notes: str      # How to adapt for specific use

    # Regulatory mappings
    regulations: list[RegulatoryReference]
    spirit_items: list[tuple[str, str]]   # [("17a", "Safety assessment")]

    # Applicability
    populations: list[str]        # ["adult", "pediatric"]

    # Governance
    version: str
    approved_at: datetime | None
    approved_by: str | None

# Example regulatory sources
REGULATORY_SOURCES = {
    "ICH_E2A": "Clinical Safety Data Management",
    "ICH_E6R3": "Good Clinical Practice (2025)",
    "ICH_E11": "Pediatric Clinical Trials",
    "TCPS2": "Tri-Council Policy Statement (Canada)",
    "HC_DIV5": "Health Canada Division 5",
    "SPIRIT_2025": "Trial Registration Checklist",
}
```

### SPIRIT Checklist Mapping
```python
@dataclass
class SpiritMapping:
    """Map elements to SPIRIT 2025 checklist items."""
    spirit_item: str              # "17a"
    spirit_item_name: str         # "Safety assessment methods"
    relevance: str                # primary|secondary|supporting

# Find all elements for a SPIRIT item
def get_elements_for_spirit_item(item: str) -> list[ProtocolElement]:
    """Retrieve protocol elements mapped to a specific SPIRIT item."""
    return query_elements_by_spirit(item)

# Build SPIRIT-compliant protocol
def build_protocol_checklist(spirit_items: list[str]) -> dict:
    """Generate protocol skeleton from SPIRIT checklist."""
    return {
        item: get_elements_for_spirit_item(item)
        for item in spirit_items
    }
```

### Population-Specific Variants
```python
# Elements can have population-specific versions
POPULATION_VARIANTS = {
    "ae_definition": {
        "adult": "ae_definition_standard",
        "pediatric": "ae_definition_pediatric",
        "neonatal": "ae_definition_neonatal",
    },
    "consent_process": {
        "adult": "consent_process_adult",
        "pediatric": "consent_process_pediatric",  # Assent + parental
        "incapacity": "consent_process_incapacity",
    },
}

def get_element_for_population(element_type: str, population: str) -> str:
    """Get appropriate element variant for population."""
    variants = POPULATION_VARIANTS.get(element_type, {})
    return variants.get(population, variants.get("adult"))
```

---

## Multi-Stage Extraction Pipelines

For clinical information extraction, use a layered approach with verification.

### Pattern
```python
# Stage 0: Rules Engine (Deterministic, high precision)
class RulesExtractor:
    """Fast, interpretable, no LLM cost."""
    def extract(self, text: str) -> RulesCandidates:
        # Regex patterns, vocabulary lookup, negation detection
        pass

# Stage 1: LLM Extraction (High recall)
class LLMExtractor:
    """Uses rules output as context/hints."""
    def extract(self, text: str, rules_hints: RulesCandidates) -> Extraction:
        pass

# Stage 2: Verification Engine (Quality gate)
class VerificationEngine:
    """Post-extraction validation before human review."""
    def verify(self, extraction: Extraction, source_text: str) -> VerificationResult:
        # Evidence span validation
        # Negation/uncertainty checking
        # Cross-field consistency
        # Rules vs LLM conflict detection
        pass

# Gating decision
if verification_result.passed:
    auto_accept(extraction)
else:
    queue_for_human_review(extraction, verification_result.issues)
```

### Verification Failure Reasons
```python
class VerificationFailureReason(Enum):
    EVIDENCE_NOT_FOUND = "evidence_not_found"
    EVIDENCE_SPAN_INVALID = "evidence_span_invalid"
    EVIDENCE_NEGATED = "evidence_negated"
    EVIDENCE_UNCERTAIN = "evidence_uncertain"
    TEMPORAL_INCONSISTENCY = "temporal_inconsistency"
    HISTORICAL_AS_CURRENT = "historical_promoted_to_current"
    CROSS_FIELD_CONFLICT = "cross_field_conflict"
    RULE_LLM_CONFLICT = "rule_llm_conflict"
```

---

## Schema Patterns for Clinical Data

### Patient Identifiers
```python
@dataclass
class PatientIdentifier:
    patient_id: str          # Internal stable ID (UUID)
    mrn: str | None          # Medical record number (may change)
    study_id: str | None     # Research study identifier
    external_ids: dict       # {"hr_id": "...", "ehr_id": "..."}

@dataclass
class PatientSourceMap:
    """Track same patient across source systems."""
    patient_id: str
    source_system: str       # HR, EHR, REGISTRY
    source_patient_id: str
    last_seen_at: datetime
    metadata: dict
```

### Temporal Data with Precision
```python
@dataclass
class ClinicalEvent:
    event_id: str
    patient_id: str
    event_datetime: datetime      # When it occurred
    event_datetime_precision: str # day|month|year|approximate
    recorded_datetime: datetime   # When it was recorded (may differ)
    temporal_context: str         # CURRENT|HISTORICAL|PLANNED|UNCERTAIN
    event_type: str
    data: dict

# Example: medication event with temporal precision
@dataclass
class MedicationEvent:
    medication_name: str
    medication_normalized: str    # Canonical name
    event_type: str               # start|continuation|dose_change|discontinuation
    event_date: date
    event_date_precision: str     # day|month|year|approximate
    is_current: bool
    confidence: float
    evidence: list[EvidenceSpan]
```

### Evidence Spans
```python
@dataclass
class EvidenceSpan:
    """Link extracted data to source text."""
    text: str
    span_start: int
    span_end: int
    source_document_id: str | None
```

### Provenance
```python
@dataclass
class DataProvenance:
    source_system: str
    extraction_datetime: datetime
    transformation_version: str
    schema_version: str

@dataclass
class ExtractionProvenance:
    """Full provenance for AI extractions."""
    extractor_model_id: str
    extractor_version: str
    extraction_timestamp: datetime
    extraction_duration_ms: int
    prompt_tokens: int
    completion_tokens: int
    pipeline_stages: list[str]
```

---

## Trial-Level Data Preservation

For research assessments, preserve granular trial data alongside computed summaries.

### Pattern
```python
@dataclass
class AssessmentResponse:
    """Preserve raw data for re-analysis."""
    session_id: str
    module_code: str
    trial_index: int
    trial_data: dict           # Full raw data (stimulus, response, RT, etc.)
    summary_scores: dict | None # Computed aggregates (optional)
    created_at: datetime

# Benefits:
# - Re-scoring: algorithm changes don't require re-collection
# - Quality checks: validate summaries match trial counts
# - Research: RT distributions, learning curves, error analysis
```

### Summary Calculation
```python
def calculate_task_summary(trials: list[dict]) -> dict:
    """Compute summary scores from trial-level data."""
    valid_trials = [t for t in trials if not t.get("anticipatory")]
    rts = [t["rt"] for t in valid_trials]

    return {
        "total_trials": len(trials),
        "valid_trials": len(valid_trials),
        "mean_rt": mean(rts),
        "median_rt": median(rts),
        "rt_variability": stdev(rts),
        # Task-specific metrics...
    }
```

---

## Visit Scheduling Patterns

For longitudinal studies, use window-based scheduling.

### Pattern
```python
@dataclass
class VisitWindow:
    """Define visit timing relative to enrollment."""
    visit_code: str           # baseline, week4, month6
    target_day: int           # Days from enrollment (0 = baseline)
    window_before_days: int   # How early is acceptable
    window_after_days: int    # How late is acceptable
    battery_id: str           # What assessments to complete

@dataclass
class ScheduledVisit:
    """Instance of a visit for a specific participant."""
    participant_id: str
    visit_window_id: str
    scheduled_date: date
    window_start: date        # Computed: scheduled - before
    window_end: date          # Computed: scheduled + after
    status: str               # scheduled|in_window|overdue|completed|missed
    reminder_count: int
    reminder_sent_at: datetime | None
    completed_at: datetime | None
```

---

## Offline-First with Sync Queue

For field data collection, design for offline operation.

### Pattern
```python
@dataclass
class SyncQueueItem:
    """Queue items for background sync."""
    id: str
    entity_type: str          # session, response, consent
    entity_id: str
    payload: dict
    status: str               # pending|processing|synced|failed
    retry_count: int
    max_retries: int
    last_attempt: datetime | None
    error: str | None
    created_at: datetime

class SyncQueueManager:
    """Manages offline sync with retry backoff."""

    def __init__(
        self,
        batch_size: int = 10,
        retry_delay_ms: int = 1000,
        max_retry_delay_ms: int = 60000,
    ):
        pass

    def enqueue(self, item: SyncQueueItem):
        """Add item to sync queue."""
        pass

    def process_queue(self):
        """Process pending items with exponential backoff."""
        pass
```

### Client-Side Storage
```typescript
// IndexedDB schema for offline-first apps
interface OfflineSession {
  id: string;
  syncStatus: 'pending' | 'syncing' | 'synced' | 'failed';
  syncAttempts: number;
  lastSyncAttempt?: string;
  serverId?: string;  // Populated after successful sync
  data: SessionData;
}
```

---

## Integration Patterns

### Watermark-Based Incremental Sync
```python
# For large external systems, use watermarks for incremental sync
@dataclass
class IntegrationRun:
    """Track integration job execution."""
    integration_name: str
    run_id: str
    status: str               # STARTED|SUCCEEDED|FAILED|PARTIAL
    started_at: datetime
    completed_at: datetime | None
    watermark_field: str      # e.g., "last_modified_at"
    watermark_value: datetime
    records_processed: int
    records_created: int
    records_updated: int
    error_summary: str | None

# Configuration
INTEGRATION_CONFIG = {
    "hr_ingest": {
        "enabled": True,
        "mode": "INCREMENTAL",  # or FULL_REFRESH
        "watermark_field": "last_modified_at",
        "full_refresh_day": "Sunday",  # Weekly full refresh
        "cron": "0 2 * * *",
    },
}
```

### External System Field Mapping
```python
# Per-study/site mapping to external systems (e.g., REDCap)
@dataclass
class FieldMapping:
    """Map internal fields to external system fields."""
    study_id: str
    source_field: str         # Internal: "cpt.d_prime"
    target_field: str         # External: "cpt_dprime"
    transform: str | None     # Optional: "round(2)"

# Store mapping per study for flexibility
study_config = {
    "redcap_enabled": True,
    "redcap_field_mapping": {
        "simple_rt": {"mean_rt": "sr_mean_rt", "median_rt": "sr_med_rt"},
        "cpt": {"d_prime": "cpt_dprime", "omission_errors": "cpt_omit"},
    },
}
```

### Sync Logging
```python
@dataclass
class ExternalSyncLog:
    """Audit trail for external system sync."""
    sync_id: str
    entity_type: str
    entity_id: str
    external_system: str
    status: str               # pending|synced|failed
    record_data: dict         # Snapshot of what was sent
    external_response: dict   # Response from external system
    synced_at: datetime | None
    error_message: str | None
```

---

## Consent Versioning

For regulatory compliance, track consent versions and withdrawals.

### Pattern
```python
@dataclass
class ConsentForm:
    """Versioned consent document."""
    form_id: str
    version: str              # "1.0", "1.1", etc.
    language: str             # "en", "fr"
    sections: list[dict]      # Structured consent items
    effective_date: date
    created_at: datetime

@dataclass
class ConsentRecord:
    """Participant's consent with audit trail."""
    participant_id: str
    consent_form_id: str
    consented_at: datetime
    signature_data: str | None    # Base64 signature image
    ip_address: str | None
    user_agent: str | None
    withdrawn_at: datetime | None
    withdrawal_reason: str | None
```

---

## Feature Flags for Clinical Features

```yaml
# config.yaml
features:
  # Data generation
  synthetic_lab_noise: true
  realistic_temporal_gaps: true

  # Extraction
  enable_llm_extraction: true
  enable_rules_extraction: true
  enable_verification: true

  # Evaluation
  include_edge_cases: true
  generate_adversarial_cases: false

  # Output
  include_provenance: true
  redact_identifiers: true

  # External sync
  redcap_enabled: false
  sync_mode: INCREMENTAL  # or FULL_REFRESH
```

### Environment-Based Feature Toggles
```python
# Conservative defaults - opt-in, disabled by default
FEATURE_FLAGS = {
    'AI_EXTRACTION': env_bool('AI_EXTRACTION', False),
    'SEMANTIC_SEARCH': env_bool('SEMANTIC_SEARCH', False),
    'DUPLICATE_DETECTION': env_bool('DUPLICATE_DETECTION', False),
    'REDCAP_SYNC': env_bool('REDCAP_SYNC', False),
}

# In code
if settings.FEATURE_FLAGS.get('AI_EXTRACTION'):
    extraction = await extract_with_llm(note)
else:
    extraction = extract_with_rules_only(note)
```

---

## Quality Flags and Risk Assessment

Track data quality issues for review prioritization.

### Pattern
```python
@dataclass
class QualityFlag:
    flag_type: str            # missing_data|contradiction|ambiguity|low_confidence
    component: str            # "disease_course.ms_type"
    message: str
    severity: str             # info|warning|error

@dataclass
class QualityAssessment:
    overall_confidence: float
    flags: list[QualityFlag]
    requires_human_review: list[str]  # Component paths needing review
```

---

## Audit Logging

For regulatory compliance (GCP, HIPAA), maintain comprehensive audit trails.

### Pattern
```python
from enum import Enum
from dataclasses import dataclass
from datetime import datetime

class AuditAction(str, Enum):
    CREATE = "create"
    READ = "read"
    UPDATE = "update"
    DELETE = "delete"
    EXPORT = "export"
    LOGIN = "login"
    LOGOUT = "logout"
    CONSENT = "consent"
    WITHDRAW = "withdraw"

@dataclass
class AuditLogEntry:
    """Immutable audit log entry for compliance."""
    log_id: str
    timestamp: datetime
    action: AuditAction
    entity_type: str          # patient, session, note, consent
    entity_id: str
    user_id: str
    user_role: str
    ip_address: str | None
    user_agent: str | None
    old_value: dict | None    # For updates
    new_value: dict | None    # For creates/updates
    reason: str | None        # Required for deletes/exports
    study_id: str | None
    site_id: str | None

# Audit log is append-only - never update or delete
class AuditLogger:
    def log(
        self,
        action: AuditAction,
        entity_type: str,
        entity_id: str,
        user: User,
        old_value: dict | None = None,
        new_value: dict | None = None,
        reason: str | None = None,
    ):
        """Write immutable audit entry."""
        entry = AuditLogEntry(
            log_id=str(uuid.uuid4()),
            timestamp=datetime.utcnow(),
            action=action,
            entity_type=entity_type,
            entity_id=entity_id,
            user_id=user.id,
            user_role=user.role,
            ip_address=get_client_ip(),
            user_agent=get_user_agent(),
            old_value=old_value,
            new_value=new_value,
            reason=reason,
            study_id=user.current_study_id,
            site_id=user.current_site_id,
        )
        self._write_immutable(entry)
```

### Export Audit Requirements
```python
@dataclass
class DataExportRequest:
    """Track all PHI exports for compliance."""
    request_id: str
    requester_id: str
    purpose: str              # research, clinical, audit, legal
    data_types: list[str]     # ["demographics", "assessments", "notes"]
    patient_ids: list[str] | None  # None = all patients in scope
    date_range: tuple[date, date] | None
    approved_by: str | None
    approved_at: datetime | None
    exported_at: datetime | None
    record_count: int | None
    file_hash: str | None     # SHA256 of exported file

# Require approval for bulk exports
def request_export(request: DataExportRequest) -> str:
    """Submit export request for approval."""
    if request.record_count > 100:
        queue_for_approval(request)
    else:
        auto_approve_and_log(request)
    return request.request_id
```

---

## PHI Handling Patterns

For HIPAA/PIPEDA compliance, protect Protected Health Information.

### Pattern
```python
from enum import Enum

class PHICategory(str, Enum):
    """HIPAA Safe Harbor 18 identifiers."""
    NAME = "name"
    GEOGRAPHIC = "geographic"           # Below state level
    DATES = "dates"                     # Except year
    PHONE = "phone"
    FAX = "fax"
    EMAIL = "email"
    SSN = "ssn"
    MRN = "mrn"
    HEALTH_PLAN = "health_plan_id"
    ACCOUNT = "account_number"
    LICENSE = "license_number"
    VEHICLE = "vehicle_id"
    DEVICE = "device_id"
    URL = "url"
    IP_ADDRESS = "ip_address"
    BIOMETRIC = "biometric"
    PHOTO = "photo"
    UNIQUE_ID = "unique_identifier"

@dataclass
class PHIField:
    """Mark fields as containing PHI."""
    field_name: str
    phi_category: PHICategory
    requires_consent: bool = True
    can_be_derived: bool = False    # Can be re-identified from other data

# Field-level PHI metadata
PHI_FIELDS = {
    "patient.first_name": PHIField("first_name", PHICategory.NAME),
    "patient.last_name": PHIField("last_name", PHICategory.NAME),
    "patient.dob": PHIField("dob", PHICategory.DATES),
    "patient.mrn": PHIField("mrn", PHICategory.MRN),
    "patient.phone": PHIField("phone", PHICategory.PHONE),
    "patient.email": PHIField("email", PHICategory.EMAIL),
    "patient.address": PHIField("address", PHICategory.GEOGRAPHIC),
}
```

### De-identification Strategies
```python
class DeidentificationStrategy(str, Enum):
    REDACT = "redact"           # Replace with [REDACTED]
    MASK = "mask"               # Replace with ***
    HASH = "hash"               # One-way hash (reversible with key)
    GENERALIZE = "generalize"   # Age 34 → "30-39"
    DATE_SHIFT = "date_shift"   # Shift all dates by random offset
    REMOVE = "remove"           # Delete field entirely

@dataclass
class DeidentificationConfig:
    """Per-field de-identification strategy."""
    strategy: DeidentificationStrategy
    preserve_format: bool = False     # Keep length/format
    preserve_referential: bool = True # Same input → same output

# Apply consistent date shifting per patient
def date_shift_patient(
    patient_id: str,
    dates: list[date],
    shift_range_days: int = 365,
) -> list[date]:
    """Shift all dates for a patient by consistent offset."""
    # Deterministic offset from patient ID
    offset = hash(f"{patient_id}:date_shift") % shift_range_days
    return [d + timedelta(days=offset) for d in dates]
```

### Minimum Necessary Principle
```python
class DataAccessLevel(str, Enum):
    """Role-based data visibility."""
    FULL_PHI = "full_phi"           # Clinical care, authorized research
    LIMITED = "limited"              # Dates generalized, names masked
    DEIDENTIFIED = "deidentified"   # Safe Harbor compliant
    AGGREGATE = "aggregate"          # Statistics only, no individual records

def filter_for_access_level(
    data: dict,
    access_level: DataAccessLevel,
) -> dict:
    """Apply minimum necessary filtering."""
    if access_level == DataAccessLevel.FULL_PHI:
        return data
    elif access_level == DataAccessLevel.DEIDENTIFIED:
        return apply_safe_harbor(data)
    elif access_level == DataAccessLevel.AGGREGATE:
        raise ValueError("Use aggregate queries for AGGREGATE level")
    else:
        return apply_limited_access(data)
```

---

## Adjudication Workflow

For clinical extraction quality, implement structured human review.

### Pattern
```python
from enum import Enum

class AdjudicationStatus(str, Enum):
    PENDING = "pending"
    IN_PROGRESS = "in_progress"
    ADJUDICATED = "adjudicated"
    ESCALATED = "escalated"
    CONSENSUS_REQUIRED = "consensus_required"

class AdjudicationDecision(str, Enum):
    ACCEPT = "accept"           # AI extraction is correct
    REJECT = "reject"           # AI extraction is wrong
    MODIFY = "modify"           # Partially correct, needs changes
    CANNOT_DETERMINE = "cannot_determine"

@dataclass
class AdjudicationTask:
    """Track human review of AI extractions."""
    task_id: str
    extraction_id: str
    document_id: str
    field_path: str             # "medications[0].name"
    ai_value: Any
    ai_confidence: float
    verification_issues: list[str]

    status: AdjudicationStatus
    assigned_to: str | None
    assigned_at: datetime | None

    decision: AdjudicationDecision | None
    corrected_value: Any | None
    rejection_reason: str | None
    adjudicator_notes: str | None
    adjudicated_at: datetime | None
    adjudicated_by: str | None

    # For inter-rater reliability
    secondary_adjudicator: str | None
    secondary_decision: AdjudicationDecision | None
    requires_consensus: bool
```

### Rejection Reason Taxonomy
```python
class RejectionReason(str, Enum):
    """Structured reasons for extraction rejection."""
    # Evidence issues
    EVIDENCE_MISSING = "evidence_missing"
    EVIDENCE_MISQUOTED = "evidence_misquoted"
    EVIDENCE_OUT_OF_CONTEXT = "evidence_out_of_context"

    # Interpretation issues
    NEGATION_MISSED = "negation_missed"
    UNCERTAINTY_MISSED = "uncertainty_missed"
    TEMPORAL_ERROR = "temporal_error"
    HISTORICAL_AS_CURRENT = "historical_as_current"

    # Value issues
    WRONG_VALUE = "wrong_value"
    WRONG_UNIT = "wrong_unit"
    WRONG_ENTITY = "wrong_entity"
    HALLUCINATION = "hallucination"

    # Context issues
    WRONG_PATIENT = "wrong_patient"
    WRONG_ENCOUNTER = "wrong_encounter"
    FAMILY_HISTORY_CONFUSED = "family_history_confused"

@dataclass
class RejectionDetails:
    reason: RejectionReason
    explanation: str
    correct_value: Any | None
    evidence_span: str | None   # What the correct evidence is
```

### Inter-Rater Reliability
```python
@dataclass
class AdjudicationMetrics:
    """Track adjudication quality and consistency."""
    field_type: str
    total_tasks: int
    agreement_rate: float        # Primary-secondary agreement
    cohens_kappa: float          # Chance-adjusted agreement
    avg_time_to_adjudicate: float
    escalation_rate: float

def calculate_inter_rater_reliability(
    primary_decisions: list[AdjudicationDecision],
    secondary_decisions: list[AdjudicationDecision],
) -> float:
    """Calculate Cohen's Kappa for adjudicator agreement."""
    # Implementation of Cohen's Kappa
    observed_agreement = sum(
        1 for p, s in zip(primary_decisions, secondary_decisions) if p == s
    ) / len(primary_decisions)

    # Calculate expected agreement by chance
    # ...

    return (observed_agreement - expected_agreement) / (1 - expected_agreement)
```

---

## Multi-Site and Multi-Tenant Patterns

For multi-center research studies, isolate data by site with cross-site aggregation.

### Pattern
```python
@dataclass
class Site:
    """Research site in a multi-center study."""
    site_id: str
    site_code: str            # Short code: "TOR01", "VAN02"
    site_name: str
    institution_name: str
    country: str
    timezone: str
    is_active: bool
    activated_at: date | None
    deactivated_at: date | None

@dataclass
class Study:
    """Multi-site clinical study."""
    study_id: str
    protocol_number: str
    title: str
    sponsor: str
    sites: list[Site]
    data_isolation: str       # "strict" | "aggregate_visible"

@dataclass
class UserSiteRole:
    """User role scoped to specific site(s)."""
    user_id: str
    study_id: str
    site_id: str | None       # None = study-level access
    role: str                 # site_coordinator, investigator, sponsor
    permissions: list[str]

# Site-scoped queries
class SiteAwareRepository:
    def __init__(self, current_user: User):
        self.user = current_user
        self.allowed_sites = self._get_allowed_sites()

    def get_patients(self, filters: dict) -> list[Patient]:
        """Automatically filter to allowed sites."""
        base_query = Patient.query.filter_by(**filters)

        if not self.user.has_permission("view_all_sites"):
            base_query = base_query.filter(
                Patient.site_id.in_(self.allowed_sites)
            )

        return base_query.all()
```

### Cross-Site Data Aggregation
```python
@dataclass
class CrossSiteAggregate:
    """Aggregated metrics without individual patient data."""
    study_id: str
    metric_name: str
    aggregation_type: str     # count, mean, median, sum
    site_values: dict[str, float]  # site_id → value
    total_value: float
    computed_at: datetime

# Aggregate without exposing individual records
def compute_cross_site_enrollment(study_id: str) -> CrossSiteAggregate:
    """Compute enrollment by site without patient-level data."""
    site_counts = db.session.query(
        Patient.site_id,
        func.count(Patient.id),
    ).filter(
        Patient.study_id == study_id,
    ).group_by(
        Patient.site_id,
    ).all()

    return CrossSiteAggregate(
        study_id=study_id,
        metric_name="enrollment_count",
        aggregation_type="count",
        site_values={site_id: count for site_id, count in site_counts},
        total_value=sum(c for _, c in site_counts),
        computed_at=datetime.utcnow(),
    )
```

### Site-Specific Configuration
```python
@dataclass
class SiteConfiguration:
    """Per-site customization within a study."""
    site_id: str
    study_id: str

    # IRB/REB details
    irb_name: str
    irb_approval_number: str
    irb_approval_date: date
    irb_expiry_date: date

    # Local consent form version
    consent_form_id: str
    consent_language: str

    # Site-specific thresholds
    enrollment_target: int
    max_concurrent_participants: int

    # Integration settings
    local_ehr_integration: bool
    ehr_system_type: str | None
    redcap_project_id: str | None

# Check site-specific requirements
def validate_enrollment(patient: Patient, site: Site) -> list[str]:
    """Validate against site-specific requirements."""
    errors = []
    config = get_site_config(site.site_id, patient.study_id)

    if config.irb_expiry_date < date.today():
        errors.append("Site IRB approval has expired")

    current_count = count_active_participants(site.site_id)
    if current_count >= config.max_concurrent_participants:
        errors.append("Site has reached maximum concurrent participants")

    return errors
```

---

## Role-Based Access Control

For multi-site research, implement hierarchical RBAC.

### Pattern
```python
from enum import Enum

class Permission(str, Enum):
    """Granular permissions for clinical research."""
    # Patient data
    VIEW_PATIENT_PHI = "view_patient_phi"
    VIEW_PATIENT_DEIDENTIFIED = "view_patient_deidentified"
    EDIT_PATIENT = "edit_patient"
    DELETE_PATIENT = "delete_patient"

    # Assessments
    VIEW_ASSESSMENTS = "view_assessments"
    ADMINISTER_ASSESSMENTS = "administer_assessments"
    ADJUDICATE_EXTRACTIONS = "adjudicate_extractions"

    # Study management
    MANAGE_SITE = "manage_site"
    MANAGE_STUDY = "manage_study"
    VIEW_ALL_SITES = "view_all_sites"
    EXPORT_DATA = "export_data"

    # System
    MANAGE_USERS = "manage_users"
    VIEW_AUDIT_LOGS = "view_audit_logs"

# Role definitions
ROLES = {
    "site_coordinator": [
        Permission.VIEW_PATIENT_PHI,
        Permission.EDIT_PATIENT,
        Permission.VIEW_ASSESSMENTS,
        Permission.ADMINISTER_ASSESSMENTS,
        Permission.MANAGE_SITE,
    ],
    "investigator": [
        Permission.VIEW_PATIENT_PHI,
        Permission.VIEW_ASSESSMENTS,
        Permission.ADJUDICATE_EXTRACTIONS,
    ],
    "research_assistant": [
        Permission.VIEW_PATIENT_DEIDENTIFIED,
        Permission.VIEW_ASSESSMENTS,
        Permission.ADMINISTER_ASSESSMENTS,
    ],
    "sponsor": [
        Permission.VIEW_PATIENT_DEIDENTIFIED,
        Permission.VIEW_ALL_SITES,
        Permission.EXPORT_DATA,
    ],
    "data_manager": [
        Permission.VIEW_PATIENT_PHI,
        Permission.VIEW_ALL_SITES,
        Permission.EXPORT_DATA,
        Permission.VIEW_AUDIT_LOGS,
    ],
}

# Check permissions with site scope
def has_permission(
    user: User,
    permission: Permission,
    site_id: str | None = None,
) -> bool:
    """Check if user has permission, optionally scoped to site."""
    user_roles = get_user_roles(user.id, site_id)

    for role in user_roles:
        if permission in ROLES.get(role.role, []):
            return True

    return False
```

### Hierarchical Organization Units
```python
@dataclass
class OrganizationUnit:
    """Hierarchical org structure for RBAC."""
    unit_id: str
    parent_id: str | None
    unit_type: str            # institution, department, team
    name: str

# Closure table for efficient hierarchy queries
# (See tech-stack-patterns for Catalyst implementation)
class OrgUnitClosure:
    """Pre-computed ancestor-descendant relationships."""
    ancestor_id: str
    descendant_id: str
    depth: int

# Check access at any level of hierarchy
def can_access_patient(user: User, patient: Patient) -> bool:
    """Check if user's org unit contains patient's site."""
    user_units = get_user_org_units(user.id)
    patient_unit = get_site_org_unit(patient.site_id)

    # User can access if any of their units is an ancestor
    return any(
        is_ancestor(unit.unit_id, patient_unit)
        for unit in user_units
    )
```

---

## When to Use These Patterns

Apply clinical patterns when working with:
- Electronic health record (EHR) data
- Clinical trial data management
- Synthetic patient generation
- Healthcare analytics pipelines
- Research data repositories
- Regulatory compliance systems (HIPAA, GCP, ICH-GCP)
- Clinical information extraction (NLP/NER)
- Cognitive and behavioral assessments
- Multi-site research studies
- REDCap and other EDC integrations
