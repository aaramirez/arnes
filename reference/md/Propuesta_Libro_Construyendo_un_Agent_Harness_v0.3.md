# Construyendo un Agent Harness
## Del Agent Loop a una Enterprise Agent Operating Layer
### Propuesta de libro — Versión 0.3

---

# Propósito

Este libro enseña a construir un Agent Harness desde cero mediante una implementación incremental.

No se plantea como una enciclopedia de frameworks de agentes. El lector construye un mismo runtime durante todo el libro.

Cada nueva capacidad aparece porque la versión anterior encuentra una limitación real.

La evolución es:

```text
Model
 ↓
Agent Loop
 ↓
Tools
 ↓
Context
 ↓
State & Sessions
 ↓
Events & Hooks
 ↓
Policy & Security
 ↓
Human Interaction
 ↓
Durable Execution
 ↓
Reliability
 ↓
Observability & Evals
 ↓
Extensions
 ↓
Automation
 ↓
Multi-Agent
 ↓
Enterprise Agent Operating Layer
```

El libro utiliza como documento rector independiente:

> **Agent Harness Architecture Constitution**

La Constitution define principios, invariants, component sovereignty, decision ownership, lifecycle, execution rules, failure semantics, human interaction, budgets, observability, evolution rules y la frontera entre decisiones agentic y deterministic.

---

# Cómo se utiliza la Architecture Constitution

La Constitution no es simplemente material introductorio.

Cada capítulo debe identificar:

```text
Constitutional Impact

Principles affected
Invariants introduced or preserved
Component ownership
Contracts introduced or modified
Lifecycle changes
Failure semantics
Security implications
Observability requirements
Budget implications
Agentic vs deterministic boundary
```

Esto permite que arquitectura, código y libro evolucionen juntos.

---

# LEVEL 0 — ARCHITECTURAL FOUNDATION

## Capítulo 0. La Architecture Constitution

### Problema

Los sistemas de agentes evolucionan rápidamente y tienden a acumular responsabilidades dentro de clases, prompts e integraciones.

### Objetivo

Establecer antes de escribir el runtime las reglas que gobernarán su evolución.

### Contenido

- por qué necesitamos una Architecture Constitution;
- diferencia entre principles e invariants;
- los 15 principios;
- component sovereignty;
- decision ownership;
- lifecycle;
- execution constitution;
- failure constitution;
- HITL constitution;
- budgets;
- observability;
- evolution rules;
- deterministic vs agentic boundary;
- ADRs;
- architecture compliance.

### Regla fundamental

> Probabilistic systems may propose decisions. Deterministic systems must govern consequences.

---

# LEVEL 1 — BUILD AN AGENT

# PARTE I — Fundamentos

## Capítulo 1. Qué es realmente un Agent Harness

### Problema
Un LLM puede responder, pero todavía no puede operar como un agente confiable.

### Conceptos
- LLM;
- Agent;
- Harness;
- Runtime;
- Tool;
- Context;
- State.

### Arquitectura

```text
User
 ↓
Agent Loop
 ↓
Model
 ↓
Decision
```

### Resultado
Primer modelo mental del sistema.

---

## Capítulo 2. Diseñando el Agent Core

### Componentes

```text
Agent
AgentConfig
AgentState
AgentMessage
AgentEvent
AgentLoop
```

### Decisión central

Separar state/configuration de ejecución.

```text
Agent Definition
      ↓
Agent State
      ↓
Validation / Preparation
      ↓
Agent Loop
```

### Temas
- responsibility boundaries;
- dependency inversion;
- composition;
- testability;
- evitar el God Object `Agent`.

---

# PARTE II — El ciclo cognitivo

## Capítulo 3. Construyendo el Agent Loop

### Flujo

```text
Model
 ↓
Decision
 ↓
Tool?
 ├── no → final
 └── yes
       ↓
     execute
       ↓
   observation
       ↓
     Model
```

### Temas
- AgentRun;
- Turn;
- continuation;
- stop conditions;
- observations.

### Implementación incremental

```ts
while (!done) {
  const context = buildContext();
  const response = await model.generate(context);

  if (!response.toolCalls?.length) {
    return response;
  }

  const results = await executeTools(response.toolCalls);
  addObservations(results);
}
```

---

## Capítulo 4. Model Independence

### Problema
No queremos construir un OpenAI Harness, Anthropic Harness o Gemini Harness.

### Arquitectura

```text
AgentMessage
      ↓
ModelGateway
      ↓
Provider Adapter
      ↓
Model
```

