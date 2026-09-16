# Construyendo un Agent Harness
## Del ciclo de decisión a una plataforma empresarial de agentes

---

# 1. Propósito del libro

Este libro propone una construcción progresiva de un **Agent Harness** desde sus fundamentos mínimos hasta una arquitectura empresarial robusta, durable, observable, extensible y preparada para operar agentes especializados y sistemas multi-agente.

La intención no es escribir una referencia enciclopédica, sino construir un mismo proyecto paso a paso.

Cada capítulo introduce una capacidad nueva, explica por qué hace falta, define la decisión arquitectónica correspondiente, implementa el componente y deja deliberadamente una limitación que será resuelta en el capítulo siguiente.

---

# 2. Objetivo pedagógico

El lector debe pasar por cuatro niveles de madurez:

```text
LEVEL 1
Build an Agent

LEVEL 2
Build a Reliable Harness

LEVEL 3
Build an Extensible Platform

LEVEL 4
Build an Enterprise Agent Operating Layer
```

La progresión general será:

```text
LLM
 ↓
LLM + Prompt
 ↓
LLM + Tools
 ↓
Context-Aware Agent
 ↓
Persistent Agent Harness
 ↓
Policies + Observability
 ↓
Smart Automation
 ↓
Specialized Agents
 ↓
Multi-Agent Orchestration
 ↓
Enterprise Agent Operating Layer
```

---

# 3. Principios arquitectónicos

## 3.1 Principios

1. **Context is a first-class architectural component.**
2. **The model is replaceable.**
3. **Tools are explicit capabilities, not prompt tricks.**
4. **Every action produces observable events.**
5. **Side effects pass through policy.**
6. **Agents are configuration over a shared runtime.**
7. **Skills encode reusable procedural knowledge.**
8. **Agent state and session state are different concerns.**
9. **Single-agent reliability precedes multi-agent complexity.**
10. **The harness owns execution state—not the model.**
11. **UI is an adapter, not part of the core.**
12. **Events observe; hooks intervene.**
13. **Authorization is deterministic and external to the LLM.**
14. **Context should be selected, not dumped.**
15. **Automation and agents should share the same execution substrate.**

## 3.2 Marcos complementarios

```text
Principles
    ↓
Invariants
    ↓
Component Contracts
    ↓
Lifecycle
    ↓
Decision Ownership
    ↓
Failure Semantics
    ↓
Execution Context
    ↓
Security / Policy
    ↓
Budgets / Limits
    ↓
Observability
    ↓
Evolution Rules
```

---

# PARTE I — FUNDAMENTOS

## Capítulo 1. Qué es realmente un Agent Harness

### Objetivo
Separar claramente los conceptos de LLM, agent, harness, aplicación, tool, context y state.

### Contenido
- LLM vs Agent vs Harness.
- Por qué un chatbot no es un agente.
- El ciclo decisión → acción → observación.
- Responsabilidades del harness.
- Arquitectura mínima.
- Qué pertenece al core y qué no.

### Arquitectura inicial

```text
User
 ↓
Agent Loop
 ↓
Model
 ↓
Tool Call
 ↓
Tool
 ↓
Observation
 ↓
Model
```

### Resultado del capítulo
El lector debe poder explicar por qué el modelo no es el agente completo.

---

## Capítulo 2. Diseñando el núcleo

### Objetivo
Definir las primeras abstracciones del sistema.

### Componentes

```text
Agent
AgentState
AgentConfig
AgentLoop
AgentMessage
AgentEvent
```

### Temas
- separación de responsabilidades;
- composición sobre herencia;
- dependency inversion;
- state vs execution;
- por qué evitar una clase `Agent` gigante.

### Arquitectura

```text
Agent Definition
      │
      ▼
Agent State
      │
      ▼
Pre-processing / Validation
      │
      ▼
Agent Loop
      │
      ▼
Events / State Updates
```

### Resultado
Primer `agent-core`.

---

# PARTE II — EL CICLO DEL AGENTE

## Capítulo 3. El Agent Loop

### Objetivo
Construir el corazón del harness.

