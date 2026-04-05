# TrustLayer: Product Requirements Document

**Product Name**: TrustLayer
**Version**: 1.0 (Phase 1 MVP)
**Author**: Senior Technical Product Manager, Clinical AI Safety
**Status**: Draft for review
**Last Updated**: Q1 2026

---

## Problem Statement

Healthcare AI systems are generating clinical content (patient messages, note summaries, diagnostic suggestions, prior authorization letters) that enters workflows affecting patient care. These outputs are not independently validated for clinical safety, factual grounding, or PHI protection before delivery. Healthcare organizations have no dedicated infrastructure to catch hallucinated clinical content, prevent PHI leakage, block unsafe medical guidance, or produce the audit evidence that regulators expect. The result is that healthcare AI outputs reach patients and clinicians with no safety controls beyond what the model itself provides.

## Users

**Primary users**:
- Clinical informatics engineers who integrate AI tools into clinical workflows
- Safety policy authors (clinical informatics, compliance, privacy)
- Clinician reviewers who evaluate escalated AI outputs

**Secondary users**:
- Compliance and privacy officers who audit AI safety and governance
- Patient safety officers who investigate AI-related incidents

**Buyer personas**:
- CMIO: accountable for clinical AI quality and safety
- CISO: accountable for PHI protection and security posture
- CIO / VP Digital Health: accountable for AI platform infrastructure

## Jobs to Be Done

1. **Clinical informatics engineer**: "I need to integrate safety checks into our AI pipeline without rebuilding our patient messaging tool. I need a service I can call between model output and delivery."
2. **Compliance officer**: "I need to demonstrate to an auditor that every AI-generated clinical output was checked for safety and privacy before it reached a patient or clinician. I need the evidence trail."
3. **Clinician reviewer**: "When the system escalates an AI-generated response to me, I need to see the patient's chart context, the specific safety concern, and a risk score so I can decide quickly. I do not want a queue full of false alarms."
4. **CMIO**: "I need to show the medical executive committee that our clinical AI tools are governed. I need data on what gets caught, what gets through, and what the safety posture looks like."
5. **Privacy officer**: "I need assurance that AI-generated text does not leak PHI into unauthorized contexts. If it does, I need to know about it immediately and have a complete trace."

## Goals

1. Intercept and evaluate every AI-generated clinical output before it reaches a clinical endpoint.
2. Detect hallucinated clinical content with a proposed interception rate above 99%.
3. Prevent PHI leakage in AI-generated outputs with a zero-tolerance target.
4. Block clinically unsafe outputs (harmful medication advice, contraindicated treatments, out-of-scope guidance) before delivery.
5. Provide a complete, immutable audit trail for every AI output and every safety decision.
6. Maintain additional safety pipeline latency below 500ms per output.

## Non-Goals

1. TrustLayer is not a clinical AI model. It does not generate clinical content.
2. TrustLayer does not replace clinician judgment. It augments clinical oversight by catching machine-detectable safety issues.
3. TrustLayer does not validate clinical accuracy (whether a diagnosis is correct). It validates clinical safety (whether an output could cause harm).
4. TrustLayer does not provide real-time clinical dashboards in Phase 1. Dashboards are Phase 2.
5. TrustLayer does not manage model training, fine-tuning, or prompt engineering.

## User Stories

### Clinical Informatics Engineer
- As an informatics engineer, I want to send AI-generated outputs to TrustLayer via API and receive a safety disposition (approved / escalated / blocked) so that I can route outputs accordingly in my pipeline.
- As an informatics engineer, I want to configure which safety checks run for a specific workflow so that the pipeline matches the risk profile of the use case.
- As an informatics engineer, I want to run TrustLayer in shadow mode so that I can evaluate detection accuracy without affecting the live clinical workflow.

### Clinician Reviewer
- As a clinician reviewer, I want to see escalated outputs with the patient's relevant chart context, the specific safety finding (e.g., "ungrounded medication claim: metformin not found in medication list"), and the risk score so that I can make a fast decision.
- As a clinician reviewer, I want to approve, modify, or reject an escalated output and have my decision and rationale logged.
- As a clinician reviewer, I want the escalation queue to have manageable volume. If I am spending more than 15 minutes per hour on AI reviews, the system is escalating too aggressively.

