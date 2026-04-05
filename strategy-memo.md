# TrustLayer: Strategy Memo

**To**: VP of Product, Health AI Platform
**From**: Senior Technical Product Manager, Clinical AI Safety
**Date**: Q1 2026
**Re**: Recommended approach for healthcare AI safety infrastructure

---

## Context

We are seeing rapid expansion of AI-generated content in clinical workflows across our customer base. Patient message drafting, clinical note summarization, prior authorization letter generation, and diagnostic support tools are moving from pilot to production. The models are getting meaningfully better at generating useful clinical content.

The problem is that "meaningfully better" is not the same as "reliably safe." We are hearing three consistent signals:

1. **Clinical informatics teams** are reporting hallucination incidents in production AI outputs. The most common pattern is fabricated medication lists, invented lab values, and clinical assertions that do not match the patient's chart. These are not rare events. In one customer's patient messaging pilot, approximately 3 to 5% of AI-generated responses contained at least one ungrounded clinical claim.

2. **Privacy and compliance officers** are raising concerns about PHI handling in AI-generated text. Models trained or prompted with clinical data sometimes surface PHI in contexts where it should not appear: a patient's name in a response to the wrong patient, diagnosis information in a summary that should have been de-identified, MRN numbers embedded in generated text.

3. **Regulatory landscape is shifting.** FDA guidance on clinical decision support is being refined. ONC certification increasingly expects transparency for AI-assisted tools. HIPAA's applicability to AI-generated content is being tested in practice. Our customers' compliance teams are asking for governance evidence that does not yet exist.

There is no established product category for healthcare AI safety infrastructure. Existing solutions are either general-purpose AI guardrails that lack clinical specificity, or manual clinician review processes that do not scale. The opportunity is to build the safety layer that healthcare organizations need before their AI deployments outpace their governance capabilities.

## Problem Framing

The fundamental issue is that healthcare AI outputs are entering clinical workflows without independent safety validation.

Today, healthcare organizations deploying AI tools rely on one or more of these approaches:

1. **Model-level safeguards.** Prompt engineering, fine-tuning, and retrieval-augmented generation to reduce unsafe outputs at the model layer. This helps but does not eliminate the problem. Models are probabilistic. They will produce unsafe outputs. The question is what catches them.

2. **Manual clinician review of all outputs.** Every AI-generated piece of content is reviewed by a clinician before delivery. This works at low volume but breaks down at scale. A patient messaging tool generating 200 responses per day cannot have every response reviewed by a physician. At volume, reviewers start skimming and approving reflexively.

3. **No safety validation at all.** The AI output goes directly into the clinical workflow. The organization relies on the model and on downstream clinicians (who may not know the content was AI-generated) to catch problems. This is the most common current state and the most dangerous.

None of these is a viable long-term strategy. Healthcare needs a dedicated safety infrastructure layer that operates independently of the model, catches the specific failure modes that matter in clinical contexts, and produces the compliance evidence that regulators expect.

## Why Existing Approaches Fail

