# Software Development — Reference Harness Specification
**Canonical ID:** RH-02  
**Version:** 0.1  
**Category:** Engineering / Tool-Intensive Execution  
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

# RH-02 — Software Development Harness

## Propósito
Construir, modificar, depurar, probar y revisar software.

## Activaciones
Usuario, issue, pull request, CI failure, webhook, queue, scheduled maintenance.

## Agentes
Developer, Planner, Test Reviewer, Code Reviewer, opcional Architecture Agent.

## State
`ProjectState`, `RepositoryState`, `DevelopmentTask`, `DevelopmentPlan`, `BuildState`, `TestState`.

## Context
Repository map, código relevante, símbolos, dependencias, tests, conventions, architecture rules.

## Skills
`inspect-repository`, `implement-feature`, `debug`, `write-tests`, `refactor`, `review-code`.

## Capabilities
Filesystem, shell, Git, GitHub, build tools, test runners, package managers.

## Workflows
Inspect → Plan → Edit → Test → Fix → Build → Diff → Review.

## HITL
Push, major refactor, schema migration, deployment, breaking change.

## Evals
Task completion, test success, regression rate, diff quality, security, cost.

## Ejercicios
1. Feature implementation.
2. Bug fixing.
3. CI failure diagnosis.
4. Pull-request review.
5. Dependency upgrade.
6. API migration.
7. Frontend refactor.
8. Database migration.

---

---

# Detailed implementation specification


## Reference state contracts

```text
ProjectState {
  repository
  branch
  baseRevision
  architectureRules[]
  task
  plan
  changedFiles[]
  buildState
  testState
  reviewState
  artifacts[]
  budget
}

DevelopmentTask {
  taskId
  objective
  acceptanceCriteria[]
  constraints[]
  riskClass
  sourceRef
}

DevelopmentPlan {
  steps[]
  dependencies[]
  expectedFiles[]
  validationPlan
  rollbackPlan
}
```

## Repository Context Engine

```text
Task
 ↓
Repository Map
 ↓
Symbol / Dependency Search
 ↓
Relevant Files
 ↓
Nearby Tests + Interfaces
 ↓
Architecture Rules / ADRs
 ↓
ContextBundle
```

The full repository is not the prompt.

## Tool contracts

```text
ToolCall {
  callId
  capabilityId
  arguments
  requestedBy
  runId
  idempotencyKey?
}

ToolResult {
  callId
  status
  output
  artifacts[]
  sideEffects[]
  error?
}
```

The runtime evaluates tool results. A command that ran successfully but returned failing tests is a valid observation, not automatically a runtime failure.

## Agent loop pseudocode

```text
WHILE NOT terminal(state):

  validateState(state)
  context = contextEngine.build(state)

  response = modelGateway.complete(
      promptBuilder.build(state, context)
  )

  decision = decisionEngine.evaluate(response, state)

  IF decision == TOOL_CALLS:
      graph = dependencyAnalyzer.build(decision.calls)
      results = toolRuntime.executeGraph(graph)
      state = reducer.applyToolResults(state, results)

  ELSE IF decision == HUMAN_REQUEST:
      state = humanInteraction.wait(decision)

  ELSE IF decision == REPLAN:
      state = planner.replan(state)

  ELSE IF decision == COMPLETE:
      state = validator.finalize(state)

  checkpoint.save(state)
```

## Parallelism

```text
read package.json ─────┐
read target source ────┼── parallel
read relevant tests ───┘

edit source → compile → test → inspect failure
```

Scheduling belongs to the runtime, not to model-provider-specific code.

## Agent communication

```text
Developer Agent
  ├── TASK_REQUEST → Test Agent
  └── TASK_REQUEST → Reviewer Agent
```

Delegation passes only selected context, capabilities, budget, deadline, and authority. External independent agents may be reached through an A2A adapter; internal sub-agents need not use A2A.

## Example policies

```text
POL-DEV-001: The model cannot execute shell directly.
POL-DEV-002: Writes outside the workspace are denied.
POL-DEV-003: Force push requires approval.
POL-DEV-004: Production deployment requires approval.
POL-DEV-005: Raw secrets never enter model context.
POL-DEV-006: Destructive schema migration requires architecture/human review.
```

## Failure and recovery exercise

Scenario: source edits succeed, then tests fail.

Expected behavior:
1. Store the edit ToolResults and test output.
2. Treat the failed tests as an observation.
3. Permit diagnosis and additional bounded iterations.
4. Enforce `maxTurns`, `maxToolCalls`, `maxCost`, and `maxWallClock`.
5. Preserve the final diff and all tool actions in audit.
6. Escalate or terminate if recovery limits are exceeded.

## Minimum eval suite

`repository_understanding`, `tool_selection`, `change_correctness`, `test_generation`, `bug_fix`, `regression`, `security`, `policy_compliance`, `diff_minimality`, `architecture_compliance`.

## Required student deliverables

- provider-independent `ModelGateway`;
- repository context design;
- tool/capability contracts;
- execution DAG;
- state reducer;
- policy examples;
- HITL sequence;
- checkpoint/recovery design;
- execution graph for delegated agents;
- eval plan;
- architecture ADRs;
- one working feature/bug/CI scenario.

## Definition of Done

The implementation proves model independence, controlled tool use, independent/dependent execution, result evaluation, bounded loops, HITL, durable checkpoints, audit, tests/build validation, failure recovery, and explicit control of side effects.


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
