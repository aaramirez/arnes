# Agent Harness Architecture Pack — v0.5

## Canonical documents
1. `ARCHITECTURE_CONSTITUTION.md` — normative architectural rules; now includes enterprise amendment P-16..P-30.
2. `Arquitectura_Agent_Harness_inspirado_en_Pi.md` — core architecture plus enterprise runtime extension.
3. `Propuesta_Libro_Construyendo_un_Agent_Harness_v0.5.md` — canonical 61-chapter book structure (Chapter 0 through 60).
4. `REGLAS_LIBRO_AGENT_HARNESS.md` — canonical editorial/pseudocode/contract consistency rules plus enterprise communication rules.
5. `harness-empresarial-general.md` — enterprise readiness criteria, extended beyond the original 14 criteria.
6. `BOOK_HARNESS_BUILD_INSTRUCTIONS.md` — Book Domain Pack reference implementation.
7. `APPLICATION_DEVELOPMENT_HARNESS.md` — Coding Domain Pack reference implementation.

## Historical documents
`Estructura_Libro_Construyendo_un_Agent_Harness.md`, `Estructura_Libro_Construyendo_un_Agent_Harness_v0.2.md` and `Propuesta_Libro_Construyendo_un_Agent_Harness_v0.3.md` are retained for history and explicitly marked superseded.

## Canonical architectural decisions added in v0.5
- Harness activation is not limited to human prompts.
- Every trigger normalizes to ActivationRequest before execution.
- Admission precedes routing/execution.
- Agent communication is a first-class boundary.
- Communication semantics, interoperability protocol and transport are separate layers.
- A2A is an external interoperability/federation adapter, not an Agent Core dependency.
- Internal delegation and external federation are distinct.
- Agent authority is delegated explicitly and least-privileged.
- Webhooks, WebSockets, HTTP, SSE, gRPC, queues, Kafka and Service Bus are transport/delivery options.
- Durable execution, idempotency and messaging reliability are core enterprise concerns.
- Capability lifecycle/versioning is governed.
- Data Governance and Execution Fabric are first-class planes.
- Execution Ledger and Immutable Audit Ledger are distinct.
- Evaluation Harness is separate from Production Runtime.
- Business outcomes/FinOps complement technical observability.
- Domain Packs specialize a shared Generic Harness.
