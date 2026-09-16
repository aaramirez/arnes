# Reglas Editoriales y Arquitectónicas para el Libro
## Construyendo un Agent Harness con Pseudocódigo, Contratos y Arquitectura Consistente

---

# 1. Propósito

Este documento define las reglas que deben gobernar la escritura del libro **Construyendo un Agent Harness**.

Su objetivo es asegurar que:

- la relación entre componentes sea explícita;
- el pseudocódigo sea consistente;
- ninguna entidad aparezca sin definición previa;
- los contratos e interfaces sean estables;
- las estructuras de datos evolucionen de forma controlada;
- los capítulos construyan incrementalmente sobre la arquitectura anterior;
- la relación entre conceptos, contratos, componentes e implementación sea siempre visible;
- la Architecture Constitution se mantenga como documento rector.

La meta no es producir ejemplos aislados de código. La meta es construir una arquitectura coherente donde cada pieza pueda rastrearse desde su concepto hasta su contrato, interacción y pseudocódigo.

---

# 2. Regla editorial fundamental

Todo concepto debe atravesar siempre esta secuencia:

```text
Concept
  ↓
Contract
  ↓
Pseudocode
  ↓
Interaction
```

Ningún pseudocódigo puede utilizar una entidad que no haya sido definida previamente mediante estructura, contrato, interfaz, enum, evento, command, state o error type.

> **No magic entities.**

Ejemplo incorrecto:

```text
result = executeTool(toolCall)
```

si previamente no se ha definido `ToolCall`, `ToolResult`, `ToolRuntime` y `execute(...)`.

Ejemplo correcto:

```text
STRUCT ToolCall
    id: ToolCallId
    capability: CapabilityId
    arguments: Map<String, Value>
END
```

```text
INTERFACE ToolRuntime
    execute(
        call: ToolCall,
        execution: ExecutionContext
    ) -> ToolResult
END
```

Solo después:

```text
result: ToolResult =
    ToolRuntime.execute(
        call,
        execution
    )
```

---

# 3. Metamodelo común

El libro debe definir un vocabulario canónico al inicio y reutilizarlo durante toda la obra.

## 3.1 Identificadores fundamentales

```text
AgentId
SessionId
RunId
TurnId
ToolCallId
CapabilityId
EventId
TraceId
Timestamp
```

## 3.2 Tipos fundamentales

```text
AgentConfig
AgentState
SessionState
ExecutionContext
AgentMessage
AgentEvent
HarnessError
ExecutionBudget
ContextSnapshot
ModelRequest
ModelResponse
ToolCall
ToolResult
```

## Regla

Estos tipos tienen una definición canónica. No deben redefinirse informalmente en capítulos posteriores. Si cambian, el cambio debe registrarse explícitamente.

---

# 4. Sintaxis canónica de pseudocódigo

El libro utilizará una sintaxis independiente de TypeScript, Python, Java u otro lenguaje.

## Palabras reservadas

```text
STRUCT
ENUM
INTERFACE
IMPLEMENTATION
FUNCTION
COMMAND
EVENT

IF
ELSE
END

FOR EACH
WHILE

RETURN
THROW

EMIT
AWAIT

OPTIONAL
LIST
MAP
```

## Ejemplo

```text
STRUCT AgentState
    runId: RunId
    sessionId: SessionId
    status: AgentRunStatus
    messages: List<AgentMessage>
    currentTurn: Integer
END
```

## Regla de tipado

Preferir:

```text
result: ToolResult =
    ToolRuntime.execute(call, execution)
```

sobre:

```text
result = doSomething()
```

---

# 5. Data structures before behavior

Antes de describir comportamiento, deben definirse las estructuras sobre las que ese comportamiento opera.

```text
STRUCT ToolCall
    id: ToolCallId
    capability: CapabilityId
    arguments: Map<String, Value>
END
```

```text
STRUCT ToolResult
    callId: ToolCallId
    status: ToolExecutionStatus
    output: Optional<Value>
    error: Optional<HarnessError>
    metadata: ToolExecutionMetadata
END
```

```text
ENUM ToolExecutionStatus
    SUCCESS
    FAILED
    DENIED
    CANCELLED
    TIMED_OUT
END
```

Solo después se introduce `ToolRuntime.execute(...)`.

