# Agent Harness Architecture Constitution
## Version 1.0 — Foundational Architecture Rules

> **Constitutional rule:** Probabilistic systems may propose decisions. Deterministic systems must govern consequences.

---

# Preámbulo

Esta Architecture Constitution define las reglas fundamentales que gobiernan el diseño, implementación, evolución y operación de un Agent Harness.

El harness existe para proporcionar una infraestructura segura, observable, extensible, durable y agnóstica al modelo, donde sistemas probabilísticos puedan interpretar objetivos, razonar y proponer acciones, mientras mecanismos determinísticos gobiernan su ejecución y sus consecuencias.

```text
Probabilistic Intelligence
        │
        │ proposes
        ▼
Deterministic Runtime
        │
        │ governs
        ▼
External World
```

El modelo puede decidir **qué intentar**.

El harness decide **qué puede ocurrir, cómo puede ocurrir, bajo qué límites y con qué trazabilidad**.

Esta constitución debe utilizarse como criterio para:

- diseño de componentes;
- architecture reviews;
- pull requests;
- incorporación de nuevas capacidades;
- definición de extensiones;
- integraciones;
- seguridad;
- gobierno;
- evaluación de trade-offs;
- breaking changes;
- evolución del runtime.

La Constitution se organiza en los siguientes artículos:

```text
Architecture Constitution
│
├── Article I   — Fundamental Principles
├── Article II  — Invariants
├── Article III — Component Sovereignty
├── Article IV  — Decision Ownership
├── Article V   — Lifecycle
├── Article VI  — Execution
├── Article VII — Failure Semantics
├── Article VIII — Human Interaction
├── Article IX  — Resources and Budgets
├── Article X   — Observability
├── Article XI  — Evolution
└── Article XII — Deterministic vs Agentic Boundary
```

---

# Article I — Fundamental Principles

## P-01 — Context is a first-class architectural component

El contexto no es simplemente historial de conversación. Debe ser construido, seleccionado, priorizado, resumido, limitado y controlado por el harness.

## P-02 — The model is replaceable

El runtime nunca debe depender estructuralmente de un proveedor específico. Los modelos se integran mediante contratos y adapters.

## P-03 — Tools are explicit capabilities, not prompt tricks

Toda acción sobre el mundo externo debe representarse mediante contratos explícitos, validables y observables.

## P-04 — Every action produces observable events

Las operaciones relevantes deben generar eventos que permitan UI, logging, tracing, persistence, analytics y audit.

## P-05 — Side effects pass through policy

Toda acción con efectos secundarios debe atravesar una capa determinística de validación, autorización y políticas.

## P-06 — Agents are configuration over a shared runtime

Los agentes deben configurarse sobre un runtime común en lugar de convertirse en aplicaciones independientes que reimplementan infraestructura.

## P-07 — Skills encode reusable procedural knowledge

El conocimiento procedural reusable debe estar separado del core, las tools y la identidad del agente.

## P-08 — Agent state and session state are different concerns

El estado operativo de una ejecución y la historia persistente de una sesión son responsabilidades diferentes.

## P-09 — Single-agent reliability precedes multi-agent complexity

No se debe introducir multi-agent como sustituto de resolver correctamente context, tools, state, reliability y governance.

## P-10 — The harness owns execution state—not the model

El modelo propone decisiones semánticas. El harness controla continuidad, ejecución, estado, límites y lifecycle.

## P-11 — UI is an adapter, not part of the core

TUI, Web, API, IDE, Slack, Teams u otras interfaces son adapters alrededor del mismo runtime.

## P-12 — Events observe; hooks intervene

Los eventos comunican hechos ocurridos. Los hooks permiten intervenir en puntos controlados del lifecycle.

## P-13 — Authorization is deterministic and external to the LLM

El modelo nunca constituye una fuente de verdad para identidad, permisos o autorización.

## P-14 — Context should be selected, not dumped

Más contexto no implica mejor razonamiento. El harness debe seleccionar información relevante dentro de presupuestos explícitos.

## P-15 — Automation and agents should share the same execution substrate

Automatizaciones, agentes interactivos y agentes autónomos deben compartir infraestructura de estado, tools, políticas, eventos, observabilidad y durable execution.

---

# Article II — Invariants

Los invariants son reglas verificables que nunca deben violarse.

## Model Invariants

### INV-01
`AgentCore` no depende directamente de APIs específicas de OpenAI, Anthropic, Google u otro proveedor.

### INV-02
Toda comunicación interna del runtime utiliza contratos propios, incluyendo `AgentMessage`.

