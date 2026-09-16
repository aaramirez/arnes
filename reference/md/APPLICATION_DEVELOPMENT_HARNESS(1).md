# Application Development Harness
## Instrucciones para construir un Harness especializado en programación de aplicaciones

## 1. Propósito

Este documento define cómo construir un **Application Development Harness** especializado en desarrollar, modificar, depurar, probar, revisar y evolucionar aplicaciones de software.

No debe diseñarse como un sistema completamente independiente.

Debe construirse como:

```text
Generic Agent Harness
        +
Coding Domain Pack
        =
Application Development Harness
```

El Generic Agent Harness aporta primitives comunes:

```text
AgentLoop
ModelGateway
ContextEngine
ToolRuntime
PolicyEngine
SessionManager
EventBus
ExecutionController
HumanInteractionService
Skills
Evals
```

El Coding Domain Pack aporta:

```text
Repository State
Coding Skills
Development Tools
Engineering Policies
Technical Validators
Repository Context
Development Evals
```

Regla fundamental:

> **Probabilistic systems may propose decisions. Deterministic systems must govern consequences.**

---

# 2. Objetivos del Harness

El sistema debe poder ejecutar tareas como:

```text
build feature
fix bug
debug failure
refactor code
write tests
review code
modify API
modify frontend
perform database migration
upgrade dependency
analyze repository
document architecture
prepare pull request
```

El objetivo no es únicamente generar código.

El harness debe gestionar el ciclo completo:

```text
Requirement
   ↓
Understand
   ↓
Inspect
   ↓
Plan
   ↓
Modify
   ↓
Validate
   ↓
Observe
   ↓
Correct
   ↓
Review
   ↓
Deliver
```

---

# 3. Arquitectura general

```text
                  APPLICATION DEV HARNESS
                            │
                  ┌─────────┴─────────┐
                  │                   │
          Development Controller   Project State
                  │
                  ▼
            Development Pipeline
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
   Planning     Coding     Validation
       │          │           │
       └──────────┼───────────┘
                  ▼
              Quality Gates
                  │
                  ▼
           Repository Changes
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Tests      Build      Diff
                              │
                              ▼
                         Delivery / PR
```

---

# 4. Source of Truth

El source of truth principal es el repositorio.

```text
repository/
├── src/
├── tests/
├── docs/
├── scripts/
├── architecture/
├── package manifests
├── configuration
└── CI/CD definitions
```

El harness no debe asumir una estructura específica.

Debe descubrir:

```text
languages
frameworks
package managers
build system
test framework
repository conventions
architecture
dependencies
CI/CD
documentation
```

---

# 5. Project State

Crear estado explícito para cada tarea de desarrollo.

```text
STRUCT ProjectState
    repository: RepositoryState
    branch: BranchRef
    task: DevelopmentTask

    architecture: ArchitectureState
    dependencies: DependencyGraph

    tests: TestState
    build: BuildState

    changes: List<FileChange>
    decisions: List<ADR>

    status: DevelopmentStatus
END
```

## RepositoryState

```text
STRUCT RepositoryState
    root: Path
    branch: BranchRef

    languages: List<Language>
    frameworks: List<Framework>
    packageManagers: List<PackageManager>

    buildCommands: List<CommandDefinition>
    testCommands: List<CommandDefinition>

    conventions: RepositoryConventions
END
```

## DevelopmentTask

```text
STRUCT DevelopmentTask
    id: TaskId
    description: Text
    acceptanceCriteria: List<Criterion>
    constraints: List<Constraint>
    riskLevel: RiskLevel
END
```

## DevelopmentStatus

```text
ENUM DevelopmentStatus
    CREATED
    ANALYZING
    PLANNING
    IMPLEMENTING
    VALIDATING
    REVIEWING
    WAITING_FOR_HUMAN
    COMPLETED
    FAILED
    CANCELLED
END
```

---

# 6. Repository Context

No enviar todo el repositorio al modelo.

Crear un `RepositoryContextEngine`.

```text
INTERFACE RepositoryContextEngine

    build(
        task: DevelopmentTask,
        state: ProjectState,
        execution: ExecutionContext
    ) -> RepositoryContext

END
```

Debe seleccionar contexto relevante:

```text
task
repository map
relevant files
symbols
dependencies
tests
architecture rules
coding conventions
recent changes
related documentation
```

Pipeline:

```text
Development Task
       ↓
Repository Search
       ↓
Candidate Files
       ↓
Dependency Analysis
       ↓
Relevance Ranking
       ↓
Context Budget
       ↓
RepositoryContext
```

