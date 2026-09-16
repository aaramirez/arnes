# Research & Intelligence — Reference Harness Specification
**Canonical ID:** RH-03  
**Version:** 0.1  
**Category:** Evidence-Based Knowledge Work  
**Source catalog:** `REFERENCE_HARNESS_CATALOG_v0.1.md`

## Architecture baseline

This reference harness is a Domain Pack specialization of the Generic Agent Harness and MUST reuse its primitives rather than placing domain logic in the core.

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
Domain Pack
        ↓
Reference Harness
        ↓
Specific Use Case
```

### Mandatory architectural concerns

Every implementation must explicitly address:

- Activation and Trigger Adapters
- Admission Control and Routing
- Agent Runtime and Execution Controller
- State, Context, and Memory boundaries
- Skills and versioned Capabilities
- Workflow dependencies and safe parallelism
- Agent-to-Agent communication and delegation when applicable
- Protocol/transport independence
- Identity, authorization, delegated authority, and secrets
- Human-in-the-Loop without coupling to a UI
- Durable execution, checkpoints, retry, and idempotency
- Events, artifacts, and side effects
- Technical observability and immutable audit evidence
- Budget/FinOps and business outcomes
- Evaluation Harness separated from Production Runtime
- Data governance and Architecture Constitution compliance

### Core execution skeleton

```text
External Trigger
      ↓
Trigger Adapter
      ↓
ActivationRequest
      ↓
AdmissionController
      ↓
ActivationRouter
      ↓
Workflow / Agent Runtime
      ↓
Context + State + Policies
      ↓
Model / Skills / Capabilities
      ↓
Validation / HITL / Side Effects
      ↓
Checkpoint + Audit + Outcomes
```



## Domain definition from the catalog

# RH-03 — Research & Intelligence Harness

## Propósito
Investigar preguntas complejas y producir conclusiones trazables a evidencia.

## Activaciones
Usuario, API, scheduled intelligence request, event indicating new evidence.

## Agentes
Research Planner, Source Researcher, Evidence Analyst, Contradiction Reviewer, Synthesizer.

## State
`ResearchState`, `ResearchQuestion`, `Source`, `Evidence`, `Claim`, `Contradiction`, `Citation`.

## Context
Fuentes internas/externas, evidence store, research history, domain rules.

## Skills
`formulate-question`, `discover-sources`, `evaluate-source`, `extract-evidence`, `compare-claims`, `synthesize`.

## Capabilities
Web/search, enterprise knowledge, document readers, citation manager.

## Workflows
Question → Plan → Search → Evidence → Claims → Contradictions → Synthesis → Citation validation.

## HITL
Scope changes, ambiguous evidence, consequential recommendations.

## Evals
Source quality, citation coverage, claim support, contradiction detection, freshness.

## Ejercicios
1. Market research.
2. Competitive intelligence.
3. Legal research.
4. Technology landscape.
5. Policy research.
6. Vendor intelligence.
7. Academic literature review.
8. Due-diligence research.

---

---

# Detailed implementation specification


## Central invariant

> A claim is not evidence, and evidence is not a source.

```text
Source
  ↓ contains
Evidence
  ↓ supports / contradicts
Claim
  ↓ contributes to
Conclusion
```

## Reference data contracts

```text
ResearchQuestion {
  id
  question
  scope
  subQuestions[]
  freshnessRequirement
  evidenceStandard
  constraints[]
}

Source {
  sourceId
  reference
  publisher
  publishedAt
  retrievedAt
  sourceType
  classification
  checksumOrVersion
}

Evidence {
  evidenceId
  sourceId
  proposition
  locationRef
  relevance
  limitations[]
}

Claim {
  claimId
  statement
  supportingEvidenceIds[]
  contradictingEvidenceIds[]
  confidence
  assumptions[]
  status
}
```

Canonical research state stores sources/evidence/claims separately from generated prose.

## Research plan

```text
ResearchPlan {
  subQuestions[]
  sourceStrategy[]
  evidenceRequirements[]
  parallelTracks[]
  stopConditions[]
  reviewRequirements[]
  budgetAllocation
}
```

Example:

```text
Main Question
├── Track A: Primary Sources
├── Track B: Independent Reporting
├── Track C: Counter-evidence
└── Track D: Historical Context
```

Independent tracks execute in parallel.

## Canonical workflow

```text
Activation
   ↓
