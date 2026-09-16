# Construyendo un Agent Harness
## Del Agent Loop a una Enterprise Agent Operating Layer
### Estructura canónica del libro — Versión 0.5

## Propósito
Construir el sistema incrementalmente. Cada capítulo introduce una necesidad, un contrato, un componente o una propiedad operacional; el pseudocódigo se deriva de la arquitectura y nunca al revés.

La progresión canónica es:

```text
Agent Loop
→ Model / Context / Tools
→ State / Skills / Policies
→ Activation
→ Admission / Routing
→ Workflow / Durable Execution
→ Agent Communication
→ Delegation / Execution Graph
→ Protocols / Transports
→ Federation / A2A
→ Enterprise Capabilities
→ Data Governance
→ Reliability
→ Execution Fabric
→ Audit / Operations
→ Evaluation / FinOps / Outcomes
→ Domain Packs
→ Enterprise Agent Operating Layer
```

# LEVEL 0 — ARCHITECTURAL FOUNDATION

## Capítulo 0. Architecture Constitution
Los 30 principios, invariants, decision ownership, contracts, lifecycle, security, budgets, failure semantics, evolution rules y ADRs. La Constitution es normativa para todo el libro.

# PARTE I — CONSTRUIR UN AGENTE

## Capítulo 1. Qué es realmente un Agent Harness
Problema, límites entre model/agent/harness/application y arquitectura mínima.

## Capítulo 2. Agent Core y separación de estado
AgentDefinition, AgentState, ExecutionContext y por qué evitar una clase Agent monolítica.

## Capítulo 3. Agent Loop
Turn, Run, termination, tool-call continuation y control del loop.

## Capítulo 4. Model Independence
ModelGateway, ModelRequest/Response, adapters y normalización de mensajes.

## Capítulo 5. Context Engineering
ContextEngine, ContextProviders, selección, provenance, token budget y diferencia con RAG.

## Capítulo 6. Tool Runtime
ToolCall, ToolResult, execution boundary, schemas y errores.

## Capítulo 7. Tool Registry y Capability Model
Discovery, capability metadata, risk, side effects y contracts.

## Capítulo 8. Paralelismo y dependencias
Execution graph local, independent tool calls, dependency ordering y shared-state hazards.

# PARTE II — HACERLO CONTROLABLE

## Capítulo 9. Sessions, Runs y State
SessionState vs AgentState vs RunState; persistencia y ownership.

## Capítulo 10. Skills
Procedural knowledge, loading, scope, versioning y separación de prompts.

## Capítulo 11. Events, Hooks y Effects
Events observe; hooks intervene; adapters producen efectos secundarios como TUI/UI/logging.

## Capítulo 12. Policy Engine
Deterministic authorization, side-effect policy, constraints y policy decisions.

## Capítulo 13. Human Interaction sin acoplar UI
Approval, input, review, decision, interrupt, batch approval, handoff y supervised autonomy.

## Capítulo 14. Budgets, Cancellation y Termination
Turns, tools, time, cost, concurrency y cancellation propagation.

## Capítulo 15. Failure Constitution
Taxonomía, retryability, recoverability, partial failure y error ownership.

# PARTE III — ACTIVAR EL HARNESS EN LA EMPRESA

## Capítulo 16. Más allá del prompt: Activation Layer
User, API, webhook, queue, event, schedule, file, database, system y agent triggers.

## Capítulo 17. ActivationRequest y Trigger Adapters
Normalización del ingress y separación entre canal y runtime.

## Capítulo 18. Admission Controller
Authorization, tenant, capacity, budget, deduplication, rate limits y relevance.

## Capítulo 19. Activation Routing
ExecutionTarget: Agent, Workflow, Job o Human Task. Activation Registry.

# PARTE IV — WORKFLOWS Y DURABILIDAD

## Capítulo 20. Agent Runtime vs Workflow Runtime
Cuándo usar razonamiento probabilístico y cuándo flujo determinístico.

## Capítulo 21. Durable Execution
RunStore, checkpoints, pause/resume, crash recovery y long-running processes.

## Capítulo 22. Schedulers, Timers y Delayed Work
Tiempo como activador, reminders, deadlines y durable timers.

## Capítulo 23. Idempotency y side effects
Idempotency keys, effect journal y effectively-once business execution.

# PARTE V — COMUNICACIÓN ENTRE AGENTES

## Capítulo 24. Agent-to-Agent Communication
Por qué comunicación entre agentes merece un boundary propio.

## Capítulo 25. Semantic Contracts
AgentTaskRequest, AgentTaskResult, Message, Artifact, Progress, Failure, Question, Cancel y Escalation.

## Capítulo 26. Delegation Manager
Authority delegation, least privilege, DelegationToken, budgets y constraints.

## Capítulo 27. Execution Graph
Parent/child runs, max depth, child limits, cycles, cancellation y observability.

## Capítulo 28. Internal Delegation vs External Federation
Sub-agents administrados versus agentes independientes.

## Capítulo 29. Protocol Independence
AgentCommunicationGateway y protocol adapters.

## Capítulo 30. Transport Independence
In-process, HTTP, webhook, WebSocket, SSE, gRPC, queues, Kafka, Service Bus y Pub/Sub.

## Capítulo 31. Delivery Semantics
Ack, retry, timeout, ordering, deduplication, backpressure, poison messages y DLQ.