---

# 7. Agentes

Comenzar con pocos agentes.

## 7.1 Developer Agent

Primera versión recomendada.

Responsabilidades:

```text
understand task
inspect repository
plan implementation
edit files
run tests
analyze failures
fix failures
review diff
```

En las primeras versiones este agente puede manejar el ciclo completo.

No introducir múltiples agentes hasta demostrar una necesidad clara.

---

# 8. Evolución de agentes

Cuando el sistema madure, separar responsabilidades.

## Planner Agent

```text
understand requirement
inspect architecture
identify affected components
identify files
identify risks
create implementation plan
```

## Developer Agent

```text
implement approved plan
modify files
add tests
refactor
```

## Test Agent

```text
run tests
analyze failures
identify regressions
evaluate coverage
```

## Reviewer Agent

```text
review diff
check architecture
check maintainability
check security
check task compliance
```

Arquitectura futura:

```text
Development Controller
        │
        ├── Planner
        │
        ├── Developer
        │
        ├── Test Reviewer
        │
        └── Code Reviewer
```

Pero mantener esta separación solo cuando produzca valor real.

---

# 9. Skills

Crear skills especializados.

```text
skills/
├── inspect-repository/
│   └── SKILL.md
├── plan-code-change/
│   └── SKILL.md
├── implement-feature/
│   └── SKILL.md
├── debug-failure/
│   └── SKILL.md
├── write-unit-tests/
│   └── SKILL.md
├── write-integration-tests/
│   └── SKILL.md
├── review-code/
│   └── SKILL.md
├── refactor-code/
│   └── SKILL.md
├── database-migration/
│   └── SKILL.md
├── frontend-component/
│   └── SKILL.md
├── api-endpoint/
│   └── SKILL.md
├── architecture-review/
│   └── SKILL.md
└── prepare-pull-request/
    └── SKILL.md
```

Skills representan procedimiento.

Ejemplo:

```text
SKILL: debug-failure

1. Reproduce the failure.
2. Capture exact error.
3. Identify failing boundary.
4. Inspect relevant code.
5. Form hypotheses.
6. Test the smallest hypothesis first.
7. Modify only relevant code.
8. Re-run failing test.
9. Run related tests.
10. Run broader validation.
11. Review diff.
```

---

# 10. Tools

## Tools iniciales

```text
read_file
write_file
edit_file
list_files
search_code
shell
git_diff
git_status
```

Contratos conceptuales:

```text
INTERFACE FileSystemTool
    read(path: Path) -> FileContent
    write(path: Path, content: Text) -> WriteResult
    edit(path: Path, patch: Patch) -> EditResult
END
```

```text
INTERFACE ShellTool
    execute(
        command: Command,
        execution: ExecutionContext
    ) -> CommandResult
END
```

```text
INTERFACE GitTool
    status() -> GitStatus
    diff() -> GitDiff
END
```

---

# 11. Tools avanzadas

Agregar solamente cuando exista necesidad.

```text
GitHub
Database
Browser
Docker
Kubernetes
Cloud
Issue Tracker
CI/CD
Package Registry
Documentation Search
```

Estas deben exponerse como capabilities.

Ejemplo:

```text
repository.read
repository.edit
tests.run
build.run
git.diff
github.create_pr
database.migrate
deployment.staging
```

El agente debe depender de la capability, no necesariamente de la implementación concreta.

---

# 12. Development Loop

Flujo principal:

```text
User Task
   ↓
Understand Requirement
   ↓
Inspect Repository
   ↓
Build Relevant Context
   ↓
Plan Change
   ↓
Edit Files
   ↓
Run Tests
   ↓
Observe Failures
   ↓
Fix
   ↓
Run Build
   ↓
Review Diff
   ↓
Validate Acceptance Criteria
   ↓
Return Result
```

Pseudocódigo:

```text
FUNCTION executeDevelopmentTask(
    task: DevelopmentTask,
    state: ProjectState,
    execution: ExecutionContext
) -> DevelopmentResult

    state.status = ANALYZING

    context: RepositoryContext =
        RepositoryContextEngine.build(
            task,
            state,
            execution
        )

    WHILE ExecutionController.canContinue(execution)

        response: ModelResponse =
            ModelGateway.generate(
                context.toModelRequest(),
                execution
            )

        IF response.hasToolCalls()

            results: List<ToolResult> =
                ToolRuntime.execute(
                    response.toolCalls,
                    execution
                )

            context.addObservations(results)

            CONTINUE
        END

        state.status = VALIDATING

        validation: ValidationResult =
            ValidationPipeline.evaluate(
                task,
                state,
                execution
            )

        IF validation.satisfied

            state.status = COMPLETED

            RETURN DevelopmentResult.success(
                response.output,
                state.changes,
                validation
            )

        END

        context.add(
            validation.feedback
        )

        state.status = IMPLEMENTING

    END

    RETURN DevelopmentResult.incomplete()
END
```

