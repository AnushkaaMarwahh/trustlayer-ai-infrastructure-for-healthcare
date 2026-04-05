# TrustLayer: Risk and Governance Framework

---

## Key Failure Modes

Healthcare AI safety failures have consequences that do not exist in other domains. A false positive in an e-commerce recommendation engine is an inconvenience. A false negative in a clinical safety layer can cause patient harm. This document catalogs TrustLayer's specific failure modes, defines governance principles grounded in healthcare safety culture, and establishes the operational framework for managing clinical AI risk.

---

### 1. Hallucinated Clinical Advice Reaches a Patient or Clinician

**What happens**: The hallucination detector fails to catch a fabricated clinical assertion in an AI output. The output is approved and delivered. A clinician or patient acts on information that is not grounded in the patient's actual medical record or in clinical evidence.

**Real-world example pattern**: An AI drafts a patient message that states "your current prescription for lisinopril 20mg should be continued." The patient is not prescribed lisinopril. The patient, trusting the message, does not mention the discrepancy to their provider. At the next visit, confusion about the medication list creates a prescribing error.

**Root causes**:
- Hallucination detector has a blind spot for this type of clinical assertion (e.g., medication mentions embedded in conversational text rather than in a list format)
- Grounding check did not have access to the complete medication list (missing context)
- The fabricated claim was structurally plausible and the model expressed it with high confidence, which makes it harder for entailment-based detection to flag

**Impact**: Critical. Direct potential for patient harm.

**Mitigations**:
- Multi-method hallucination detection (entailment checking + knowledge base cross-reference)
- Conservative initial detection thresholds during shadow mode
- Weekly sample audit of approved outputs by clinical informatics analyst
- Automated alerting if a downstream system (e.g., medication reconciliation) detects a discrepancy that the safety pipeline missed

### 2. PHI Leakage in AI-Generated Output

**What happens**: An AI-generated output contains protected health information in a context where it should not appear. Examples: a patient's name or MRN appears in a response routed to the wrong patient. Diagnosis information surfaces in a summary that should have been de-identified. A provider's NPI appears in a patient-facing message.

**Root causes**:
- PHI scanner misses healthcare-specific identifier patterns (MRNs in non-standard formats, dates of service embedded in narrative text, facility names that double as patient identifiers)
- AI model surfaces PHI from its context window that should not have been included in the output
- Redaction logic fails to remove all instances (e.g., catches the first mention of a patient name but misses a reference later in the text)

**Impact**: Critical. HIPAA violation with financial penalties (up to $50K per violation, $1.5M annual cap per category), reputational damage, and patient trust erosion. Breach notification requirements may apply.

**Mitigations**:
- Healthcare-tuned entity recognition (not generic PII detection)
- Zero-tolerance blocking: any PHI detected in unauthorized context triggers a block, not a warning
- Red-team testing with clinical text samples across specialties before deployment
- Regular retraining of PHI detection models as clinical text patterns evolve

### 3. Unsafe Clinical Guidance Not Caught (False Negative)

**What happens**: The clinical safety classifier fails to identify an AI output that contains harmful guidance. Examples: a dosage recommendation outside the safe range for the patient's renal function, a treatment suggestion contraindicated by the patient's medication list, or guidance that could delay appropriate emergency care.

**Root causes**:
- Clinical safety classifier not trained on the specific harm pattern
- Rule-based checks lack coverage for the specific drug, condition, or interaction
- The unsafe guidance is expressed indirectly or conditionally ("you might consider..." rather than "you should take...")
- Patient context is incomplete (missing allergy list, missing renal function labs)

**Impact**: Critical. Direct potential for patient harm. Regulatory liability.

**Mitigations**:
- Combined rule-based + ML-based safety checking (rule-based catches known patterns; ML catches novel patterns)
- Drug interaction and dosage databases updated quarterly
- Conservative "if in doubt, escalate" default for clinical safety findings
- Mandatory shadow mode with clinician validation before enforcement for any new safety check

### 4. Overblocking Safe Outputs (False Positive)

**What happens**: The safety pipeline incorrectly flags a safe, compliant AI output as unsafe and blocks or escalates it. At low rates, this is a minor inconvenience. At high rates, it degrades the value of the AI workflow, wastes clinician reviewer time, and creates organizational pressure to weaken safety controls.

