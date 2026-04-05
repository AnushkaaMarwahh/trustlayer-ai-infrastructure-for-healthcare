# TrustLayer: Metrics Framework

---

## North Star Metric

**Safe and Compliant Response Rate**: The percentage of healthcare AI outputs that are both clinically safe and policy-compliant before they reach any clinical endpoint (patient portal, EHR, clinician workflow, or downstream system).

This metric captures the core value proposition. If TrustLayer is working, the vast majority of AI outputs that reach clinical endpoints have passed independent safety validation. The remainder are caught and handled through escalation or blocking.

**Proposed target**: > 95% at launch, trending toward > 98% at steady state. These numbers mean that 95%+ of outputs are safe and compliant on first pass through the pipeline (no escalation or blocking required). The remaining outputs are caught and handled, not delivered unvalidated.

**Measurement method**: Automated pipeline results provide the primary signal. Weekly expert review of a stratified sample (50 approved, 30 escalated, 20 blocked outputs) by a clinical informatics analyst validates pipeline accuracy. Discrepancies between pipeline decisions and expert judgment are counted against the metric.

---

## Supporting Metrics

These decompose the north star into measurable components. All targets are proposed operating benchmarks for initial rollout and will be calibrated from pilot data.

### Grounded Response Rate
**Definition**: Percentage of clinical claims in approved AI outputs that are traceable to provided source evidence (patient records, clinical guidelines, formulary data).
**Proposed target**: > 95%
**Why it matters**: Ungrounded claims are the precursor to hallucination-related harm. Even if a claim happens to be correct, if it cannot be traced to evidence, it is not verifiable in a clinical context.
**Owner**: Clinical AI Engineering

### Hallucination Interception Rate
**Definition**: Percentage of hallucinated clinical assertions (fabricated medications, invented lab values, contradicted clinical facts) that are caught by the safety pipeline before delivery.
**Proposed target**: > 99%
**Why it matters**: This is the most critical safety metric. Every hallucination that reaches a patient or clinician is a potential harm event and an audit failure.
**Owner**: Clinical AI Engineering + Clinical Informatics

### PHI Leakage Prevention Rate
**Definition**: Percentage of AI outputs containing PHI in unauthorized contexts that are caught and redacted or blocked before delivery.
**Proposed target**: 100% (zero-tolerance guardrail)
**Why it matters**: PHI leakage is a HIPAA violation with financial penalties, reputational damage, and patient trust erosion. There is no acceptable non-zero rate for this in healthcare.
**Owner**: Privacy Engineering + Compliance

### Unsafe Output Block Rate
**Definition**: Percentage of clinically unsafe outputs (harmful medication advice, contraindicated treatments, out-of-scope guidance) that are blocked before delivery.
**Proposed target**: > 99.5%
**Why it matters**: Unsafe clinical guidance that reaches a patient can directly cause harm. The block rate must be extremely high, with the understanding that a small number of edge cases may reach the clinician escalation path rather than outright blocking.
**Owner**: Clinical Safety Engineering

### Clinician Escalation Precision
**Definition**: Percentage of escalated outputs where the clinician reviewer agrees that human judgment was warranted (the output was genuinely ambiguous, risky, or required clinical context the system could not evaluate).
**Proposed target**: > 80%
**Why it matters**: If escalation precision drops below 80%, clinicians are spending too much time reviewing false alarms. This erodes trust, slows workflows, and creates pressure to disable the safety layer. In healthcare, clinician time is expensive and scarce. Wasting it on false positives has a direct opportunity cost.
**Owner**: Clinical Informatics + Product

### Safe Response Latency
**Definition**: Additional latency added by the TrustLayer safety pipeline per output, measured at p99.
**Proposed target**: < 500ms
**Why it matters**: Healthcare AI workflows are not latency-sensitive at the millisecond level, but adding multiple seconds of delay degrades user experience and can create workflow friction. 500ms is the threshold at which the safety check is effectively invisible to the end user in most clinical workflows.
**Owner**: Platform Engineering

---

## Guardrail Metrics

These are thresholds that should not be crossed. Breaching a guardrail triggers immediate investigation.

### Clinically Unsafe Miss Rate
**Definition**: Percentage of clinically unsafe outputs that pass through the safety pipeline and reach a clinical endpoint without being caught.
**Recommended threshold**: < 0.05%
**Breach response**: Immediate tightening of safety classifier thresholds. Root cause analysis within 24 hours. If the miss involved patient harm, escalation to the patient safety committee.