---

# 13. Planning

Para cambios no triviales crear un `DevelopmentPlan`.

```text
STRUCT DevelopmentPlan
    objective: Text

    affectedComponents: List<ComponentRef>
    affectedFiles: List<Path>

    steps: List<DevelopmentStep>

    risks: List<Risk>

    validations: List<ValidationRequirement>
END
```

Ejemplo:

```text
DevelopmentPlan

Objective:
    Add password reset endpoint.

Affected Components:
    AuthController
    AuthService
    EmailService

Files:
    auth.controller
    auth.service
    auth.spec

Steps:
    1. Add request contract.
    2. Add service method.
    3. Add controller endpoint.
    4. Add tests.
    5. Run auth tests.
    6. Run build.

Risks:
    email side effect
    token security
```

---

# 14. Parallelism and Dependencies

Permitir ejecución paralela cuando las operaciones sean independientes.

Ejemplo:

```text
read(auth.service) ─────┐
read(auth.controller) ──┼──► context
read(auth.spec) ────────┘
```

No paralelizar cuando exista dependencia.

```text
edit code
   ↓
compile
   ↓
run tests
```

Tampoco ejecutar concurrentemente operaciones con side effects incompatibles.

El `ToolRuntime` debe evaluar:

```text
dependency
sideEffect
sharedState
risk
```

antes de paralelizar.

---

# 15. Quality Gates

Las validaciones técnicas deben ser principalmente determinísticas.

```text
Formatting
Lint
Type Check
Unit Tests
Integration Tests
Build
Security Scan
Architecture Rules
Git Diff Review
Acceptance Criteria
```

Pipeline:

```text
Code Changes
     ↓
Format
     ↓
Lint
     ↓
Type Check
     ↓
Unit Tests
     ↓
Integration Tests
     ↓
Build
     ↓
Architecture Validation
     ↓
Diff Review
```

No todos los proyectos tendrán todos los gates.

Descubrirlos desde el repositorio.

---

# 16. Scripts determinísticos

Ejemplo:

```text
scripts/
├── format
├── lint
├── typecheck
├── test
├── integration-test
├── build
├── validate-architecture
├── validate-dependencies
├── detect-secrets
└── validate-diff
```

El modelo puede decidir cuándo una validación es relevante.

El harness decide cómo se ejecuta y si puede omitirse.

---

# 17. Validation Pipeline

```text
INTERFACE ValidationPipeline

    evaluate(
        task: DevelopmentTask,
        state: ProjectState,
        execution: ExecutionContext
    ) -> ValidationResult

END
```

```text
STRUCT ValidationResult
    satisfied: Boolean
    checks: List<ValidationCheck>
    feedback: List<ValidationFeedback>
END
```

```text
STRUCT ValidationCheck
    name: Text
    status: ValidationStatus
    evidence: Optional<Value>
END
```

---

# 18. Policies

Crear políticas determinísticas.

```text
policies/
├── filesystem.yaml
├── git.yaml
├── database.yaml
├── deployment.yaml
├── secrets.yaml
└── architecture.yaml
```

Ejemplo:

```yaml
filesystem:
  repository_write: allow
  outside_repository: deny

git:
  status: allow
  diff: allow
  commit: allow
  push: require_approval

database:
  read: allow
  schema_change: require_approval
  destructive_change: deny

deployment:
  development: allow
  staging: require_approval
  production: deny
```

No implementar estas restricciones únicamente mediante instrucciones al modelo.

---

# 19. Human-in-the-Loop

Ejemplos de acciones que pueden requerir aprobación:

```text
git push
database migration
dependency major upgrade
deployment
large refactor
breaking API change
security-sensitive modification
architecture change
```

Flujo:

```text
Developer Agent
      ↓
proposes action
      ↓
ToolRuntime
      ↓
PolicyEngine
      ↓
REQUIRE_APPROVAL
      ↓
HumanInteractionService
      ↓
WAITING_FOR_HUMAN
      ↓
Approve / Reject
      ↓
Resume
```

La aprobación debe ser independiente de la interfaz.

---

# 20. Failure Semantics

Clasificar errores.