### Componentes

```text
ModelProvider
ModelRequest
ModelResponse
ProviderAdapter
```

### Experimento
Cambiar de provider sin modificar `AgentLoop`.

---

# PARTE III — Tools

## Capítulo 5. Tool Runtime

### Componentes

```text
Tool
ToolCall
ToolResult
ToolRegistry
ToolExecutor
```

### Primeras tools

```text
read_file
write_file
shell
```

### Regla
El AgentLoop nunca ejecuta directamente el mundo externo.

---

## Capítulo 6. Parallelism, Dependencies and Side Effects

### Paralelo

```text
read(a) ───┐
read(b) ───┼─→ observations
read(c) ───┘
```

### Secuencial

```text
write
 ↓
test
 ↓
inspect
```

### Metadata

```text
executionMode
sideEffect
risk
```

---

# PARTE IV — Context Engineering

## Capítulo 7. Qué ve realmente el modelo

### Construcción inicial

```text
System
+
Agent Instructions
+
Conversation
+
Tool Results
```

### Evolución

```text
Project Context
Skills
Working Memory
Retrieved Knowledge
Policies
```

---

## Capítulo 8. Context Providers

### Contrato

```ts
interface ContextProvider {
  getContext(request: ContextRequest): Promise<ContextBlock[]>;
}
```

### Providers

```text
File
Git
Documentation
Database
Project
Policy
Memory
RAG
```

### Tema central
RAG es una fuente de contexto; Context Engineering es la disciplina completa.

---

## Capítulo 9. Context Budgets and Compaction

### Problemas
- token limits;
- costo;
- tool output;
- conversaciones largas;
- stale context.

### Estrategias

```text
pin
rank
summarize
compact
discard
retrieve again
```

### Milestone
**v0.3 — Context-aware Agent**

---

# LEVEL 2 — BUILD A RELIABLE HARNESS

# PARTE V — State, Sessions and Persistence

## Capítulo 10. Agent State vs Session State

```text
AgentState
≠
SessionState
```

### AgentState
Estado operacional actual.

### SessionState
Historia persistente.

---

## Capítulo 11. Session Manager and Event History

### Historia

```text
Session
├── message
├── model_response
├── tool_call
├── tool_result
├── policy_decision
└── checkpoint
```

### Capacidades
- replay;
- debugging;
- persistence;
- reconstruction.

---

## Capítulo 12. Branching, Checkpoints and Recovery

```text
Session
 ├── Branch A
 └── Branch B
```

### Temas
- checkpoint;
- fork;
- resume;
- recovery.

---

# PARTE VI — Events and Hooks

## Capítulo 13. Event-Driven Agent Core

```text
Agent Core
    │
    └── Events
         ├── UI
         ├── Logs
         ├── Sessions
         ├── Audit
         └── Telemetry
```

### Eventos
- agent_start;
- turn_start;
- model events;
- tool events;
- agent_end.

---

## Capítulo 14. Events vs Hooks

```text
EVENT
Something happened.

HOOK
Something is about to happen
and may be intercepted.
```

### Hooks
- beforeToolCall;
- afterToolCall;
- beforeModelCall;
- context hooks.

### Milestone
**v0.4 — Event-driven Harness**

---

# PARTE VII — Governance and Security

## Capítulo 15. Policy Engine

```text
Tool Intent
 ↓
Validation
 ↓
Policy
 ↓
Authorization
 ↓
Execution
```

### Resultados

```text
ALLOW
DENY
REQUIRE_APPROVAL
ALLOW_WITH_CONSTRAINTS
```

---

## Capítulo 16. Identity, Permissions and Capabilities

### Conceptos

```text
UserIdentity
AgentIdentity
Role
Permission
Capability
```

### Evolución

```text
Agent has Tool
      ↓
Agent has Capability
```

---

## Capítulo 17. Sandboxed Execution

### Controles
- filesystem;
- network;
- credentials;
- environment;
- processes;
- CPU;
- memory.

```text
ToolRuntime
 ↓
Policy
 ↓
Sandbox
 ↓
Execution
```

---

# PARTE VIII — Human-in-the-Loop

## Capítulo 18. Human Interaction as a Runtime Primitive

### Tipos

```text
Approval
Input
Review
Decision
```

### Servicio

```text
HumanInteractionService
```

### Regla
Human Interaction no depende del TUI.

---

## Capítulo 19. Durable Human Interaction

```text
RUNNING
 ↓
WAITING_FOR_HUMAN
 ↓
Persist
 ↓
Process can terminate
 ↓
Human responds
 ↓
Restore
 ↓
RUNNING
```