---

# 6. Ficha arquitectónica obligatoria por componente

Antes de mostrar cualquier pseudocódigo de un componente, debe existir una ficha arquitectónica.

## Plantilla

```text
COMPONENT: <ComponentName>

Responsibility:
    <Primary responsibility>

Consumes:
    <Inputs>

Depends on:
    <Dependencies>

Produces:
    <Outputs>

Owns:
    <Decisions exclusively owned>

Does NOT own:
    <Explicitly excluded responsibilities>
```

## Ejemplo

```text
COMPONENT: AgentLoop

Responsibility:
    Coordinates the cognitive execution cycle.

Consumes:
    AgentConfig
    AgentState
    ExecutionContext

Depends on:
    ContextEngine
    ModelGateway
    ToolRuntime
    EventBus
    ExecutionController

Produces:
    AgentRunResult
    AgentEvent

Owns:
    Turn continuation
    Cognitive cycle orchestration

Does NOT own:
    Persistence
    Authorization
    UI rendering
    Provider-specific APIs
```

---

# 7. Interfaces before implementations

Las dependencias deben expresarse mediante contratos.

```text
INTERFACE ModelGateway
    generate(
        request: ModelRequest,
        execution: ExecutionContext
    ) -> ModelResponse
END
```

Después pueden aparecer implementaciones:

```text
IMPLEMENTATION OpenAIModelGateway IMPLEMENTS ModelGateway
```

```text
IMPLEMENTATION AnthropicModelGateway IMPLEMENTS ModelGateway
```

El `AgentLoop` depende de `ModelGateway`, no de un provider específico.

---

# 8. Dependency direction

Las dependencias deben apuntar hacia contratos estables.

Correcto:

```text
AgentLoop
    ↓
ModelGateway
```

Incorrecto:

```text
AgentLoop
    ↓
OpenAIProvider
```

> Depend on abstractions, not implementations.

---

# 9. Architecture Dependency Map

El libro mantendrá un mapa explícito de dependencias que evolucionará por capítulos.

## Versión temprana

```text
AgentLoop
 ├── ModelGateway
 ├── ContextEngine
 └── ToolRuntime
```

## Versión posterior

```text
AgentLoop
 ├── ModelGateway
 ├── ContextEngine
 ├── ToolRuntime
 │    ├── PolicyEngine
 │    ├── CapabilityRegistry
 │    └── Sandbox
 ├── ExecutionController
 ├── SessionManager
 └── EventBus
```

Toda nueva dependencia debe justificarse arquitectónicamente.

---

# 10. Contract Registry

El libro mantendrá un registro canónico de contratos.

```text
C-001 AgentMessage
C-002 AgentConfig
C-003 AgentState
C-004 ExecutionContext
C-005 ContextSnapshot
C-006 ModelRequest
C-007 ModelResponse
C-008 ToolCall
C-009 ToolResult
C-010 AgentEvent
C-011 HarnessError
C-012 ExecutionBudget
```

Cada contrato tendrá:

```text
ID
Name
Version
Introduced In
Current Definition
Used By
Modified By
Constitutional Impact
```

Si cambia `C-008 ToolCall v1` a `C-008 ToolCall v2`, el capítulo debe explicar qué cambió, por qué, qué componentes afecta, compatibilidad y migration impact.

---

# 11. Component Registry

El libro mantendrá igualmente un registro de componentes.

```text
CMP-001 AgentLoop
CMP-002 ModelGateway
CMP-003 ContextEngine
CMP-004 ToolRuntime
CMP-005 PolicyEngine
CMP-006 SessionManager
CMP-007 EventBus
CMP-008 ExecutionController
CMP-009 HumanInteractionService
CMP-010 CapabilityRegistry
```

Cada componente tendrá:

```text
ID
Name
Responsibility
Owns
Does Not Own
Dependencies
Consumes
Produces
Introduced In
Constitutional Articles
```

---

# 12. Declaración de cambios por capítulo

Cada capítulo debe declarar explícitamente qué introduce y qué modifica.

```text
Introduces Components
    CMP-xxx

Introduces Contracts
    C-xxx

Modifies Contracts
    C-xxx

Introduces Events
    ...

Introduces Commands
    ...

Introduces Errors
    ...

Introduces States
    ...
```