### INV-03
El modelo nunca constituye una fuente de autorización.

## Tool Invariants

### INV-04
Todo `ToolCall` debe validarse antes de ejecutarse.

### INV-05
Todo `ToolCall` pasa por `ToolRuntime`.

### INV-06
Todo side effect pasa por `PolicyEngine`.

### INV-07
Todo `ToolResult` vuelve al ciclo del agente como observación explícita cuando el lifecycle continúa.

## Execution Invariants

### INV-08
El harness es propietario del execution state.

### INV-09
Todo `AgentRun` tiene límites explícitos.

### INV-10
Todo `AgentRun` debe poder cancelarse.

### INV-11
Side effects críticos deben soportar idempotencia, deduplicación o una protección equivalente.

## State Invariants

### INV-12
`AgentState` y `SessionState` permanecen conceptualmente independientes.

### INV-13
Una ejecución durable debe poder reconstruirse desde estado persistido suficiente.

## Human Interaction Invariants

### INV-14
Human Interaction nunca depende de una interfaz particular.

### INV-15
Una acción que requiere aprobación no puede ejecutarse antes de una resolución válida.

## UI Invariants

### INV-16
`AgentCore` puede ejecutarse sin UI.

### INV-17
La UI observa y presenta el runtime mediante contratos; no contiene la lógica soberana del AgentLoop.

## Observability Invariants

### INV-18
Toda acción significativa produce un evento observable.

### INV-19
Toda decisión crítica debe poder trazarse hasta su actor, contexto y policy relevante.

### INV-20
Todo error operacional pertenece a una categoría conocida.

---

# Article III — Component Sovereignty

El runtime se compone de dominios con responsabilidades explícitas.

```text
Agent Runtime
│
├── AgentCore
├── AgentLoop
├── ModelGateway
├── ContextEngine
├── ToolRuntime
├── PolicyEngine
├── SessionManager
├── HumanInteractionService
├── EventBus
├── ExecutionController
└── CapabilityRegistry
```

## AgentCore

Responsable de representar y coordinar las primitives fundamentales del agente.

No debe absorber responsabilidades de UI, persistencia específica, proveedores o business integrations.

## AgentLoop

Responsable de:

- turn lifecycle;
- model → action → observation cycle;
- continuation;
- completion;
- coordinación de la ejecución cognitiva.

No responsable de:

- autorización;
- rendering;
- almacenamiento concreto;
- APIs específicas de proveedores.

## ModelGateway

Responsable de:

- selección de provider;
- adaptación de mensajes;
- invocación del modelo;
- streaming;
- normalización de respuestas.

## ContextEngine

Responsable de:

- selección;
- ranking;
- composición;
- compaction;
- context budgets;
- provenance.

## ToolRuntime

Responsable de:

- resolver tools/capabilities;
- validar llamadas;
- ejecutar hooks;
- coordinar ejecución;
- devolver resultados normalizados.

## PolicyEngine

Responsable de:

- allow;
- deny;
- constraints;
- approval requirements;
- policy evaluation.

## SessionManager

Responsable de:

- persistencia;
- recuperación;
- branching;
- checkpoints;
- reconstrucción.

## HumanInteractionService

Responsable de:

- representar solicitudes humanas;
- persistir interacciones pendientes;
- recibir resoluciones;
- permitir reanudación.

## EventBus

Responsable de distribuir eventos del runtime a consumidores desacoplados.

## ExecutionController

Responsable de:

- budgets;
- cancellation;
- deadlines;
- runtime limits;
- operational continuation.

## CapabilityRegistry

Responsable de desacoplar la intención de una capacidad de su implementación concreta.

---

# Article IV — Decision Ownership

Cada decisión arquitectónica debe tener un owner definido.

```text
LLM
→ What should I try?

AgentLoop
→ Should another reasoning turn occur?

ContextEngine
→ What should the model know?

ModelGateway
→ How should the selected model be invoked?

ToolRuntime
→ How should an approved action be executed?

PolicyEngine
→ May this action occur?

HumanInteractionService
→ How is required human intervention represented and resolved?

SessionManager
→ What execution history and checkpoints persist?

ExecutionController
→ May the run continue operationally?

CapabilityRegistry
→ What implementation satisfies a requested capability?

UI
→ How is runtime state presented and human input transported?
```

## Ownership Rule

> Ningún componente debe absorber silenciosamente decisiones que pertenecen a otro dominio.

Cuando una decisión no tiene owner claro, debe resolverse arquitectónicamente antes de implementar la feature.

---

# Article V — Lifecycle

El Agent Harness debe utilizar un lifecycle explícito.

