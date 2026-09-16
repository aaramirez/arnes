# Book & Content Production — Reference Harness Specification
**Canonical ID:** RH-01  
**Version:** 0.1  
**Category:** Knowledge Work / Artifact Production  
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

# RH-01 — Book & Content Production Harness

## Propósito
Crear, revisar, validar, versionar y publicar contenido estructurado de larga duración.

## Actores
Autores, editores, revisores técnicos, diseñadores, profesores, publishers.

## Activaciones
Usuario, commit, modificación de capítulo, schedule, review request, build event.

## Agentes
- Book/Content Architect
- Author
- Technical Reviewer
- Pedagogical Reviewer
- Consistency Reviewer
- Editor

## State
`ContentProjectState`, `ChapterState`, `ContentManifest`, `ContractRegistry`, `Glossary`.

## Context
Contenido previo, estructura, guías editoriales, fuentes, ADRs, terminología.

## Skills
`write-section`, `define-concept`, `review-technical-content`, `review-pedagogy`, `revise`, `publish`.

## Capabilities
Files, search, diagrams, document generation, Web renderer, PDF renderer.

## Workflows
Plan → Draft → Review → Revise → Validate → Publish.

## HITL
Cambios de estructura, arquitectura, claims sensibles, publicación.

## Evals
Exactitud, consistencia, claridad, progresión pedagógica, referencias.

## Business outcomes
Tiempo de producción, defectos editoriales, retrabajo, calidad, publicación.

## Ejercicios
1. Libro técnico.
2. Manual de operaciones.
3. Curso universitario.
4. Knowledge playbook.
5. Documentación de API.
6. Manual de onboarding.
7. Reporte anual.
8. Policy handbook.

---

---

# Detailed implementation specification


## Reference state contracts

```text
ContentProjectState {
  projectId
  projectVersion
  manifest
  glossary
  contractRegistry
  architectureDecisions[]
  contentUnits[]
  reviewQueue[]
  releaseState
}

ContentUnitState {
  contentId
  title
  status
  sourceArtifactRef
  dependencies[]
  contractsUsed[]
  findings[]
  version
  checksum
}
```

Recommended lifecycle:

```text
PLANNED → DRAFTING → TECHNICAL_REVIEW → PEDAGOGICAL_REVIEW
        → REVISION → VALIDATED → RELEASED
```

## Context engineering

Do not send the whole book or corpus on every turn. Assemble a bounded `ContextBundle` containing the target content, neighboring dependencies, relevant glossary terms, contracts, Constitution rules, ADRs, review findings, and evidence.

```text
ContextItem {
  content
  sourceRef
  version
  retrievedAt
  classification
}
```

## Canonical workflow

```text
Activation
  ↓
Load Project State
  ↓
Plan / Select Content Unit
  ↓
Draft or Revise
  ↓
┌─────────────────┬──────────────────┐
│ Technical Review│ Pedagogical Review│
└────────┬────────┴─────────┬────────┘
         └──────────┬───────┘
                    ↓
           Consistency Review
                    ↓
             Findings?
          YES ──────┴────── NO
           ↓                 ↓
         Revise           Validate
                             ↓
                         Human Gate
                             ↓
                          Publish
```

Technical and pedagogical reviews may run in parallel; publication depends on required validation gates.

## Example policies

```text
POL-BOOK-001: Blocking findings prevent release.
POL-BOOK-002: Editors cannot silently change registered technical contracts.
POL-BOOK-003: Evidence-required claims preserve source provenance.
POL-BOOK-004: Generated output is not canonical until validation succeeds.
```

## HITL

Human tasks are durable objects and may be surfaced by web, chat, email, ticket, mobile, or approval queue. Typical gates: TOC approval, architecture changes, unresolved reviewer conflicts, and release approval.

## Failure and recovery exercise

Scenario: PDF rendering fails after all chapters are validated.

Expected behavior:
1. Preserve validated content and review state.
2. Record `BUILD_FAILURE`.
3. Retry only the publishing step when safe.
4. Use release idempotency key = project version + target.
5. Never regenerate chapters merely because the renderer failed.
6. Audit the failure and resumed build.

## Minimum eval suite

`technical_accuracy`, `terminology_consistency`, `contract_consistency`, `cross_reference_integrity`, `citation_support`, `pedagogical_clarity`, `artifact_build`.

## Required student deliverables

- architecture diagram;
- activation sequence;
- state contracts;
- content/review workflow DAG;
- at least 5 capability contracts;
- HITL sequence;
- failure/recovery sequence;
- audit model;
- eval plan;
- Markdown plus Web/PDF publishing flow;
- ADRs explaining at least three architecture choices.

## Definition of Done

The implementation demonstrates at least two activation modes, explicit durable state, bounded context construction, two reviewer responsibilities, safe parallel review, validation, HITL, checkpoint/resume, artifact versioning, audit, evals, and a final publication artifact.


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