### Compliance / Privacy Officer
- As a compliance officer, I want to query the audit log for any time period, workflow, or safety check type so that I can produce compliance evidence for auditors.
- As a privacy officer, I want to receive an immediate alert if PHI is detected in an AI output that was not properly redacted.
- As a compliance officer, I want to see aggregate safety metrics (interception rates, escalation rates, block rates) so that I can report governance posture to leadership.

### CMIO
- As a CMIO, I want a kill switch that immediately disables all AI output delivery across the organization so that I can respond to safety incidents without waiting for engineering.

## Functional Requirements

### FR-01: Grounding Checks
The system must verify that clinical claims in AI-generated outputs are traceable to provided source documents (patient records, clinical guidelines, formulary data). Each claim must be classified as grounded (traceable to source), partially grounded (related source exists but claim extends beyond it), or ungrounded (no supporting source found). Grounding results must be included in the safety trace.

### FR-02: Hallucination Detection
The system must identify clinical assertions in AI outputs that are not supported by the provided evidence. This includes fabricated medications, invented lab values, fabricated clinical events, and clinical facts that contradict the source material. Detection must operate on clinical text, handling medical terminology, abbreviations, and structured/unstructured clinical data. Proposed interception target: > 99% of hallucinated clinical claims.

### FR-03: PHI Detection and Redaction
The system must scan AI-generated outputs for all 18 HIPAA identifier categories. Detection must be tuned for clinical text patterns (MRNs, provider NPIs, dates of service in clinical formatting, facility names). PHI that should not appear in the output context must be redacted or must trigger a block. Proposed target: zero PHI leakage incidents in unauthorized contexts.

### FR-04: Clinical Safety Classification
The system must evaluate AI outputs for potential clinical harm. Categories include: unsafe medication recommendations (wrong drug, wrong dose, contraindicated), inappropriate diagnostic suggestions, treatment advice outside the AI system's intended scope, and guidance that could delay appropriate care. Classification must use a combination of rule-based checks (drug interaction databases, dosage range validation) and a trained safety classifier. Outputs classified as clinically unsafe must be blocked.

### FR-05: Policy Engine
The system must support configurable governance policies per workflow, per department, and per AI model. Policies must be expressible as rules (e.g., "patient-facing responses must not include specific dosage numbers," "responses exceeding 500 words require review," "all oncology-related outputs require clinician review regardless of safety score"). In Phase 1, policies are managed via configuration by admins. Self-service policy management is Phase 2.

### FR-06: Clinician Escalation
The system must route outputs with ambiguous or moderate-risk safety findings to a clinician review queue. Each escalated item must include: the AI-generated output, the triggering safety finding, the risk score, the patient's relevant chart context, and a recommended disposition. Clinicians must be able to approve, modify, or reject. SLA tracking must be supported.

### FR-07: Output Blocking
The system must prevent delivery of outputs that fail critical safety checks (PHI leakage, clinically unsafe content, policy violations). Blocked outputs must generate an incident record with the full safety trace. The requesting system must receive a clear signal that the output was blocked, with a categorized reason.

### FR-08: Audit Logging
The system must create an immutable, append-only audit record for every AI output processed. Each record must include: timestamp, source AI model, workflow context, raw model output, each safety check result, composite risk score, routing decision, policy version, reviewer identity and decision (if escalated), and final disposition. Records must be queryable and exportable. Recommended retention: 7 years (aligned with healthcare record retention requirements).

### FR-09: Traceability
Every safety decision must be traceable to a specific policy version, safety check version, and (if applicable) evidence source. When a stakeholder asks "why was this output blocked?" or "why was this output approved?", the system must provide a complete, deterministic answer.

### FR-10: Role-Based Access
The system must enforce role-based access:
- **Admin**: Full system configuration, policy management, kill switch access
- **Policy Author**: Create and modify safety policies, view audit logs
- **Clinician Reviewer**: View and act on escalation queue, view related audit entries
- **Viewer**: Read-only access to dashboards and audit logs (dashboards in Phase 2)
- **Pipeline**: API access to submit outputs and receive dispositions