```text
CREATED
   ↓
INITIALIZING
   ↓
RUNNING
   ↓
┌────────────────────────┐
│ WAITING_FOR_MODEL      │
│ WAITING_FOR_TOOL       │
│ WAITING_FOR_HUMAN      │
└────────────────────────┘
   ↓
RUNNING
   ↓
COMPLETED
```

Estados alternativos:

```text
PAUSED
FAILED
CANCELLED
EXPIRED
```

Contrato sugerido:

```ts
type AgentRunStatus =
  | "created"
  | "initializing"
  | "running"
  | "waiting_for_model"
  | "waiting_for_tool"
  | "waiting_for_human"
  | "paused"
  | "completed"
  | "failed"
  | "cancelled"
  | "expired";
```

## Lifecycle Rule

El LLM no controla directamente la máquina de estados operacional.

Ejemplo:

```text
WAITING_FOR_HUMAN
        │
        ├── approved  → RUNNING
        ├── rejected  → RUNNING / FAILED
        ├── expired   → FAILED / EXPIRED
        └── cancelled → CANCELLED
```

---

# Article VI — Execution Constitution

Toda acción ejecutable debe atravesar un camino controlado.

```text
Tool Intent
     ↓
Resolve Capability
     ↓
Validate Schema
     ↓
beforeToolCall
     ↓
Policy Evaluation
     ↓
Authorization
     ↓
Human Approval?
     ↓
Execution Budget
     ↓
Sandbox
     ↓
Execute
     ↓
afterToolCall
     ↓
ToolResult
     ↓
Observation
```

## Execution Rules

1. `AgentLoop` no ejecuta directamente side effects.
2. Las tools se ejecutan exclusivamente mediante `ToolRuntime`.
3. Las policies se evalúan antes del side effect.
4. Los hooks pueden intervenir en puntos definidos.
5. Las tool calls independientes pueden paralelizarse.
6. Las operaciones dependientes o con shared mutable state deben respetar secuencialidad.
7. Toda ejecución debe aceptar cancellation cuando técnicamente sea posible.
8. Las operaciones críticas deben ser auditables.

---

# Article VII — Failure Constitution

Todo fallo debe clasificarse.

```text
ValidationError
PolicyError
ToolError
ModelError
ContextError
HumanInteractionError
PersistenceError
InfrastructureError
BudgetExceeded
Cancellation
FatalError
```

Contrato conceptual:

```ts
interface HarnessError {
  category: ErrorCategory;
  recoverable: boolean;
  retryable: boolean;
  retryAfter?: number;
  cause?: Error;
  metadata?: unknown;
}
```

## Failure Examples

```text
HTTP 503
→ transient / retryable

Invalid tool arguments
→ validation / recoverable

Policy denied
→ policy / non-retryable

Tool timeout
→ tool / potentially retryable

Model unavailable
→ model / potentially provider fallback

Budget exceeded
→ stop execution

User cancellation
→ graceful cancellation

Corrupt session
→ infrastructure/fatal
```

## Failure Rule

Retries, fallbacks y recovery deben obedecer políticas determinísticas; no deben depender exclusivamente del modelo.

---

# Article VIII — Human Interaction Constitution

Human-in-the-loop es una capacidad del runtime, no de la TUI.

Tipos mínimos:

```text
Approval
Input
Review
Decision
```

Flujo:

```text
HumanInteractionRequest
        ↓
Persist
        ↓
WAITING_FOR_HUMAN
        ↓
Channel Adapter
        ↓
Human
        ↓
Resolution
        ↓
Resume
```

Canales posibles:

```text
TUI
Web
Slack
Teams
Mobile
Email
API
```

## Human Interaction Rule

> Las interfaces transportan la interacción; el runtime define y persiste su significado.

Una ejecución durable no debe mantener necesariamente un proceso abierto mientras espera intervención humana.

---

# Article IX — Resource and Budget Constitution

Todo run debe tener un `ExecutionBudget`.

```ts
interface ExecutionBudget {
  maxTurns: number;
  maxToolCalls: number;
  maxInputTokens: number;
  maxOutputTokens: number;
  maxCost: number;
  maxRuntimeMs: number;
  maxConcurrentTools: number;
}
```

El `ExecutionController` aplica estos límites.

```text
AgentLoop
   │
   ▼
ExecutionController
   │
   ├── turns?
   ├── tool calls?
   ├── tokens?
   ├── cost?
   ├── runtime?
   └── concurrency?
```

## Budget Rule

El modelo nunca es la única autoridad para determinar cuándo debe detenerse una ejecución.

---