### Implementación inicial

```ts
while (!done) {
  context = buildContext();
  response = model.generate(context);

  if (response.toolCalls) {
    executeTools();
    continue;
  }

  return response;
}
```

### Temas
- agent run;
- turn;
- continuation;
- observation;
- final response;
- tool results;
- errores;
- stop conditions.

### Flujo

```text
Model decides
     ↓
Tool Call
     ↓
Harness executes
     ↓
Tool Result
     ↓
Model re-evaluates
     ↓
Continue?
```

---

## Capítulo 4. Mensajes independientes del modelo

### Objetivo
Desacoplar el harness de proveedores específicos.

### Concepto

```text
AgentMessage
     ↓
Model Adapter
     ↓
OpenAI / Anthropic / Gemini / Local
```

### Temas
- contrato interno estable;
- provider adapters;
- streaming;
- metadata propia;
- switching de modelos.

### Resultado

Crear:

```ts
interface ModelProvider
```

e implementar al menos dos providers.

---

# PARTE III — HERRAMIENTAS Y ACCIÓN

## Capítulo 5. Tool Runtime

### Objetivo
Permitir que el modelo interactúe con el mundo externo.

### Componentes

```text
Tool
ToolRegistry
ToolCall
ToolResult
ToolExecutor
```

### Contrato

```ts
interface Tool<TInput = unknown, TOutput = unknown> {
  name: string;
  description: string;
  schema: unknown;
  execute(input: TInput): Promise<TOutput>;
}
```

### Tools iniciales

```text
read_file
write_file
shell
```

### Resultado
El agente puede observar y modificar un entorno real.

---

## Capítulo 6. Paralelismo, dependencias y side effects

### Objetivo
Controlar el modo de ejecución de tool calls.

### Temas
- independencia;
- dependencia de datos;
- shared mutable state;
- side effects;
- paralelismo seguro;
- secuencialidad obligatoria.

### Ejemplo paralelo

```text
read(a) ─────┐
read(b) ─────┼──► aggregate
read(c) ─────┘
```

### Ejemplo secuencial

```text
write
 ↓
test
 ↓
inspect result
```

### Metadata operacional

```ts
execution: {
  mode: "parallel" | "sequential";
  sideEffect: "none" | "local" | "external";
}
```

---

# PARTE IV — CONTEXT ENGINEERING

## Capítulo 7. Qué ve realmente el modelo

### Objetivo
Construir el primer `ContextBuilder`.

### Contexto inicial

```text
System
Agent Instructions
Conversation
Tool Results
```

### Evolución

```text
System Instructions
+
Agent Instructions
+
Project Context
+
Relevant Skills
+
Working Memory
+
Recent History
+
Tool Observations
+
Current Request
```

### Resultado
El lector entiende que el contexto es una decisión del harness.

---

## Capítulo 8. Context Providers

### Objetivo
Convertir el contexto en una arquitectura extensible.

### Contrato

```ts
interface ContextProvider {
  getContext(
    request: ContextRequest
  ): Promise<ContextBlock[]>;
}
```

### Ejemplos

```text
GitContextProvider
FileContextProvider
DocumentationProvider
PolicyProvider
DatabaseContextProvider
UserMemoryProvider
```

### Temas
- Context Engineering vs RAG;
- composición;
- relevancia;
- retrieval selectivo.

---

## Capítulo 9. Context budgets y compaction

### Objetivo
Controlar el crecimiento del contexto.

### Componentes

```text
ContextBudget
Summarizer
Compactor
```

### Políticas

```text
pin
summarize
discard
retrieve again
```

### Problemas
- ventanas de contexto;
- resultados gigantes;
- conversaciones largas;
- información irrelevante;
- costo.

---

# PARTE V — ESTADO Y MEMORIA

## Capítulo 10. Agent State vs Session State

### Objetivo
Separar ejecución actual de historia persistente.

### Agent State

```text
messages
model
tools
system prompt
streaming state
errors
```

### Session State

```text
history
branches
checkpoints
metadata
persistent events
```