### FR-11: Kill Switch
The system must provide an emergency kill switch accessible to Admin and CMIO roles. Activation must immediately halt all AI output delivery. The kill switch must operate independently of the safety pipeline application layer (infrastructure-level circuit breaker). Activation and deactivation must be logged.

### FR-12: Configurable Safety Policies
The system must support workflow-specific safety configurations: which checks are active, threshold sensitivity per check, escalation vs. block behavior, and scope-of-practice boundaries. This enables different safety profiles for patient messaging (higher sensitivity) vs. internal clinical documentation (moderate sensitivity) vs. administrative AI tasks (lower sensitivity).

## Non-Functional Requirements

### NFR-01: Performance
- Safety pipeline latency: proposed target < 500ms at p99 per output
- Clinician queue load time: < 1 second
- Audit log query response: < 3 seconds for standard queries

### NFR-02: Availability
- Target 99.95% uptime for the safety pipeline service
- Graceful degradation: if TrustLayer is unreachable, AI outputs are queued (not delivered) until service recovers

### NFR-03: Scalability
- Support 5,000 output evaluations per hour at launch
- Architecture must be horizontally scalable to 50,000+ evaluations per hour

### NFR-04: Security
- All data encrypted at rest (AES-256) and in transit (TLS 1.3)
- PHI handling must comply with HIPAA Technical Safeguard requirements
- Audit logs cryptographically signed
- BAA (Business Associate Agreement) compliant infrastructure

### NFR-05: Clinical Text Handling
- Safety checks must handle clinical abbreviations, medical terminology, structured data (lab values, vital signs), and unstructured narrative text
- Must support common clinical document formats (HL7 FHIR, CDA, plain text)

## Dependencies

- Access to patient chart context (medication lists, problem lists, recent notes) for grounding verification. This requires EHR integration or a clinical data API.
- Drug interaction and dosage range databases (e.g., First Databank, Medi-Span) for medication safety checking.
- Clinical knowledge bases for evidence-based grounding (clinical practice guidelines, formulary data).
- SSO/SAML integration for clinician reviewer authentication.

## Assumptions

1. AI systems generate outputs in text form (not images or structured orders). Image and structured-data safety are out of scope for Phase 1.
2. Healthcare organizations deploying AI have existing clinical data infrastructure (EHR, clinical data APIs) that can provide patient context for grounding checks.
3. Clinician reviewers are available during clinical hours. After-hours escalation policies will need to be defined per organization.
4. AI pipeline teams are willing to integrate a safety service call between model output and delivery. This is a lightweight API integration.

## Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Hallucination detection misses a clinically significant fabrication | Critical | Medium | Multi-method detection (entailment + knowledge base); conservative initial thresholds; mandatory shadow mode |
| PHI scanner misses healthcare-specific identifiers | Critical | Low | Healthcare-tuned entity recognition; red-team testing with clinical text samples |
| False-positive rate degrades clinician trust | High | High | Precision as a primary metric; clinician feedback loop; threshold calibration per workflow |
| EHR integration for patient context is complex/slow | High | Medium | Start with API-based context passing; defer deep EHR integration to Phase 3 |
| Clinical safety classifier does not generalize across specialties | High | High | Start with one specialty; invest in specialty-specific training data over time |

## Out of Scope (Phase 1)

- Self-service policy management UI (admin-configured in Phase 1; self-service in Phase 2)
- Real-time safety dashboards (Phase 2)
- Adaptive safety thresholds / ML-based threshold tuning (Phase 2)
- Deep EHR integration (Epic, Oracle Health native connectors) (Phase 3)
- Multi-turn conversation safety analysis
- Image or structured order safety validation
- Cross-organization safety benchmarking

## Success Criteria (Phase 1)

All targets below are proposed benchmarks for initial rollout. They will be calibrated based on pilot data during shadow mode.

| Metric | Proposed Target |
|--------|----------------|
| Safe and compliant response rate | > 95% |
| Hallucination interception rate | > 99% |
| PHI leakage incidents | 0 |
| Clinically unsafe miss rate | < 0.05% |
| Clinician escalation precision | > 80% |
| False-positive block rate | < 10% |
| Safety pipeline latency | < 500ms at p99 |
| Audit log coverage | 100% |
| Clinician review SLA compliance | > 90% |