### Canales

```text
TUI
Web
Slack
Teams
Email
Mobile
API
```

### Milestone
**v0.5 — Durable HITL Harness**

---

# PARTE IX — Interfaces

## Capítulo 20. CLI and TUI

```text
AgentCore
 ↓ events
UIAdapter
 ↓
TUI
```

### Temas
- terminal rendering;
- streaming;
- steering;
- tool displays;
- input.

### Caso de estudio
Analizar la separación de TUI utilizada por Pi y qué decisiones vale la pena reutilizar.

---

## Capítulo 21. API, Web and External Channels

### Interfaces

```text
REST
WebSocket
Web
Slack
IDE
```

### Objetivo
Demostrar que ninguna requiere modificar AgentCore.

---

# PARTE X — Reliability Engineering

## Capítulo 22. Failure Semantics

### Taxonomía

```text
Validation
Policy
Tool
Model
Context
Persistence
Infrastructure
Budget
Cancellation
Fatal
```

### Propiedades

```text
recoverable
retryable
fatal
```

---

## Capítulo 23. Retries, Timeouts and Circuit Breakers

```text
RetryPolicy
TimeoutPolicy
CircuitBreaker
FallbackPolicy
```

### Regla
Operational recovery is deterministic.

---

## Capítulo 24. Idempotency and Execution Ledger

```text
ToolCall
 ↓
Idempotency Key
 ↓
Execution Ledger
 ↓
Execute / Reuse
```

### Casos
- email;
- payments;
- orders;
- deploy;
- external API mutations.

---

## Capítulo 25. Execution Controller and Budgets

### Límites

```text
maxTurns
maxToolCalls
maxTokens
maxCost
maxRuntime
maxConcurrency
```

### Milestone
**v0.6 — Reliable Harness**

---

# LEVEL 3 — BUILD AN EXTENSIBLE PLATFORM

# PARTE XI — Observability and Evals

## Capítulo 26. Tracing an Agent Run

```text
Run
├── context
├── model
├── tools
├── policies
├── human interactions
├── errors
├── cost
└── result
```

### Correlation
- runId;
- sessionId;
- agentId;
- traceId.

---

## Capítulo 27. Agent Evals

### Métricas
- task success;
- tool selection accuracy;
- tool argument accuracy;
- policy compliance;
- context relevance;
- latency;
- cost;
- recovery;
- human intervention rate.

---

# PARTE XII — Skills and Extensibility

## Capítulo 28. Skills

```text
Agent = Who
Tools = What
Skills = How
Context = Knowledge
Policies = Allowed
```

### Formato
`SKILL.md`

### Temas
- discovery;
- loading;
- relevance;
- procedural knowledge.

---

## Capítulo 29. Extensions

### Extension API

```text
registerTool
registerHook
registerContextProvider
registerPolicy
registerCommand
registerEventConsumer
```

### Regla
Extensiones no pueden saltarse la Constitution.

### Milestone
**v0.7 — Extensible Agent Platform**

---

# PARTE XIII — Automation

## Capítulo 30. From Interactive Agent to Autonomous Execution

### Triggers

```text
User
Schedule
Event
Webhook
Queue
External System
```

### Componentes

```text
Scheduler
TriggerRegistry
Worker
Queue
```

---

## Capítulo 31. Durable Workflows

### Arquitectura

```text
Workflow
  │
  ├── Deterministic Step
  ├── Agentic Step
  ├── Human Step
  └── External Action
```

### Tema central
Cuándo usar workflow determinístico y cuándo permitir decisión agentic.

### Milestone
**v0.8 — Agent Automation Runtime**

---

# LEVEL 4 — BUILD AN ENTERPRISE AGENT OPERATING LAYER

# PARTE XIV — Multi-Agent

## Capítulo 32. Why Not Multi-Agent Yet?

### Problema

```text
Unreliable Agent
×
More Agents
=
More Uncertainty
```

### Criterios para justificar multi-agent
- context isolation;
- specialized capabilities;
- independent lifecycle;
- security boundary;
- meaningful delegation.

---

## Capítulo 33. Delegation

```text
Supervisor
    ↓ Task Contract
Specialist
    ↓ Result Contract
Supervisor
```

### Temas
- task boundaries;
- context boundaries;
- capabilities;
- result contracts.

---

## Capítulo 34. Agent-to-Agent Protocols

### Contratos

```text
Task
Context
Capabilities
Constraints
Result
Evidence
```