### Resultado

> Agent State ≠ Session State

---

## Capítulo 11. Event sourcing y sesiones

### Objetivo
Persistir la ejecución como una secuencia de hechos.

```text
Session
 ├── message
 ├── tool_call
 ├── tool_result
 ├── policy_decision
 ├── approval
 └── checkpoint
```

### Temas
- replay;
- debugging;
- auditabilidad;
- reconstrucción de estado.

---

## Capítulo 12. Branching y recuperación

### Objetivo
Permitir bifurcaciones y recuperación.

```text
Session
 ├── branch A
 └── branch B
```

### Temas
- checkpoints;
- resume;
- recovery;
- fork;
- rollback conceptual.

---

# PARTE VI — ARQUITECTURA BASADA EN EVENTOS

## Capítulo 13. Event-driven Agent Core

### Objetivo
Desacoplar el core de sus efectos secundarios.

### Eventos

```text
agent_start
turn_start
message_start
message_update
message_end
tool_execution_start
tool_execution_update
tool_execution_end
turn_end
agent_end
```

### Arquitectura

```text
Agent Core
   ↓ events
TUI
Logger
Session
Telemetry
Audit
```

---

## Capítulo 14. Events vs Hooks

### Objetivo
Separar observación de intervención.

```text
Event
"algo ocurrió"

Hook
"algo está por ocurrir"
```

### Hooks

```text
beforeToolCall
afterToolCall
```

### Casos de uso

Events:
- UI;
- logging;
- telemetry;
- audit.

Hooks:
- authorization;
- validation;
- approval;
- sandbox;
- redaction.

---

# PARTE VII — SEGURIDAD Y GOBIERNO

## Capítulo 15. Policy Engine

### Objetivo
Interponer gobierno determinístico antes de ejecutar acciones.

```text
Tool Call
 ↓
Validation
 ↓
Policy
 ↓
Authorization
 ↓
Approval
 ↓
Execution
```

### Clasificación

```text
Read
Low-risk Write
External Side Effect
Critical
```

---

## Capítulo 16. Identity, permissions y capabilities

### Objetivo
Separar identidad, permisos y capacidades.

```text
UserIdentity
AgentIdentity
Role
Permission
Capability
```

### Evolución

De:

```text
Agent has tool X
```

a:

```text
Agent has capability X
```

---

## Capítulo 17. Sandboxing

### Objetivo
Aislar la ejecución de herramientas.

### Límites
- filesystem;
- network;
- environment variables;
- credentials;
- CPU;
- memory;
- subprocesses.

```text
Tool Runtime
 ↓
Sandbox
 ↓
Execution
```

---

# PARTE VIII — HUMAN-IN-THE-LOOP

## Capítulo 18. Human Interaction como componente

### Objetivo
Diseñar HITL sin acoplarlo a ninguna interfaz.

### Servicio

```text
HumanInteractionService
```

### Tipos

```text
Approval
Input
Review
Decision
```

### Regla

> Las interfaces transportan la interacción; el runtime define y persiste su significado.

---

## Capítulo 19. Pausar y reanudar agentes

### Objetivo
Introducir durable execution.

### Estado

```text
WAITING_FOR_HUMAN
```

### Flujo

```text
Running
 ↓
Waiting for Human
 ↓
Persist
 ↓
Process may die
 ↓
Human responds
 ↓
Resume
```

### Temas
- checkpoints;
- durable execution;
- resume;
- approval expiry;
- rejection;
- external channels.

---

# PARTE IX — INTERFACES

## Capítulo 20. CLI y TUI

### Objetivo
Construir una interfaz desacoplada del core.

```text
Agent Core
 ↓ events
UI Adapter
 ↓
TUI
```

### Temas
- adapter pattern;
- streaming;
- tool rendering;
- user input;
- steering.

---

## Capítulo 21. API y Web UI

### Objetivo
Demostrar que el mismo runtime funciona en múltiples interfaces.

```text
REST
WebSocket
Web UI
```

### Regla
El core no debe modificarse.

---

# PARTE X — RELIABILITY