```text
RepositoryError
BuildError
TestFailure
LintFailure
TypeCheckFailure
ToolError
PolicyError
ModelError
InfrastructureError
BudgetExceeded
Cancellation
```

Ejemplo:

```text
TestFailure
    recoverable = TRUE
    retryable = FALSE
    agentCanReason = TRUE
```

```text
TemporaryPackageRegistryFailure
    recoverable = TRUE
    retryable = TRUE
```

```text
PolicyDeniedProductionDeployment
    recoverable = FALSE
    retryable = FALSE
```

---

# 21. Execution Budget

Toda tarea debe tener límites.

```text
STRUCT DevelopmentBudget
    maxTurns: Integer
    maxToolCalls: Integer
    maxRuntime: Duration
    maxCost: Money
    maxFilesModified: Integer
    maxConcurrentTools: Integer
END
```

Para tareas sensibles agregar:

```text
maxLinesChanged
allowedPaths
deniedPaths
```

El modelo no controla estos límites.

---

# 22. Events

Emitir eventos observables.

```text
DevelopmentTaskStarted
RepositoryInspected
ContextBuilt
PlanCreated
FileRead
FileModified
CommandExecuted
TestStarted
TestCompleted
BuildStarted
BuildCompleted
PolicyEvaluated
HumanApprovalRequested
HumanApprovalResolved
DiffReviewed
DevelopmentTaskCompleted
DevelopmentTaskFailed
```

Envelope:

```text
STRUCT DevelopmentEvent
    eventId: EventId
    timestamp: Timestamp
    taskId: TaskId
    runId: RunId
    traceId: TraceId
    eventType: DevelopmentEventType
    payload: Value
END
```

---

# 23. Observability

Un development run debe poder reconstruirse.

```text
Development Run
├── task
├── repository snapshot
├── context selected
├── model calls
├── files read
├── files modified
├── commands
├── test results
├── build results
├── policy decisions
├── approvals
├── retries
├── failures
├── diff
├── cost
└── final result
```

---

# 24. Evals

Evaluar el harness en dimensiones concretas.

```text
Task Completion
Code Correctness
Test Success
Regression Rate
Architecture Compliance
Tool Selection Accuracy
Tool Argument Accuracy
Repository Context Relevance
Diff Quality
Security Compliance
Cost
Latency
Human Intervention Rate
Recovery Success
```

Crear datasets de tareas conocidas para regresión.

---

# 25. Repository Layout del Harness

```text
application-dev-harness/
│
├── agents/
│   ├── developer.md
│   ├── planner.md
│   ├── reviewer.md
│   └── test-reviewer.md
│
├── skills/
│   ├── inspect-repository/
│   ├── plan-code-change/
│   ├── implement-feature/
│   ├── debug-failure/
│   ├── write-unit-tests/
│   ├── review-code/
│   ├── refactor-code/
│   ├── database-migration/
│   ├── frontend-component/
│   ├── api-endpoint/
│   ├── architecture-review/
│   └── prepare-pull-request/
│
├── policies/
│   ├── filesystem.yaml
│   ├── git.yaml
│   ├── database.yaml
│   ├── deployment.yaml
│   ├── secrets.yaml
│   └── architecture.yaml
│
├── packages/
│   ├── development-core/
│   ├── repository-context/
│   ├── validation/
│   ├── coding-tools/
│   └── development-evals/
│
├── scripts/
│   ├── validate-project
│   ├── validate-diff
│   └── run-quality-gates
│
├── evals/
│
└── docs/
    └── adr/
```

Si existe un Generic Harness compartido, las primitives comunes deben importarse desde él y no duplicarse.

---

# 26. Primera versión

No construir toda la arquitectura inicialmente.

## DEV-HARNESS v0.1

Implementar:

```text
1 Developer Agent

Skills:
- inspect-repository
- implement-feature
- debug-failure
- write-unit-tests

Tools:
- read_file
- edit_file
- search_code
- shell
- git_diff
- git_status

Context:
- repository map
- relevant file selection

Validators:
- tests
- build
- diff

State:
- ProjectState
- DevelopmentTask

Execution:
- budgets
- cancellation
```

Flujo mínimo:

```text
Task
 ↓
Inspect
 ↓
Context
 ↓
Edit
 ↓
Test
 ↓
Fix
 ↓
Build
 ↓
Diff
 ↓
Result
```

---

# 27. Evolución recomendada

```text
v0.1
Single Developer Agent

v0.2
Repository Context Engine

v0.3
Development Plans

v0.4
Policy Engine integration

v0.5
Durable HITL

v0.6
Reliability + execution ledger

v0.7
Reviewer and testing roles

v0.8
GitHub / CI extensions

v0.9
Automation

v1.0
Production Application Development Harness
```