### Milestone
**v0.9 — Multi-Agent Runtime**

---

# PARTE XV — Enterprise Platform

## Capítulo 35. Capability Registry

### Intención

```text
customer.lookup
```

### Resolución

```text
CapabilityResolver
├── Local Tool
├── REST API
├── MCP
├── Workflow
└── Remote Agent
```

El agente conoce la capacidad; no necesariamente su implementación.

---

## Capítulo 36. Enterprise Context Layer

### Context domains

```text
Company
Organization
People / Roles
Processes
Policies
Products
Customers
Projects
Systems
Knowledge
```

### Propiedades
- provenance;
- freshness;
- ownership;
- access control;
- relevance;
- classification.

---

## Capítulo 37. Enterprise Governance

### Dominios

```text
Identity
RBAC / ABAC
Audit
Approvals
Data Classification
Secrets
Retention
Compliance
Model Governance
Cost Governance
Agent Governance
```

---

# PARTE XVI — Proyecto Final

## Capítulo 38. Building the Enterprise Agent Harness

### Arquitectura

```text
                         Interfaces
                             │
                             ▼
                       Agent Gateway
                             │
                             ▼
                       Agent Runtime
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
    Context Engine      Model Gateway      Tool Runtime
          │                  │                  │
          ▼                  ▼                  ▼
 Context Providers        Models          Capability Layer
                                                │
                                      ┌─────────┼──────────┐
                                      ▼         ▼          ▼
                                    Tools      MCP      Workflows

Cross-cutting:

Sessions
Policy Engine
Execution Controller
Human Interaction
Event Bus
Telemetry
Audit
Evals
Security
Budgets
```

### Resultado

**v1.0 — Enterprise Agent Operating Layer**

---

# PARTE XVII — Operating the Platform

## Capítulo 39. Architecture Compliance

### Preguntas obligatorias para cada feature

```text
Which principles are affected?
Which invariants must remain true?
Who owns the decision?
Which contracts change?
How does lifecycle change?
How can it fail?
What are the security implications?
What events are produced?
What budgets apply?
Does it cross the deterministic boundary?
```

---

## Capítulo 40. Architecture Decision Records

### Formato

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

### Objetivo
Permitir evolución controlada sin convertir la Constitution en un documento estático.

---

# Estructura pedagógica de cada capítulo

Cada capítulo seguirá:

```text
1. Problem
2. Why the current harness is insufficient
3. Concept
4. Constitutional impact
5. Architecture decision
6. Component contracts
7. Implementation
8. Running example
9. Failure cases
10. Tests
11. Observability
12. Security implications
13. What we deliberately do NOT solve yet
14. Next increment
```

---

# Evolución del repositorio

```text
chapter-03 → v0.1 Agent Loop
chapter-04 → v0.1.1 Model Independence
chapter-06 → v0.2 Tool Runtime
chapter-09 → v0.3 Context-Aware Agent
chapter-14 → v0.4 Event-Driven Harness
chapter-19 → v0.5 Durable HITL
chapter-25 → v0.6 Reliable Harness
chapter-29 → v0.7 Extensible Platform
chapter-31 → v0.8 Automation Runtime
chapter-34 → v0.9 Multi-Agent Runtime
chapter-38 → v1.0 Enterprise Agent Operating Layer
```

Cada milestone debe existir como tag o branch ejecutable.

---

# Proyecto transversal del libro

El lector construirá un único harness durante todo el libro.

```text
agent-harness/
│
├── apps/
│   ├── cli/
│   ├── api/
│   └── web/
│
├── packages/
│   ├── ai/
│   ├── agent-core/
│   ├── context/
│   ├── tools/
│   ├── policies/
│   ├── sessions/
│   ├── human-interaction/
│   ├── execution/
│   ├── events/
│   ├── telemetry/
│   ├── evals/
│   ├── skills/
│   ├── extensions/
│   └── tui/
│
├── agents/
├── skills/
├── policies/
├── evals/
├── docs/
│   ├── ARCHITECTURE_CONSTITUTION.md
│   └── adr/
│
└── README.md
```

---

# Doctrina final

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

El objetivo final no es construir un chatbot sofisticado ni replicar Pi.

El objetivo es entender las primitives fundamentales de un Agent Harness y construir una infraestructura donde modelos, tools, contexto, personas, automatizaciones y agentes puedan evolucionar sin romper las fronteras arquitectónicas fundamentales.

> **Probabilistic systems may propose decisions. Deterministic systems must govern consequences.**