# Article X — Observability Constitution

Todo `AgentRun` debe producir una historia reconstruible.

```text
AgentRun
│
├── run_started
├── context_built
├── model_requested
├── model_response
├── tool_requested
├── policy_evaluated
├── tool_started
├── tool_completed
├── human_requested
├── human_resolved
├── turn_completed
└── run_completed
```

Cada evento debería incluir como mínimo:

```text
eventId
timestamp
runId
sessionId
agentId
traceId
eventType
payload
```

Esta fuente de eventos debe poder alimentar:

```text
Logs
Tracing
Audit
Replay
Analytics
Evals
Cost Analysis
Debugging
UI
```

## Observability Rule

La observabilidad debe surgir de primitives del runtime, no de instrumentación ad hoc dispersa por las aplicaciones.

---

# Article XI — Evolution Constitution

## EVO-01 — Keep the core small

Una nueva capacidad debe intentar implementarse primero mediante:

```text
Extension
Hook
ContextProvider
Tool
Policy
Event Consumer
Adapter
```

antes de modificar `AgentCore`.

## EVO-02 — Multi-agent is not the default solution

No introducir multi-agent para resolver problemas que puedan resolverse mejor mediante tools, context, workflows o una mejor arquitectura single-agent.

## EVO-03 — Deterministic controls do not belong in prompts

Nunca utilizar una instrucción como único mecanismo para seguridad, autorización, budgets o compliance.

## EVO-04 — Declare side effects

Toda integración nueva debe declarar sus side effects y características operacionales.

## EVO-05 — Define failure semantics

Todo componente nuevo debe definir cómo falla, si puede recuperarse y si puede reintentarse.

## EVO-06 — Everything important is observable

Toda capacidad nueva debe definir sus eventos y trazabilidad.

## EVO-07 — Core dependencies require justification

Toda nueva dependencia dentro del core requiere justificación arquitectónica.

## EVO-08 — Breaking contracts require an ADR

Breaking changes sobre contratos fundamentales requieren un Architecture Decision Record.

## EVO-09 — Context has provenance

Nuevas fuentes de contexto deben identificar origen, freshness y reglas de acceso cuando corresponda.

## EVO-10 — Extensions cannot bypass constitutional boundaries

Una extensión no puede saltarse ToolRuntime, PolicyEngine, lifecycle o controles de ejecución para realizar acciones que el core prohibiría.

---

# Article XII — Deterministic vs Agentic Boundary

Esta frontera constituye una de las reglas fundamentales del sistema.

## Agentic Decisions

El LLM puede:

```text
interpret intent
form hypotheses
identify missing information
select an appropriate capability
analyze observations
propose a semantic next action
evaluate whether the semantic objective appears satisfied
```

## Deterministic Decisions

El runtime controla:

```text
identity
authorization
schema validation
permissions
budgets
timeouts
retries
idempotency
sandboxing
rate limits
approval requirements
state transitions
audit requirements
resource limits
```

Arquitectura:

```text
                PROBABILISTIC

                    LLM
                     │
               proposes intent
                     │
                     ▼
════════════════════════════════════
        DETERMINISTIC BOUNDARY
════════════════════════════════════
                     │
                     ▼
              Harness Runtime
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Policy       State       Execution
        │            │            │
        └────────────┼────────────┘
                     ▼
                Real World
```

## Supreme Constitutional Rule

> **Probabilistic systems may propose decisions. Deterministic systems must govern consequences.**

---

# Constitutional Compliance

Toda nueva feature significativa debe responder:

```text
1. Which principles does this affect?
2. Which invariants must remain true?
3. Which component owns the decision?
4. Which contracts change?
5. How does the lifecycle change?
6. What are the failure semantics?
7. What are the security implications?
8. What events are produced?
9. What budgets apply?
10. Does this change the deterministic/agentic boundary?
```

---

# Architecture Decision Records

Cambios significativos deben documentarse mediante ADRs.

Formato mínimo:

```text
ADR-ID
Title
Status
Context
Decision
Alternatives
Consequences
Constitutional Articles Affected
Migration Strategy
```

---

# Final Architecture Doctrine

```text
Model
    proposes

Harness
    governs

Context
    informs

Tools
    act

Policies
    constrain

State
    tracks

Sessions
    persist

Events
    expose

Humans
    intervene

Interfaces
    present

ExecutionController
    limits
```

La Constitution existe para asegurar que, a medida que el harness evoluciona desde un Agent Loop mínimo hacia una plataforma empresarial de agentes, **la inteligencia probabilística permanezca desacoplada del control operacional determinístico**.