Ejemplo:

```text
Chapter 5 — Tool Runtime

Introduces Components
    CMP-004 ToolRuntime
    CMP-011 ToolRegistry

Introduces Contracts
    C-008 ToolCall
    C-009 ToolResult
    C-013 ToolDefinition

Modifies Contracts
    C-007 ModelResponse
        + toolCalls

Introduces Events
    ToolExecutionStarted
    ToolExecutionCompleted

Introduces Errors
    ToolValidationError
    ToolExecutionError
```

---

# 13. Declaración de lo que NO cambia

Cada capítulo también debe declarar qué contratos o fronteras permanecen intactos.

```text
Unchanged

AgentMessage remains provider-independent.
AgentLoop still owns continuation.
ToolRuntime does not own authorization yet.
Session persistence remains out of scope.
```

Esto evita que cada incremento parezca una reescritura completa.

---

# 14. Tres vistas obligatorias por interacción

Toda interacción relevante debe mostrarse mediante tres representaciones.

## Vista 1 — Componentes

```text
AgentLoop → ToolRuntime → PolicyEngine
```

## Vista 2 — Sequence

```text
AgentLoop
   │ execute
   ▼
ToolRuntime
   │ authorize
   ▼
PolicyEngine
   │ decision
   ▼
ToolRuntime
```

## Vista 3 — Pseudocódigo

```text
result: ToolResult =
    ToolRuntime.execute(
        toolCall,
        executionContext
    )
```

Las tres vistas deben ser coherentes entre sí.

---

# 15. Sequence diagrams obligatorios

Toda colaboración importante entre componentes tendrá sequence diagram.

## Tool Execution

```text
AgentLoop
   │
   │ execute(calls)
   ▼
ToolRuntime
   │
   │ evaluate(call)
   ▼
PolicyEngine
   │
   │ ALLOW
   ▼
ToolRuntime
   │
   │ execute
   ▼
Tool
   │
   │ result
   ▼
ToolRuntime
   │
   │ ToolResult
   ▼
AgentLoop
```

## Human-in-the-loop

```text
AgentLoop
   │
   ▼
ToolRuntime
   │
   ▼
PolicyEngine
   │
   └── REQUIRE_APPROVAL
             │
             ▼
HumanInteractionService
             │
             ▼
SessionManager.persist()
             │
             ▼
WAITING_FOR_HUMAN
```

---

# 16. State machines explícitas

Los estados importantes no deben quedar implícitos.

```text
ENUM AgentRunStatus
    CREATED
    INITIALIZING
    RUNNING
    WAITING_FOR_MODEL
    WAITING_FOR_TOOL
    WAITING_FOR_HUMAN
    PAUSED
    COMPLETED
    FAILED
    CANCELLED
END
```

Transiciones:

```text
CREATED
  → INITIALIZING

INITIALIZING
  → RUNNING
  → FAILED

RUNNING
  → WAITING_FOR_MODEL
  → WAITING_FOR_TOOL
  → WAITING_FOR_HUMAN
  → COMPLETED
  → FAILED
  → CANCELLED
```

Cuando una feature agregue un nuevo estado o transición, debe declararlo explícitamente.

---

# 17. Commands, Events and State

```text
Command
    asks something to happen

Event
    reports that something happened

State
    represents current reality
```

Ejemplo:

```text
COMMAND ExecuteTool
        ↓
EVENT ToolExecuted
        ↓
AgentState updated
```

Nunca usar un evento como command ni un command como estado persistente.

---

# 18. Event envelope común

```text
STRUCT AgentEvent
    eventId: EventId
    eventType: AgentEventType
    timestamp: Timestamp
    runId: RunId
    sessionId: SessionId
    agentId: AgentId
    traceId: TraceId
    payload: Value
END
```

Eventos especializados:

```text
ToolExecutionStarted
ToolExecutionCompleted
PolicyEvaluated
HumanInteractionRequested
HumanInteractionResolved
ModelRequested
ModelResponded
```

---

# 19. Error contracts

```text
STRUCT HarnessError
    category: ErrorCategory
    code: String
    message: Text
    recoverable: Boolean
    retryable: Boolean
    metadata: Map<String, Value>
END
```