### PHI Incident Rate
**Definition**: Number of PHI leakage events in AI-generated outputs that reach unauthorized contexts.
**Recommended threshold**: 0 (zero tolerance)
**Breach response**: Immediate kill switch consideration. Root cause analysis within 12 hours. Privacy incident report per HIPAA breach notification requirements.

### Audit Coverage
**Definition**: Percentage of AI outputs processed by TrustLayer with a complete audit record.
**Recommended threshold**: 100%
**Breach response**: Immediate engineering investigation. Any gap in audit coverage is a regulatory compliance failure.

### False-Positive Block Rate
**Definition**: Percentage of blocked or escalated outputs where subsequent review determines the output was actually safe and compliant.
**Recommended threshold**: < 10%
**Signal**: Rising false-positive rates indicate safety checks are miscalibrated. This does not cause patient harm directly, but it degrades system utility and clinician trust.

---

## Operational Metrics

### Clinician Override Rate
**Definition**: Percentage of escalated outputs where the clinician reviewer overrides TrustLayer's recommended disposition (e.g., approves an output flagged for blocking, or rejects an output recommended for approval).
**Target**: No fixed target. Monitored as a trend signal.
**Signal**: A rising override rate suggests the safety pipeline and clinician judgment are diverging. This could mean the safety checks need calibration, or that clinicians need more context on what the safety flags mean. Either way, it requires investigation.

### Review Queue Depth
**Definition**: Number of pending items in the clinician escalation queue at any point in time.
**Target**: No fixed target. Monitored for bottleneck detection.
**Signal**: Queue depth consistently exceeding 30 minutes of review backlog suggests either escalation volume is too high (precision problem) or reviewer capacity is insufficient (staffing problem).

### Policy Rule Coverage
**Definition**: Percentage of observed output types and clinical workflows that are covered by explicit safety policies.
**Proposed target**: > 90% of output volume covered by explicit policies.
**Why**: Outputs not covered by explicit policies fall through to default safety checks, which may be less appropriate for the specific clinical context.

### Safety Check Pass-Through Distribution
**Definition**: Distribution of outputs across the three dispositions (approved, escalated, blocked) per workflow.
**Target**: No fixed target. Monitored for balance.
**Signal**: If more than 20% of outputs for a given workflow are being escalated or blocked, the safety profile for that workflow may need calibration. If less than 1% are being escalated or blocked, the safety checks may be too permissive.

### System Uptime
**Definition**: Percentage of time TrustLayer is available to process outputs.
**Proposed target**: 99.95%
**Why**: TrustLayer is in the critical path. If it is down, AI outputs are queued, not delivered. Extended downtime means clinical AI tools are effectively disabled.

---

## Failure Indicators

These signals suggest systemic problems that warrant leadership escalation.

| Indicator | What It Suggests |
|-----------|------------------|
| Unsafe miss rate exceeds 0.1% | Safety pipeline is fundamentally missing a category of unsafe outputs |
| Any PHI leakage incident | PHI detection has a gap; immediate investigation and potential system pause |
| Escalation precision drops below 60% | Safety checks are over-aggressive; clinician trust erosion is likely |
| False-positive block rate exceeds 20% | Safety pipeline is blocking too many safe outputs; adoption risk |
| Override rate exceeds 30% | Pipeline and clinician judgment have diverged significantly; recalibration needed |
| Queue depth exceeds 2 hours of backlog | Review capacity cannot keep up; operational model is unsustainable |
| Audit coverage drops below 100% | Logging failure; regulatory exposure is immediate |
| Safety latency exceeds 2 seconds | Performance degradation will cause workflow abandonment |

---

## Measurement Cadence

| Metric Category | Review Frequency | Owner |
|-----------------|-----------------|-------|
| North star | Weekly | Head of Product |
| Supporting metrics | Weekly | Product + Clinical Informatics |
| Guardrail metrics | Real-time monitoring with alerting | Engineering + Compliance |
| Operational metrics | Daily (automated) | Engineering |
| Failure indicators | Triggered (alert-based) | Incident Response Team |

---

## Dashboard Requirements (Phase 2)

Phase 1 metrics are available through audit log queries and weekly reports. Phase 2 introduces a safety dashboard:

- North star trend (weekly, monthly)
- Disposition distribution per workflow (approved / escalated / blocked)
- Hallucination interception rate trend
- PHI detection event log
- Escalation precision and queue depth
- Safety check latency percentiles
- Override rate trend with drill-down
- Guardrail breach alert panel