**General-purpose AI guardrails** (content safety filters, generic hallucination detectors) are not designed for healthcare. They do not understand clinical terminology, cannot validate against patient-specific evidence, do not detect PHI with healthcare-specific patterns (MRNs, provider NPIs, dates of service in clinical formatting), and cannot evaluate clinical safety (whether a recommended dosage is within safe range for the patient's renal function, for example). Healthcare AI safety requires clinical domain knowledge at every layer of the detection pipeline.

**Prompt engineering and RAG** reduce hallucination rates but do not eliminate them. In our analysis of customer deployments, RAG-based systems still produce ungrounded clinical claims in 1 to 3% of outputs. For patient-facing applications, a 1% hallucination rate at scale means dozens of fabricated clinical claims reaching patients per day.

**Manual review** is the gold standard for safety but does not scale. It also creates a perverse incentive: the more AI outputs an organization generates, the more clinician time it consumes for review, which reduces the net efficiency gain that justified AI in the first place.

## Target Users

**Primary**: Clinical informatics teams (often reporting to the CMIO) who are responsible for deploying, validating, and governing AI tools in clinical workflows. They need safety infrastructure they can configure per workflow, monitor operationally, and present to compliance as evidence.

**Secondary**: Compliance and privacy officers who need to demonstrate HIPAA adherence for AI-generated content, produce audit trails for regulatory inquiries, and establish governance policies. They are policy authors and auditors, not daily operators.

**Tertiary**: Clinician reviewers who see the escalation queue. They need enough clinical context and risk signal to make a fast judgment call, not a flood of false positives that wastes their time.

**Buyer**: CMIO (Chief Medical Information Officer) and CISO (Chief Information Security Officer), who are jointly accountable for the safety and privacy posture of clinical AI. In some organizations, the CIO or VP of Digital Health owns this budget.

## Options Considered

### Option A: Model-Layer Safety Toolkit

Build a set of safety-enhancing tools (better RAG pipelines, medical knowledge grounding, safety-tuned classifiers) that customers integrate directly into their AI models and prompts.

**Advantages**: No new infrastructure. Improves model quality at the source. Low integration overhead.
**Disadvantages**: Safety controls are embedded in the model pipeline, not independently auditable. Different models require different integrations. No unified audit trail. Compliance teams cannot inspect or configure safety independently. Does not catch failures that happen despite model-level safeguards.

### Option B: Dedicated Safety Infrastructure Layer (Recommended)

Build a standalone safety service that intercepts AI outputs after generation and before delivery. Runs a pipeline of healthcare-specific safety checks. Routes outputs to delivery, escalation, or block. Produces an independent audit trail.

**Advantages**: Independent of the model. Auditable by compliance. Configurable per workflow. Catches what model-level safeguards miss. Single integration point regardless of upstream model. Unified governance.
**Disadvantages**: Adds latency to the output path. Requires integration into each clinical workflow pipeline. Becomes a critical dependency.

### Option C: Enhanced Clinician Review Platform

Build tooling that makes manual clinician review faster and more structured: review queues with AI risk scoring, standardized review templates, review time tracking, and compliance documentation.

**Advantages**: Keeps humans in the loop for all outputs. High safety confidence. Regulatory defensibility.
**Disadvantages**: Does not scale. Every AI output still requires human review. Does not reduce the review burden; just makes it more organized. At volume, this approach costs more in clinician time than it saves in AI efficiency.

## Tradeoffs

| Dimension | Model Toolkit (A) | Safety Layer (B) | Review Platform (C) |
|-----------|-------------------|-------------------|---------------------|
| Catches model failures | Partially | Yes (independent) | Yes (human) |
| Independent auditability | No | Yes | Partial |
| Scalability | Scales with model | Scales with service | Does not scale |
| Clinician burden | None | Low (escalation only) | High (all outputs) |
| PHI protection | Model-dependent | Dedicated scanner | Human catches |
| Regulatory evidence | Weak | Strong | Strong but manual |
| Integration effort | Per-model | One-time per pipeline | Per-workflow |
| Latency impact | None | Moderate (< 500ms) | High (human time) |

## Recommended Direction

**Option B: Dedicated Safety Infrastructure Layer.**

The core reason is the same principle that governs pharmaceutical quality control: you do not rely on the manufacturing process alone to guarantee safety. You have independent quality checks that are separate from production, operated by a different team, and auditable. Healthcare AI needs the same architecture.

A dedicated safety layer creates a clear separation between "what the model produces" and "what reaches the clinical workflow." It gives compliance teams an independent control point they can inspect, configure, and audit without touching the model pipeline. It catches the failures that model-level safeguards miss. And it scales with the service, not with the number of clinician reviewers.

The latency tradeoff is acceptable for healthcare workflows. Clinical AI tools do not require real-time performance. A patient message drafted by AI is not time-critical to the millisecond. A clinical note summary can tolerate an additional 300 to 500ms of processing. Even diagnostic support tools have response time budgets measured in seconds, not milliseconds. The safety latency is within acceptable bounds for every healthcare AI use case we have evaluated.

## Why This Direction Wins

1. **It creates the safety standard for healthcare AI.** The organization that defines how healthcare AI outputs are validated, classified, and governed establishes the framework that health systems adopt. Safety taxonomy becomes sticky.

2. **It unlocks clinical AI adoption.** Healthcare organizations that want to deploy AI but cannot accept the liability of unvalidated outputs need this layer. TrustLayer does not compete with AI model vendors. It makes their products deployable in regulated environments.

3. **It creates a compound data advantage.** Every AI output processed through TrustLayer generates labeled safety data: what kinds of hallucinations occur, in which workflows, with which models, at what rates. This data improves detection accuracy over time and is extremely difficult for competitors to replicate without the same volume and diversity of clinical AI traffic.

## Major Risks

1. **Detection accuracy.** If grounding checks or hallucination detection produce too many false positives, clinicians lose trust and the safety layer becomes an obstacle rather than an enabler. Mitigation: extended shadow mode with clinician feedback; precision as a first-class metric.

2. **Clinical domain coverage.** Healthcare is not one domain. Oncology, cardiology, primary care, pediatrics, and mental health all have different terminology, risk profiles, and safety thresholds. Scaling safety checks across clinical specialties is a multi-year investment. Mitigation: start with one workflow in one specialty; expand methodically.

3. **EHR integration complexity.** Epic, Oracle Health, and other EHR systems have different integration patterns, API structures, and data models. Deep integration is necessary for production use but expensive to build. Mitigation: start with vendor-agnostic pipeline integration; pursue EHR-specific connectors in Phase 3.

4. **Regulatory ambiguity.** FDA, ONC, and other regulatory bodies are still defining how AI governance should work in healthcare. Building to a moving target is risky. Mitigation: design for auditability and configurability rather than for compliance with a specific regulation. The audit trail and configurable policies should satisfy any reasonable governance framework.

## Open Questions

1. Should TrustLayer's clinical safety classifier be a single model or a pipeline of specialized classifiers (medication safety, dosage validation, scope-of-practice checking)? The pipeline approach is more accurate but more complex to maintain.
2. How should TrustLayer handle multi-turn conversations where individual responses are safe but the accumulated conversation creates a risk? This is especially relevant for patient-facing chatbots.
3. What is the right relationship with clinical advisory boards? Safety policies need clinical input, but advisory boards operate on quarterly timelines, and safety needs evolve faster.
4. Should TrustLayer offer a "confidence score" alongside approved outputs so that downstream clinicians understand the safety margin? This could improve trust but also creates a new cognitive burden.
5. How do we price this? Per-output, per-workflow, or per-organization? Per-output aligns incentives (organizations pay proportionally to AI usage) but creates unpredictable costs. Enterprise licensing is simpler but may not reflect value.

---

*This memo recommends proceeding with Option B. Phase 1 scope: single AI workflow (patient message drafting), single clinical department, with grounding checks, hallucination detection, PHI scanning, clinical safety classification, clinician escalation, and audit logging as core deliverables.*