## Capítulo 32. A2A
Agent Cards/discovery, tasks/messages/artifacts, A2A adapter y por qué A2A no pertenece al Agent Core.

## Capítulo 33. Agent Registry y Discovery
Managed, internal, external y federated agents; capability-based discovery.

# PARTE VI — CAPABILITIES E INTEGRACIÓN EMPRESARIAL

## Capítulo 34. Enterprise Capability Plane
CapabilityRegistry como catálogo operacional y de seguridad.

## Capítulo 35. Capability Lifecycle
Versioning, backward compatibility, rollout, feature flags, deprecation y retirement.

## Capítulo 36. Identity, Principal y Delegated Authority
Human, service, agent, workflow y system principals.

## Capítulo 37. Credential Broker y Secrets
Short-lived credentials, secret isolation y model-blind credentials.

## Capítulo 38. Integration Adapters
CRM, ERP, databases, cloud, network, GitHub y sistemas legacy.

# PARTE VII — DATA & CONTEXT PLANE

## Capítulo 39. Enterprise Context Layer
Retrieval, permissions, provenance, freshness, ranking y context budgets.

## Capítulo 40. Memory Architecture
Working, session, episodic, semantic y organizational memory.

## Capítulo 41. Artifact Store
Files, reports, code, images, evidence, versions y retention.

## Capítulo 42. Data Governance
Classification, residency, retention, encryption, deletion, lineage, consent y legal hold.

## Capítulo 43. Tenant Isolation
Context, memory, credentials, artifacts, logs, vector stores y budgets.

# PARTE VIII — RELIABILITY & EXECUTION FABRIC

## Capítulo 44. Reliability Manager
Retries, exponential backoff, circuit breakers, bulkheads y fallbacks.

## Capítulo 45. Concurrency Control
Optimistic concurrency, resource versions, locks y conflict handling.

## Capítulo 46. Execution Fabric
Interactive/async workers, workload classes, horizontal scaling y placement.

## Capítulo 47. Deployment Topology
Cloud, on-prem, hybrid, regional execution, jurisdiction y data locality.

## Capítulo 48. Model Router
Capability, classification, latency, cost, region, policy y availability-based routing.

# PARTE IX — AUDIT, OPERATIONS Y GOVERNANCE

## Capítulo 49. Execution Ledger
Reconstrucción causal de activations, decisions, model calls, tools, approvals y effects.

## Capítulo 50. Immutable Audit Ledger
Regulatory evidence, append-only records, tamper evidence y retention.

## Capítulo 51. Technical Observability
Logs, metrics, traces, run topology, latency, tokens y errors.

## Capítulo 52. Agent Operations
RunRegistry, health, backlog, stuck runs, pending approvals, DLQ y incident signals.

## Capítulo 53. Kill Switches y Operational Control
Run, agent, capability, integration, tenant y global autonomy controls.

# PARTE X — EVALUATION, ECONOMICS & OUTCOMES

## Capítulo 54. Evaluation Harness
Golden sets, regression, adversarial tests, policy/tool tests y offline evaluation.

## Capítulo 55. Certification & Release Management
Agent/skill/policy/model/tool versions, promotion gates, canary y rollback.

## Capítulo 56. FinOps
Cost per run/agent/team/tenant, model/tool/compute/external API/human cost.

## Capítulo 57. Business Observability
OutcomeTracker, resolution, escalation, SLA/SLO, revenue/value, time-to-resolution y ROI.

# PARTE XI — DOMAIN PACKS Y PLATAFORMA FINAL

## Capítulo 58. Domain Packs
Generic Harness + Domain State + Skills + Tools + Policies + Validators + UI.

## Capítulo 59. Reference Harnesses
Application Development Harness, Book Production Harness, Research Harness, Business Operations Harness y Data Analysis Harness.

## Capítulo 60. Enterprise Agent Operating Layer
Integración final de Ingress, Execution, Agent Interoperability, Capability, Data, Control, Reliability, Observability/Governance y Execution Fabric.

# Arquitectura final del libro

```text
ENTERPRISE WORLD
Users · APIs · Queues · Events · Schedules · Systems · Agents
                         │
                         ▼
INGRESS & ACTIVATION PLANE
ActivationGateway · TriggerAdapters · ActivationRegistry
AdmissionController · ActivationRouter
                         │
                         ▼
EXECUTION PLANE
WorkflowRuntime · AgentRuntime · JobRuntime · Human Tasks
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
AGENT INTEROPERABILITY             AGENT CORE
CommunicationGateway               AgentLoop
DelegationManager                  ContextEngine
ExecutionGraph                     ModelGateway
Internal/A2A adapters              ExecutionController
          │                             │
          └──────────────┬──────────────┘
                         ▼
CAPABILITY & INTEGRATION PLANE
ToolRuntime · PolicyEngine · CapabilityRegistry/Lifecycle
CredentialBroker · IntegrationAdapters
                         │
                         ▼
ENTERPRISE SYSTEMS

Cross-cutting:
DATA & CONTEXT · CONTROL · RELIABILITY · OBSERVABILITY/GOVERNANCE · EXECUTION FABRIC
```

# Regla pedagógica
Cada capítulo debe incluir: problema, arquitectura antes, contratos nuevos/modificados, component card, pseudocódigo tipado, sequence diagram, failure semantics, tests/evals, architecture after, Constitution impact, ADR impact y qué NO cambia.