## Capítulo 22. Failure semantics

### Objetivo
Definir cómo se comporta el sistema frente a errores.

```text
Validation
Recoverable
Transient
Policy
Tool
Model
Infrastructure
Fatal
```

### Ejemplos

```text
Invalid tool args
→ recoverable

HTTP 503
→ transient

Policy denied
→ no retry

Corrupt session
→ fatal/infrastructure
```

---

## Capítulo 23. Retries, timeouts y circuit breakers

### Objetivo
Agregar resiliencia determinística.

```text
RetryPolicy
TimeoutPolicy
CircuitBreaker
```

### Regla
Los retries operacionales no deben depender del criterio del LLM.

---

## Capítulo 24. Idempotencia

### Objetivo
Evitar duplicar side effects.

### Casos

```text
send_email
charge_card
create_order
deploy
```

### Arquitectura

```text
Tool Call
 ↓
Idempotency Key
 ↓
Execution Ledger
 ↓
Already executed?
```

---

## Capítulo 25. Budgets y límites

### Objetivo
Controlar loops, costo y uso de recursos.

```text
maxTurns
maxToolCalls
maxTokens
maxCost
maxRuntime
maxConcurrentTools
```

---

# PARTE XI — OBSERVABILIDAD Y EVALUACIÓN

## Capítulo 26. Tracing

### Objetivo
Reconstruir completamente una ejecución.

```text
Run
├── context
├── model calls
├── tool calls
├── policy decisions
├── retries
├── tokens
├── cost
└── result
```

---

## Capítulo 27. Evals

### Objetivo
Medir confiabilidad del sistema.

### Métricas
- task success;
- tool selection;
- argument accuracy;
- policy compliance;
- context relevance;
- latency;
- cost;
- recovery;
- intervention rate.

---

# PARTE XII — SKILLS Y EXTENSIBILIDAD

## Capítulo 28. Skills

### Objetivo
Separar conocimiento procedural de identidad y tools.

```text
Agent = who
Tools = what
Skills = how
Context = knowledge
Policies = allowed
```

### Formato

```text
skills/
  debugging/
    SKILL.md
```

---

## Capítulo 29. Extensions

### Objetivo
Convertir el harness en plataforma extensible.

```ts
interface HarnessExtension {
  name: string;
  setup(api: HarnessAPI): Promise<void>;
}
```

### Puede registrar

```text
tools
hooks
context providers
policies
commands
UI
```

---

# PARTE XIII — AUTOMATIZACIÓN

## Capítulo 30. Del agente interactivo al agente autónomo

### Objetivo
Permitir ejecuciones sin usuario presente.

```text
Scheduler
Triggers
Events
Queues
```

### Temas
- cron;
- event triggers;
- external events;
- workers.

---

## Capítulo 31. Durable workflows

### Objetivo
Combinar agentes y workflows determinísticos.

```text
Agent Execution
+
Workflow Engine
+
State Persistence
```

### Distinción

```text
Deterministic
validation
permissions
routing
retries
timeouts
approvals
schemas
budgets

Agentic
interpret intent
form hypothesis
select tool
analyze observations
choose next semantic step
```

---

# PARTE XIV — MULTI-AGENT

## Capítulo 32. Cuándo NO usar multi-agent

### Objetivo
Evitar complejidad prematura.

```text
bad single agent
×
multiple agents
=
bigger problem
```

### Regla

> Single-agent reliability precedes multi-agent complexity.

---

## Capítulo 33. Delegación

### Objetivo
Agregar agentes especializados.

```text
Supervisor
 ↓
Specialist Agent
```

### Temas
- task delegation;
- context boundaries;
- return contracts;
- specialization.

---

## Capítulo 34. Agent-to-Agent communication

### Objetivo
Definir contratos explícitos entre agentes.

```text
Task
Result
Context
Capabilities
```

### Regla
No compartir estado arbitrariamente.

---

# PARTE XV — ENTERPRISE AGENT PLATFORM

## Capítulo 35. Capability Registry

### Objetivo
Desacoplar intención de implementación.

