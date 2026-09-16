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


# 3. Architecture Constitution

La **Architecture Constitution** es el documento rector del harness. No describe únicamente cómo está construido el sistema, sino las reglas que deben mantenerse verdaderas a medida que evoluciona.

Debe funcionar como criterio para:

- diseño;
- revisión de arquitectura;
- pull requests;
- incorporación de nuevos componentes;
- extensiones;
- seguridad;
- evolución del runtime;
- evaluación de trade-offs.

La constitución se compone de las siguientes capas:

```text
Architecture Constitution
│
├── Principles
├── Invariants
├── Component Contracts
├── Lifecycle
├── Decision Ownership
├── Failure Semantics
├── Execution Context
├── Security / Policy
├── Budgets / Limits
├── Observability
└── Evolution Rules
```

## 3.1 Los 15 principios arquitectónicos

### 1. Context is a first-class architectural component

El contexto no es simplemente el historial de conversación. Debe ser construido, seleccionado, priorizado, resumido y controlado por el harness.

### 2. The model is replaceable

El runtime nunca debe depender de un proveedor específico. Los modelos se integran mediante adapters.

### 3. Tools are explicit capabilities, not prompt tricks

Toda acción sobre el mundo externo debe representarse mediante contratos explícitos y observables.

### 4. Every action produces observable events

Las operaciones relevantes deben generar eventos que permitan UI, logging, tracing, persistence y audit.

### 5. Side effects pass through policy

Toda acción con efectos secundarios debe pasar por una capa determinística de validación y políticas.

### 6. Agents are configuration over a shared runtime

Los agentes no deben ser aplicaciones independientes. Deben configurarse sobre un runtime común.

### 7. Skills encode reusable procedural knowledge

El conocimiento procedural debe estar separado del core y de la identidad del agente.

### 8. Agent state and session state are different concerns

El estado operativo de una ejecución no debe confundirse con la historia persistente de una sesión.

### 9. Single-agent reliability precedes multi-agent complexity

No se debe introducir multi-agent hasta demostrar confiabilidad de un agente individual.

### 10. The harness owns execution state—not the model

El modelo propone. El harness controla la ejecución, continuidad, límites y estado.

### 11. UI is an adapter, not part of the core

TUI, Web, API, Slack o IDE son interfaces intercambiables alrededor del mismo runtime.

### 12. Events observe; hooks intervene

Los eventos notifican hechos ocurridos. Los hooks permiten intervenir antes o después de operaciones críticas.

### 13. Authorization is deterministic and external to the LLM

El modelo nunca debe ser la fuente de verdad para permisos o autorización.

### 14. Context should be selected, not dumped

Más contexto no implica mejor razonamiento. El harness debe elegir el contexto relevante.

### 15. Automation and agents should share the same execution substrate

Las automatizaciones y los agentes deben utilizar la misma infraestructura de estado, tools, políticas, eventos y ejecución durable.

---

## 3.2 Invariants

Los invariants son reglas que nunca deben violarse.

Ejemplos iniciales:

1. Ningún tool call se ejecuta sin validación.
2. Ningún side effect crítico ocurre sin pasar por policy.
3. El modelo nunca es fuente de autorización.
4. Todo tool result vuelve al loop como observación.
5. Toda ejecución significativa produce eventos.
6. El core puede correr sin TUI.
7. El core puede correr sin una implementación específica de persistencia.
8. Cambiar de proveedor no requiere modificar el agent loop.
9. El estado persistente de una sesión debe permitir recuperación.
10. Un error recuperable de una tool no debe destruir automáticamente todo el Agent Run.
11. Ninguna interacción humana depende de una interfaz específica.
12. Todo run debe tener límites de ejecución.
13. Side effects críticos deben ser idempotentes o protegidos contra duplicación.
14. Las políticas de seguridad deben ejecutarse fuera del razonamiento probabilístico del modelo.
15. Toda decisión crítica debe poder ser trazada hasta su contexto, policy y actor.

---

## 3.3 Component Contracts

Cada componente debe tener responsabilidades claramente delimitadas.

```text
AgentLoop
    │
    ├── consumes → AgentState
    ├── consumes → ContextSnapshot
    ├── calls    → ModelProvider
    ├── calls    → ToolRuntime
    └── emits    → AgentEvent
```

```text
ToolRuntime
    │
    ├── receives → ToolCall
    ├── validates
    ├── applies hooks
    ├── evaluates policy
    ├── executes
    └── returns → ToolResult
```

```text
ContextEngine
    │
    ├── AgentConfig
    ├── SessionState
    ├── Skills
    ├── ContextProviders
    └── CurrentTask
          ↓
      ContextSnapshot
```

```text
SessionManager
    │
    ├── persists
    ├── restores
    ├── checkpoints
    ├── branches
    └── reconstructs state
```

```text
HumanInteractionService
    │
    ├── creates requests
    ├── persists pending interactions
    ├── receives resolution
    └── resumes execution
```

---

## 3.4 Agent Lifecycle

El lifecycle completo debe ser explícito:

```text
Agent Created
     ↓
Configuration Loaded
     ↓
Session Loaded / Created
     ↓
Context Prepared
     ↓
Agent Run Started
     ↓
┌────────────────────────┐
│       TURN LOOP        │
│                        │
│ Model                  │
│   ↓                    │
│ Decision               │
│   ↓                    │
│ Tool Calls             │
│   ↓                    │
│ Observations           │
│   ↓                    │
│ Model                  │
└────────────────────────┘
     ↓
Completion / Error / Abort / Pause
     ↓
State Persisted
     ↓
Agent Run Ended
```

Estados sugeridos:

```ts
type AgentRunStatus =
  | "created"
  | "running"
  | "waiting_for_tool"
  | "waiting_for_approval"
  | "waiting_for_human"
  | "paused"
  | "completed"
  | "failed"
  | "cancelled";
```

---

## 3.5 Decision Ownership

Cada decisión debe tener un único owner arquitectónico.

```text
LLM
→ decide qué intentar

AgentLoop
→ decide cuándo continuar el ciclo

ContextEngine
→ decide qué información mostrar al modelo

ToolRuntime
→ decide cómo ejecutar una acción

PolicyEngine
→ decide si la acción puede ejecutarse

SessionManager
→ decide cómo persistir y recuperar estado

HumanInteractionService
→ decide cómo modelar una intervención humana

Scheduler / Workflow Engine
→ decide cuándo reanudar o iniciar ejecución

UI
→ decide cómo presentar información
```

Regla:

> Ningún componente debe absorber decisiones que pertenecen a otro dominio.

---

## 3.6 Failure Semantics

El harness debe clasificar y manejar fallos explícitamente.

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

Ejemplos:

```text
Invalid tool arguments
→ recoverable
→ devolver observación al modelo

HTTP 503
→ transient
→ retry controlado

Policy denied
→ policy
→ no retry automático

Tool timeout
→ tool/transient
→ timeout + retry policy

Model unavailable
→ model/transient
→ provider fallback

Corrupt session
→ infrastructure/fatal
→ detener ejecución
```

---

## 3.7 Execution Context

Toda ejecución debe transportar un contexto operacional consistente.

```ts
interface ExecutionContext {
  runId: string;
  sessionId: string;
  agentId: string;

  user?: UserIdentity;

  environment: "local" | "dev" | "staging" | "prod";

  permissions: PermissionSet;

  abortSignal: AbortSignal;

  traceId: string;
}
```

Debe acompañar:

```text
AgentLoop
 ↓
ToolRuntime
 ↓
PolicyEngine
 ↓
Tools
 ↓
Telemetry
```

---

## 3.8 Security and Policy

La seguridad debe vivir en lógica determinística.

```text
Tool Call
 ↓
Schema Validation
 ↓
Identity
 ↓
Authorization
 ↓
Policy
 ↓
Approval if required
 ↓
Sandbox
 ↓
Execution
```

Clasificación sugerida:

```text
READ ONLY
LOW-RISK WRITE
EXTERNAL SIDE EFFECT
CRITICAL ACTION
```

---

## 3.9 Budgets and Limits

Todo Agent Run debe tener límites explícitos.

```text
maxTurns
maxToolCalls
maxTokens
maxCost
maxRuntime
maxConcurrentTools
```

Ejemplo:

```yaml
limits:
  maxTurns: 25
  maxToolCalls: 50
  maxRuntimeSeconds: 600
  maxCostUsd: 2.00
```

El modelo nunca debe ser la única autoridad para decidir cuándo detener una ejecución.

---

## 3.10 Observability

Toda ejecución debe poder reconstruirse.

```text
Agent Run
├── request
├── context assembled
├── model used
├── model calls
├── tool calls
├── tool results
├── policy decisions
├── human interactions
├── retries
├── errors
├── tokens
├── cost
└── final result
```

---

## 3.11 Evolution Rules

La arquitectura debe evolucionar manteniendo reglas explícitas:

1. No agregar multi-agent antes de single-agent reliability.
2. No introducir nuevos side effects sin policy.
3. No introducir dependencia directa de proveedores dentro del core.
4. No colocar lógica de UI dentro del runtime.
5. No usar prompts para reemplazar controles determinísticos.
6. No agregar memoria persistente sin provenance y ownership.
7. No agregar tool integrations sin contratos observables.
8. Toda nueva capacidad debe definir failure semantics.
9. Toda nueva capacidad debe definir impacto en observability.
10. Toda nueva capacidad debe preservar backward compatibility o justificar explícitamente su ruptura.

---

# 4. Cómo usar la Architecture Constitution durante el libro

La Constitution no es un capítulo aislado.

Cada capítulo debe indicar explícitamente:

```text
Principles affected
Invariants introduced
Contracts created or modified
Lifecycle changes
Decision ownership
Failure semantics
Security implications
Observability requirements
```

Ejemplo:

```text
Chapter 5 — Tool Runtime

Principles:
#3 Tools are explicit capabilities
#4 Every action produces observable events
#5 Side effects pass through policy

New invariants:
- Every ToolCall is validated.
- Every ToolResult is represented explicitly.

New contracts:
Tool
ToolRegistry
ToolExecutor

Lifecycle change:
Agent can now transition from model decision to external action.

New failure semantics:
ToolValidationError
ToolExecutionError
ToolTimeout
```

Esto convierte la Architecture Constitution en una herramienta viva para desarrollar el harness.


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