```text
ENUM ErrorCategory
    VALIDATION
    POLICY
    TOOL
    MODEL
    CONTEXT
    PERSISTENCE
    INFRASTRUCTURE
    BUDGET
    CANCELLATION
    FATAL
END
```

Evitar:

```text
THROW Error
```

Preferir:

```text
THROW HarnessError(
    category = TOOL,
    code = "TOOL_TIMEOUT",
    recoverable = TRUE,
    retryable = TRUE
)
```

---

# 20. Canonical Example

El libro debe utilizar un caso transversal que evolucione con la arquitectura.

> **Fix the failing authentication test.**

Evolución:

```text
Chapter 3
Model analyzes request.

Chapter 5
Agent reads source files.

Chapter 7
Agent receives repository context.

Chapter 13
UI observes tool execution events.

Chapter 15
Policy restricts a tool action.

Chapter 18
Human approval is requested.

Chapter 25
Execution budget limits the run.

Chapter 29
GitHub integration is installed as an extension.
```

El objetivo es mostrar la evolución del mismo sistema, no ejemplos inconexos.

---

# 21. Arquitectura antes y después de cada capítulo

Cada capítulo comienza con `Current Architecture` y termina con `Architecture After This Chapter`.

Before:

```text
AgentLoop
 ├── ModelGateway
 └── ContextEngine
```

After:

```text
AgentLoop
 ├── ModelGateway
 ├── ContextEngine
 └── ToolRuntime
```

El lector siempre debe saber qué cambió arquitectónicamente.

---

# 22. Constitutional Impact

Cada capítulo debe declarar su relación con `ARCHITECTURE_CONSTITUTION.md`.

```text
Constitutional Impact

Principles affected
    P-xx

Invariants introduced
    INV-xx

Invariants preserved
    INV-xx

Component ownership changes
    ...

Lifecycle changes
    ...

Security implications
    ...

Observability implications
    ...

Deterministic vs agentic boundary
    ...
```

---

# 23. Pseudocode consistency rules

## PC-01
Toda función declara parámetros y retorno.

## PC-02
Toda entidad relevante tiene tipo.

## PC-03
Todo componente utilizado existe en el Component Registry.

## PC-04
Todo contrato utilizado existe en el Contract Registry.

## PC-05
Los nombres son canónicos y no cambian informalmente.

## PC-06
Las llamadas entre componentes respetan el Dependency Map.

## PC-07
Resultados y errores están modelados.

## PC-08
No esconder side effects dentro de helpers genéricos.

## PC-09
Todo branching importante muestra condición o estado explícito.

## PC-10
El pseudocódigo debe ser suficientemente abstracto para no depender de lenguaje, pero suficientemente preciso para implementarse.

---

# 24. Separation of contracts and implementations

Patrón obligatorio:

```text
INTERFACE SessionStore

IMPLEMENTATION InMemorySessionStore
IMPLEMENTATION FileSessionStore
IMPLEMENTATION DatabaseSessionStore
```

Igualmente:

```text
INTERFACE HumanInteractionChannel

IMPLEMENTATION TUIChannel
IMPLEMENTATION WebChannel
IMPLEMENTATION SlackChannel
```

La lógica de dominio depende de interfaces, no de implementaciones.

---

# 25. Architecture Constitution hierarchy

```text
ARCHITECTURE_CONSTITUTION.md
        │
        ▼
COMPONENT_REGISTRY.md
        │
        ▼
CONTRACT_REGISTRY.md
        │
        ▼
ADR/
        │
        ▼
Chapter Architecture
        │
        ▼
Pseudocode
        │
        ▼
Implementation
```

La Constitution define reglas fundamentales; el Component Registry define responsabilidades y ownership; el Contract Registry define interfaces y estructuras canónicas; los ADRs documentan decisiones; los capítulos enseñan evolución; el pseudocódigo formaliza comportamiento; la implementación materializa la arquitectura.

---

# 26. Estructura obligatoria de cada capítulo

Cada capítulo seguirá exactamente esta secuencia:

1. **Current Architecture** — Mostrar el estado actual del sistema.
2. **Problem** — Explicar el problema nuevo.
3. **Why the Current Architecture Is Insufficient** — Explicar la limitación actual.
4. **Constitutional Impact** — Identificar principles, invariants y boundaries afectados.
5. **New Concepts** — Introducir exclusivamente conceptos nuevos.
6. **New Data Structures** — Definir `STRUCT`, `ENUM` y tipos nuevos.
7. **New Contracts / Interfaces** — Definir interfaces antes de implementación.
8. **Component Responsibilities** — Mostrar ficha arquitectónica de cada componente nuevo.
9. **Dependency Relationships** — Actualizar Dependency Map.
10. **Sequence Diagram** — Mostrar colaboración temporal.
11. **Pseudocode** — Implementar usando únicamente entidades ya definidas.
12. **State Transitions** — Mostrar cambios de estado.
13. **Failure Semantics** — Definir fallos, recoverability y retries.
14. **Events Produced** — Definir eventos generados.
15. **Security / Policy Implications** — Explicar impacto de seguridad.
16. **Tests** — Definir tests arquitectónicos, unitarios o de comportamiento.
17. **Architecture After This Chapter** — Mostrar arquitectura resultante.
18. **What We Deliberately Do Not Solve Yet** — Declarar limitaciones restantes.
19. **Next Increment** — Explicar el problema natural del siguiente capítulo.

---

# 27. Secuencia de construcción dentro de un capítulo

```text
Problem
   ↓
Architecture limitation
   ↓
Concept
   ↓
Data structure
   ↓
Contract
   ↓
Component
   ↓
Dependency
   ↓
Interaction
   ↓
Pseudocode
   ↓
State
   ↓
Failures
   ↓
Events
   ↓
Tests
   ↓
New Architecture
```

No invertir esta secuencia salvo una razón pedagógica explícita.

---

# 28. Tests arquitectónicos

Además de tests funcionales, el libro debe introducir tests que validen invariants.

```text
TEST AgentLoopDoesNotDependOnOpenAIProvider
TEST CriticalToolCannotBypassPolicyEngine
TEST AgentCoreCanRunWithoutTUI
TEST HumanInteractionCanResumeFromPersistedState
```

El objetivo es convertir parte de la Architecture Constitution en reglas verificables.

---

# 29. Naming consistency

Nombres canónicos:

```text
AgentLoop
AgentState
SessionState
ExecutionContext
ModelGateway
ContextEngine
ToolRuntime
PolicyEngine
SessionManager
EventBus
ExecutionController
HumanInteractionService
CapabilityRegistry
```

Cambiar un nombre canónico requiere actualización del Contract Registry, Component Registry, ADR si es arquitectónico y migration note.

---

# 30. Versioning

Cada milestone tendrá una versión ejecutable.

```text
v0.1 Agent Loop
v0.2 Tool Runtime
v0.3 Context-Aware Agent
v0.4 Event-Driven Harness
v0.5 Durable HITL
v0.6 Reliable Harness
v0.7 Extensible Platform
v0.8 Automation Runtime
v0.9 Multi-Agent Runtime
v1.0 Enterprise Agent Operating Layer
```

Cada capítulo debe declarar:

```text
Starting Version
Ending Version
Contracts Changed
Components Changed
```

---

# 31. Regla final de consistencia

Antes de cerrar un capítulo, verificar:

```text
[ ] Every pseudocode entity is defined.
[ ] Every component has a responsibility boundary.
[ ] Every dependency points to a known contract.
[ ] Every new contract exists in the Contract Registry.
[ ] Every new component exists in the Component Registry.
[ ] Sequence diagrams match pseudocode.
[ ] State transitions are explicit.
[ ] Errors are classified.
[ ] Events are defined.
[ ] Constitutional impact is documented.
[ ] Current and resulting architecture are shown.
[ ] Remaining limitations are explicit.
[ ] The next chapter follows naturally.
```

---

# 32. Doctrina editorial

El libro no debe enseñar únicamente cómo escribir código de agentes.

Debe enseñar a pensar:

```text
Concept
    ↓
Responsibility
    ↓
Contract
    ↓
Boundary
    ↓
Interaction
    ↓
State
    ↓
Behavior
    ↓
Failure
    ↓
Observation
    ↓
Evolution
```

La arquitectura debe poder entenderse incluso antes de ver una implementación real.

El pseudocódigo debe ser la representación ejecutable de esa arquitectura conceptual.

> **The book does not derive architecture from code. The code and pseudocode are derived from architecture.**