```text
customer.lookup
```

puede resolverse mediante:

```text
REST API
MCP
Local Tool
Remote Agent
Workflow
```

### Arquitectura

```text
Agent
 ↓
Capability
 ↓
Capability Resolver
 ↓
Tool / MCP / API / Agent / Workflow
```

---

## Capítulo 36. Enterprise Context Layer

### Objetivo
Organizar contexto empresarial reusable.

```text
Company
Departments
Processes
Products
Customers
Policies
Projects
People / Roles
```

### Temas
- ownership;
- freshness;
- provenance;
- ranking;
- access control.

---

## Capítulo 37. Governance

### Objetivo
Agregar controles empresariales completos.

### Temas
- audit;
- identity;
- RBAC / ABAC;
- approvals;
- compliance;
- data classification;
- retention;
- secrets;
- model governance;
- cost governance.

---

# PARTE XVI — PROYECTO FINAL

## Capítulo 38. Construyendo el harness completo

### Objetivo
Integrar todas las piezas.

```text
                Interfaces
                    │
                    ▼
              Agent Gateway
                    │
                    ▼
              Agent Runtime
             /      |       \
            /       |        \
     Context     Model      Tools
       Engine    Gateway    Runtime
         │          │          │
         │          │       Policies
         │          │          │
         ▼          ▼          ▼
      Context      LLM      Capabilities

Cross-cutting:
Sessions
Human Interaction
Events
Telemetry
Audit
Budgets
Evals
Security
```

---

# 4. Estructura pedagógica de cada capítulo

Cada capítulo debe seguir el mismo patrón:

```text
1. Problema
2. Concepto
3. Decisión arquitectónica
4. Contrato
5. Implementación
6. Ejemplo
7. Failure cases
8. Tests
9. Qué todavía NO resolvemos
10. Próximo incremento
```

Ejemplo:

```text
Chapter 5 — Tool Runtime

Problem
LLM wants to interact with the world.

Concept
Explicit tool contract.

Architecture
Model → ToolCall → ToolRuntime → ToolResult.

Code
Implement registry and executor.

Test
read_file works.

Failure
Invalid args.

Not yet
Permissions.

Next
Policy engine.
```

---

# 5. Organización por niveles de madurez

## Level 1 — Build an Agent

Capítulos 1–9

```text
LLM
+
Agent Loop
+
Tools
+
Context
```

## Level 2 — Build a Reliable Harness

Capítulos 10–25

```text
State
+
Sessions
+
Events
+
Policies
+
Human Interaction
+
Reliability
```

## Level 3 — Build an Extensible Platform

Capítulos 26–31

```text
Observability
+
Evals
+
Skills
+
Extensions
+
Automation
```

## Level 4 — Build an Enterprise Agent Operating Layer

Capítulos 32–38

```text
Multi-Agent
+
Capabilities
+
Enterprise Context
+
Governance
+
Operating Platform
```

---

# 6. Relación entre libro y repositorio

El libro y el código deben evolucionar juntos.

```text
chapter-03 → v0.1 agent loop
chapter-05 → v0.2 tools
chapter-09 → v0.3 context
chapter-14 → v0.4 events/hooks
chapter-19 → v0.5 durable HITL
chapter-25 → v0.6 reliability
chapter-29 → v0.7 extensibility
chapter-31 → v0.8 automation
chapter-34 → v0.9 multi-agent
chapter-38 → v1.0 enterprise harness
```

Cada capítulo debe corresponder a una versión ejecutable del repositorio.

---

# 7. Manifiesto final del libro

```text
Model
    proposes

Harness
    governs

Tools
    act

Context
    informs

State
    remembers

Policies
    constrain

Events
    expose

Sessions
    persist

Humans
    intervene

UI
    presents
```

La meta final no es construir un chatbot sofisticado ni un coding agent específico.

La meta es construir una **Agent Operating Layer** reutilizable sobre la cual puedan vivir múltiples agentes, automatizaciones y experiencias de usuario sin acoplar el runtime a un proveedor, una interfaz o un dominio específico.