Separar nuevos agentes únicamente cuando exista evidencia de que una responsabilidad necesita contexto, lifecycle o capabilities independientes.

---

# 28. Ejemplo transversal

Usar un mismo caso para probar la evolución:

> **Fix the failing authentication test.**

## v0.1

```text
Agent reads test.
Agent reads authentication code.
Agent edits code.
Agent runs test.
```

## v0.2

```text
RepositoryContextEngine discovers:
auth.service
auth.controller
auth.spec
token utility
```

## v0.3

```text
Agent creates DevelopmentPlan.
```

## v0.4

```text
Policy prevents unauthorized filesystem access.
```

## v0.5

```text
A sensitive configuration change requires approval.
```

## v0.6

```text
A failed side effect is safely recovered.
```

## v0.7

```text
Reviewer evaluates final diff.
```

## v1.0

```text
Task
 ↓
Plan
 ↓
Implement
 ↓
Validate
 ↓
Review
 ↓
PR
```

---

# 29. Relationship with Other Specialized Harnesses

Mantener explícito que este es un Domain Pack.

```text
                     GENERIC HARNESS
                           │
          ┌────────────────┼─────────────────┐
          ▼                ▼                 ▼
      Coding Pack       Book Pack       Research Pack
          │                │                 │
          ▼                ▼                 ▼
    Coding Harness    Book Harness    Research Harness
```

Primitives compartidas:

```text
AgentLoop
ModelGateway
ContextEngine
ToolRuntime
PolicyEngine
SessionManager
EventBus
ExecutionController
HumanInteractionService
Skills
Evals
```

Primitives específicas de Coding:

```text
ProjectState
DevelopmentTask
RepositoryContext
DevelopmentPlan
ValidationPipeline
Git capabilities
Build capabilities
Test capabilities
Development policies
Development evals
```

---

# 30. Definition of Done

Un Application Development Harness funcional debe demostrar:

```text
[ ] Puede inspeccionar un repositorio desconocido.
[ ] Construye contexto relevante sin enviar todo el repositorio.
[ ] Puede modificar archivos mediante ToolRuntime.
[ ] Puede ejecutar tests.
[ ] Puede observar y razonar sobre fallos.
[ ] Puede corregir una implementación.
[ ] Puede ejecutar build y quality gates.
[ ] Puede revisar el diff final.
[ ] Respeta filesystem policies.
[ ] Respeta execution budgets.
[ ] Side effects sensibles requieren aprobación.
[ ] Las operaciones son observables.
[ ] Los errores tienen semántica explícita.
[ ] El modelo puede cambiar sin modificar Development Loop.
[ ] Las skills pueden evolucionar sin modificar el core.
[ ] El Coding Domain Pack permanece separado del Generic Harness.
```

---

# 31. Doctrina final

El Application Development Harness no debe convertirse en una colección de prompts para programar.

Debe ser un sistema operacional:

```text
Model
    reasons

Repository Context
    informs

Skills
    guide procedure

Tools
    modify and execute

Policies
    constrain

Validators
    verify

State
    tracks

Events
    expose

Humans
    approve consequential actions

Evals
    measure quality

Generic Harness
    governs execution
```

La meta final es que el mismo Generic Harness pueda ejecutar diferentes dominios mediante Domain Packs:

```text
Generic Harness
      +
Coding Domain Pack
      =
Application Development Harness
```

Esto permite reutilizar la infraestructura fundamental sin acoplar el runtime al dominio de programación.

# 32. Enterprise activation and agent communication amendment

The Coding Harness is activated through the Generic Activation Layer, not only by an interactive user. Supported sources may include issue creation, pull-request events, CI failures, repository webhooks, queues, schedules, APIs and other agents.

```text
GitHub/CI/API/Queue/User
        ↓
ActivationGateway
        ↓
AdmissionController
        ↓
ActivationRouter
        ↓
Coding Harness
```

For multi-agent development, use `AgentCommunicationGateway`. Internal Planner/Developer/Test/Reviewer delegation remains an internal orchestration concern; independently deployed security, architecture or vendor agents may interoperate through an A2A adapter. HTTP, webhooks, WebSockets, SSE, gRPC and queues are transport choices and MUST NOT leak into coding-agent semantics.

Add enterprise controls: delegated capability scopes, execution graph limits, repository/tenant isolation, credential broker, capability versioning, durable CI runs, idempotent external actions, immutable audit evidence, evaluation/certification before production rollout, and operational kill switches.