**Root causes**:
- Safety check thresholds set too conservatively for the workflow's actual risk profile
- Grounding checker flags claims that are clinically valid but expressed differently than the source document
- PHI scanner flags clinical terminology as PHI (e.g., a drug name that is also a person's name)
- Policy rules too broad (e.g., "flag all dosage mentions" when the workflow legitimately includes dosage information)

**Impact**: High for adoption. Does not cause direct patient harm, but sustained overblocking leads to clinician distrust, workflow abandonment, and pressure to disable safety controls.

**Mitigations**:
- Escalation precision monitored as a primary metric (proposed target: > 80%)
- False-positive block rate monitored (proposed threshold: < 10%)
- Clinician feedback loop: reviewers flag false positives, which feed into threshold calibration
- Workflow-specific safety profiles rather than one-size-fits-all thresholds

### 5. Clinician Review Bottleneck

**What happens**: The volume of escalated outputs exceeds clinician reviewer capacity. Queue depth grows. SLAs are breached. AI-generated outputs are delayed. Clinicians reviewing the queue experience fatigue and begin approving reflexively, which defeats the purpose of human review.

**Root causes**:
- Escalation rate too high (too many outputs routed to clinician review)
- Reviewer pool too small for the output volume
- Poor queue management: escalated items not prioritized by urgency or specialty
- Workflows with high output volume (e.g., patient messaging at a large health system) overwhelm the review model

**Impact**: High. Bottlenecks negate the value of AI automation. Reviewer fatigue creates a secondary safety risk where clinicians rubber-stamp escalated outputs.

**Mitigations**:
- Escalation precision targeting > 80% to minimize unnecessary escalations
- Queue prioritization by clinical urgency
- SLA enforcement with secondary reviewer escalation
- Volume monitoring with alerts when queue depth exceeds sustainable levels
- Consideration of batched review for lower-risk escalations (e.g., a clinician reviews 10 similar escalations at once rather than one at a time)

### 6. Policy Drift

**What happens**: Safety policies become misaligned with clinical reality. Clinical guidelines change. New medications enter the formulary. New AI workflows are deployed without corresponding safety policies. The safety layer enforces rules that no longer match current clinical practice.

**Root causes**:
- No scheduled policy review cadence
- Clinical guidelines updated without notification to the safety policy team
- New AI workflows deployed without a safety profile
- Drug databases not updated with recent approvals or recalls

**Impact**: Medium. Gradual degradation of safety pipeline relevance. Rising override rates as clinicians disagree with outdated policies.

**Mitigations**:
- Quarterly policy review with clinical advisory board input
- Override rate monitoring as the primary drift signal
- Policy expiration dates (recommended: 6 months, after which policies must be reaffirmed)
- Drug database update schedule aligned with FDA approval cycles
- Automated alerts when new AI workflows are deployed without matching safety policies

### 7. Auditability Gaps

**What happens**: Some AI outputs pass through TrustLayer without a complete audit record. Compliance audits fail because the organization cannot demonstrate governance coverage. A HIPAA inquiry about a specific AI-generated output cannot be answered because the safety trace is incomplete.

**Root causes**:
- Logging infrastructure failure
- AI pipeline bypasses TrustLayer (misconfiguration or intentional workaround)
- Log schema changes that leave older entries without required fields

**Impact**: Critical for compliance. Audit gaps are regulatory failures. In healthcare, the absence of a record is presumed to be a governance failure.

**Mitigations**:
- Audit coverage monitored as a zero-tolerance guardrail (target: 100%)
- Synchronous logging before output delivery (log entry must succeed before output is released)
- Cryptographic signing of audit records
- Daily automated log integrity verification
- Network-level enforcement: AI outputs cannot reach clinical endpoints without passing through TrustLayer

---

## Governance Principles

TrustLayer's governance model reflects healthcare safety culture, not general-purpose software engineering practices.

### Principle 1: Do No Harm by Default
If the safety pipeline cannot confidently determine that an output is safe, the output does not reach the patient or clinician. In clinical contexts, the cost of delivering an unsafe output far exceeds the cost of blocking a safe one. This is the healthcare version of "default deny."

### Principle 2: Evidence-Based Validation
Clinical claims in AI outputs must be traceable to evidence. If a claim cannot be grounded in the patient's record or in clinical guidelines, it is flagged. Unverifiable is not the same as wrong, but in healthcare, unverifiable is not acceptable for delivery without human review.

### Principle 3: Privacy as a Hard Constraint
PHI protection is not a configurable threshold. It is a legal requirement with zero tolerance. Any PHI exposure in an unauthorized context triggers blocking, not a risk score adjustment.

### Principle 4: Clinician Authority
The safety pipeline provides detection, classification, and recommendations. Clinical judgment belongs to the human reviewer. Clinicians can override any safety disposition. Overrides are documented, not prevented.

### Principle 5: Full Traceability
Every output, every safety check, every decision, and every override is recorded in a way that can be retrieved, explained, and presented to a regulator. "What happened?" must always be answerable.

### Principle 6: Graceful Degradation
If TrustLayer is unavailable, AI outputs are queued, not delivered. The system fails closed. Cached safety policies may be used for a limited window (recommended: 2 minutes, which is shorter than the equivalent in non-healthcare contexts because the stakes are higher) before full output suspension.

---

## Escalation Rules

### Clinician Escalation Chain
1. Output enters clinician review queue, assigned to a reviewer matched to the clinical domain.
2. If not reviewed within SLA (recommended defaults: 10 minutes for high-urgency, 2 hours for standard), escalates to secondary reviewer.
3. If secondary reviewer does not act within 50% of original SLA, escalates to the clinical informatics lead.
4. If the escalation remains unresolved after the tertiary SLA, the output is blocked and an incident is created.

Note on SLA design: Healthcare escalation SLAs are tighter than typical enterprise SLAs because the downstream patient or clinician may be waiting for the AI-generated content. A patient message that takes 8 hours to clear review defeats the purpose of AI-generated messaging.

### Blocked Output Investigation
1. Blocked output generates an incident record with the full safety trace.
2. Clinical informatics team reviews blocked outputs daily to identify false positives.
3. If a pattern of false-positive blocks emerges, the relevant safety check or policy is flagged for threshold adjustment.
4. Threshold adjustments require simulation mode testing before enforcement.

### Override Documentation
When a clinician overrides a safety disposition:
1. The override and rationale are logged in the audit trail.
2. Overrides are not prevented or second-guessed in real-time.
3. Aggregate override patterns are reviewed monthly to identify drift.
4. If a specific safety check has an override rate exceeding 25%, it is flagged for calibration review.

---

## Kill Switch Logic

### Activation
- Accessible to Admin and CMIO roles.
- Single-action activation.
- All AI output delivery is immediately halted across all workflows.
- Outputs in the clinician review queue are frozen.
- Outputs mid-delivery are allowed to complete (TrustLayer cannot recall content already displayed in an EHR or patient portal), but no new outputs are released.

### Deactivation
- Same roles can deactivate.
- Deactivation resumes normal processing.
- Frozen queue items re-enter the review queue.
- Post-incident review is mandatory within 24 hours (tighter than non-healthcare equivalents).

### Independence
The kill switch is implemented at the infrastructure level. It operates independently of the safety pipeline application. If the safety pipeline crashes, the kill switch still works. If the database is down, the kill switch still works.

### When to Activate
The kill switch should be activated if:
- A PHI breach is detected and the scope is uncertain
- A patient safety event is linked to an AI output that passed through TrustLayer
- A systemic failure is detected in a safety check (e.g., hallucination detector returning pass for all outputs)
- A regulatory body issues an order to halt AI-generated content

---

## Ownership Model

| Component | Owner | Accountability |
|-----------|-------|---------------|
| Safety pipeline (software) | Clinical AI Engineering | Build, maintain, performance |
| Safety policies (content) | Clinical Informatics + Compliance | Author, review, update policies |
| Hallucination detection model | ML Engineering | Model accuracy, calibration |
| PHI detection model | Privacy Engineering | Detection coverage, zero-leak target |
| Clinical safety classifier | Clinical AI Engineering + Clinical SMEs | Harm detection accuracy |
| Drug interaction / dosage DB | Clinical Informatics | Data currency, update cadence |
| Clinician review operations | Clinical department leadership | Reviewer staffing, SLA compliance |
| Kill switch | CMIO Office | Activation authority, incident response |
| Audit log infrastructure | Platform Engineering | Availability, integrity, retention |
| Compliance reporting | Compliance Team | Report generation, regulatory mapping |

---

## Review Cadence

### Quarterly Safety Review
- All active safety policies reviewed for clinical relevance and calibration.
- Override rate per safety check analyzed; checks with > 25% override rate investigated.
- New clinical guidelines and formulary changes mapped to policy updates.
- Hallucination patterns from the past quarter analyzed for detection blind spots.
- Review output: updated policy set, threshold adjustments, and summary for CMIO.

### Monthly Metrics Review
- North star and supporting metrics reviewed by Product and Clinical Informatics.
- Guardrail breaches reviewed with root cause analysis.
- Escalation volume and precision reviewed; reviewer staffing adjusted if needed.
- Model drift signals analyzed (changes in hallucination rates, safety score distributions).

### Post-Incident Review
- Triggered by: kill switch activation, patient safety event linked to AI output, PHI breach, or regulatory inquiry.
- Completed within 24 hours.
- Deliverable: root cause analysis, immediate mitigation, long-term fix, policy change if needed.
- Findings shared with CMIO, compliance, and clinical informatics leadership.
- For patient safety events: additional reporting through the organization's patient safety committee and adverse event reporting channels.

### Annual Governance Audit
- External or internal audit of the full safety framework.
- Scope: pipeline detection accuracy, policy completeness, audit log integrity, reviewer effectiveness, compliance coverage.
- Output: audit report with findings and remediation plan.
- Aligned with HIPAA annual risk assessment requirements.