Normalize Question
   ↓
Research Plan
   ↓
Parallel Source Discovery
   ↓
Source Evaluation
   ↓
Evidence Extraction
   ↓
Evidence Graph
   ↓
Claims + Contradictions
   ↓
Gap Analysis
   ├── gaps + budget → research again
   └── sufficient
          ↓
       Synthesis
          ↓
Claim/Citation Validation
          ↓
Human Review if required
          ↓
Final Research Artifact
```

## Evidence graph

```text
S1 → E1 ─supports──→ C1
S2 → E2 ─supports──→ C1
S3 → E3 ─contradicts→ C1
```

Contradiction changes confidence or forces qualification/review; it must not be silently discarded.

## Stop conditions

Research must be bounded by combinations of:
- required claim coverage;
- critical evidence gaps;
- source diversity;
- freshness;
- cost;
- source/search-call count;
- wall-clock deadline;
- diminishing evidence value.

`INSUFFICIENT_EVIDENCE` is a legitimate terminal result.

## Agent communication

The planner can delegate bounded sub-questions to primary-source, market, scientific, legal, or contradiction agents. Returned packages are locally validated. An external specialist may use A2A through `AgentCommunicationGateway`, but federation does not imply trust.

## Example policies

```text
POL-RES-001: Material claims require evidence.
POL-RES-002: Source provenance is preserved.
POL-RES-003: Generated summaries cannot masquerade as primary sources.
POL-RES-004: Restricted data cannot be sent to unauthorized external agents/models.
POL-RES-005: Freshness-sensitive research must satisfy recency requirements.
POL-RES-006: High-quality contradictory evidence must be surfaced.
```

## Failure and recovery exercise

Scenario: two high-authority sources conflict on a material fact.

Expected behavior:
1. Preserve both source records.
2. Extract separate evidence objects.
3. Link both to the claim with opposing relationships.
4. Lower/qualify confidence rather than fabricate certainty.
5. Search for resolving evidence if budget permits.
6. Request expert review when configured.
7. Audit the final treatment.

## Minimum eval suite

`source_relevance`, `source_quality`, `evidence_extraction_accuracy`, `claim_evidence_alignment`, `citation_correctness`, `contradiction_detection`, `unsupported_claim_detection`, `freshness_compliance`, `synthesis_faithfulness`.

## Required student deliverables

- Source/Evidence/Claim schemas;
- research plan contract;
- parallel research DAG;
- at least two source adapters;
- evidence graph;
- contradiction and gap handling;
- stop conditions;
- data-governance rules;
- audit model;
- eval suite;
- final memo plus machine-readable evidence package;
- ADRs.

## Definition of Done

The implementation demonstrates explicit provenance, bounded parallel research, source evaluation, evidence extraction, claim validation, contradiction handling, evidence gaps, stop conditions, durable state, policy enforcement, audit, and a final artifact whose material claims can be traced to evidence.


---

## Student conformance checklist

- [ ] Domain logic remains outside Generic Core.
- [ ] At least two activation sources are normalized to `ActivationRequest`.
- [ ] Admission rules are explicit.
- [ ] State contracts are typed and durable where required.
- [ ] Context is intentionally assembled and provenance-aware.
- [ ] Skills and capabilities have explicit responsibilities.
- [ ] Independent work can execute in parallel; dependencies are explicit.
- [ ] Human interaction is a runtime contract, not a UI dependency.
- [ ] Failures are typed and recovery behavior is defined.
- [ ] Side effects have retry/idempotency semantics.
- [ ] Audit can reconstruct consequential actions.
- [ ] Technical metrics and domain/business outcomes are distinct.
- [ ] Evaluation runs outside Production Runtime.
- [ ] At least three ADRs document meaningful architectural decisions.
