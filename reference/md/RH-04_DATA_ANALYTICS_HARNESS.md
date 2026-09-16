# RH-04 — Data & Analytics Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness  
**Status:** Reference specification for student and enterprise implementations

---

## 1. Architectural position

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
Domain Pack
        ↓
RH-04 — Data & Analytics
        ↓
Specific Use Case
```

This specification must reuse Generic Harness primitives. Domain-specific behavior belongs in agents, skills, policies, workflows, capabilities, validators, and context providers—not in the core loop.

## 2. Purpose

Convert business questions into governed, reproducible, validated analysis and decision artifacts.

## 3. Actors

Business requester, Data Analyst, Data Engineer, Statistical Reviewer, Data Owner, Decision Maker.

## 4. Activation sources

User/API question, schedule, dataset arrival, KPI anomaly, event bus, another agent.

Every source is normalized:

```text
External Source
    ↓
TriggerAdapter
    ↓
ActivationRequest
    ↓
AdmissionController
    ↓
ActivationRouter
    ↓
Workflow / Agent Runtime
```

`ActivationRequest` is channel-independent.

## 5. Admission

Minimum checks:

```text
identity / principal
tenant / scope
authorization
risk classification
duplicate activation
budget
data classification
workflow availability
```

Possible outcomes: `ACCEPT`, `REJECT`, `DEFER`, `DEDUPLICATE`, `ROUTE`, `REQUIRE_APPROVAL`.

## 6. Agents and responsibilities

Data Discovery Agent; Analysis Planner; Query/Transformation Agent; Statistical Reviewer; Business Interpretation Agent.

Agents communicate through explicit task/result contracts. They do not depend on shared implicit memory.

## 7. State

AnalysisState, DatasetState, AnalysisPlan, QueryExecution, MetricDefinition, ModelResult, ValidationState.

Reference envelope:

```text
DomainRunState {
  runId
  activationId
  principal
  workflowPosition
  domainState
  childRuns[]
  pendingHumanTasks[]
  artifacts[]
  budget
  status
  version
}
```

State is durable where the workflow can wait, resume, retry, or cross process boundaries.

## 8. Context engineering

Data catalog, schemas, lineage, metric definitions, semantic layer, approved datasets, prior analysis artifacts.

```text
ContextBundle
├── task
├── selected domain state
├── retrieved evidence/data
├── applicable policies
├── relevant history
├── capability descriptions
└── provenance metadata
```

Context is intentionally selected. It is not synonymous with memory or complete conversation history.

## 9. Memory boundaries

Separate:
- execution state;
- durable domain state;
- retrievable enterprise knowledge;
- user/customer/account memory where permitted;
- artifacts.

The model context is never the canonical database.

## 10. Skills

discover-data, formulate-metric, plan-analysis, write-query, transform-data, analyze, validate-statistics, visualize, interpret.

Skills are reusable domain instruction packages and may be versioned independently.

## 11. Capabilities

catalog.search, warehouse.query, sandbox.python, files.read/write, visualization.render, lineage.lookup.

Minimum capability contract:

```text
CapabilityDefinition {
  capabilityId
  version
  inputSchema
  outputSchema
  sideEffect
  riskLevel
  idempotencySemantics
  timeout
  requiredPermissions[]
}
```

The model requests capabilities; the Tool Runtime authorizes and executes them.

## 12. Canonical workflow

```text
Question → Data Discovery → Analysis Plan → Query/Transform → Analyze → Validate → Visualize → Interpret → Human/Decision Artifact.
```

## 13. Parallelism and dependencies

Independent dataset profiling and metric checks may run in parallel; interpretation depends on validated analytical results.

The runtime builds an execution DAG:

```text
ExecutionNode {
  nodeId
  operation
  dependencies[]
  status
  retryPolicy
}
```

The model may propose tasks, but scheduling and dependency enforcement belong to the runtime.

## 14. Agent communication

```text
Agent A
  ↓ AgentTaskRequest
AgentCommunicationGateway
  ↓
Agent B
  ↓ AgentTaskResult
Agent A resumes
```

Internal sub-agents may use in-process messaging, queues, or other internal transports. Independent external agents may use an A2A adapter. Semantics remain independent from protocol and transport.

## 15. Delegated authority

A child agent receives only the authority needed for the delegated task.

```text
DelegationToken {
  issuer
  subject
  capabilities[]
  resourceScope
  parentRunId
  expiresAt
}
```

No permission inheritance by default.

## 16. Human-in-the-loop

Sensitive-data access, material metric-definition changes, production writes, consequential business recommendations.

HITL is represented as a durable runtime object, not as a TUI dependency:

```text
HumanTask {
  taskId
  reason
  proposedAction
  evidenceRefs[]
  alternatives[]
  requiredAuthority
  expiresAt?
}
```

## 17. Policies

Read-only by default; approved datasets only; PII minimization; semantic definitions override ad-hoc metric invention; production writes require explicit authority.

Policy decisions are explicit and auditable: `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REDACT`, `LIMIT`.

## 18. Identity, authorization, and secrets

Use least privilege, tenant isolation, short-lived delegated credentials, and a Credential Broker. Raw secrets must not be placed in model context.

## 19. Failure semantics

Required exercise:

> A query succeeds technically but returns a statistically misleading result due to insufficient sample size. The harness must distinguish execution success from analytical validity.

Failures should distinguish model, transport, tool, domain-validation, policy, human, and side-effect uncertainty.

## 20. Retry and idempotency

Retries depend on error semantics. A transient read can often be retried; an uncertain side effect must be reconciled before another attempt.

Use:
- idempotency keys;
- deduplication;
- status reconciliation;
- bounded exponential backoff;
- DLQ where messaging applies.

## 21. Durable execution

Persist checkpoints before/after consequential boundaries and before waiting for humans, external systems, schedules, or child agents.

```text
Checkpoint {
  runId
  stateVersion
  workflowPosition
  pendingOperations[]
  pendingHumanTasks[]
  createdAt
}
```

## 22. Events

Suggested event vocabulary:

```text
RunStarted
DomainTaskPlanned
CapabilityRequested
CapabilityCompleted
HumanReviewRequested
HumanDecisionReceived
DomainValidationFailed
SideEffectCommitted
RunCompleted
RunFailed
```

Domain packs add specialized events without changing the event primitive.

## 23. Artifacts

Artifacts are immutable/versioned outputs or evidence references. Store large outputs as artifact references instead of inflating run state.

## 24. Audit

Audit must reconstruct:
- activation and principal;
- state/version;
- model, skill, policy, and capability versions;
- context provenance;
- delegations;
- tool calls and side effects;
- human decisions;
- terminal outcome.

Operational logs are not a substitute for audit evidence.

## 25. Observability

Separate technical telemetry from business/domain observability.

Technical examples:
`latency`, `tokens`, `tool_failures`, `queue_wait`, `retries`, `child_runs`, `cost`.

Business outcomes:
Time-to-insight, reproducibility, analyst productivity, decision quality, cost per analysis.

## 26. Budgets / FinOps

Bound runs with combinations of:

```text
maxTurns
maxToolCalls
maxChildRuns
maxDelegationDepth
maxTokens
maxCost
maxWallClock
```

Budgets may be allocated per workflow branch.

## 27. Evals

Minimum domain eval suite:

query_correctness, metric_correctness, statistical_validity, data_leakage, reproducibility, visualization_fidelity, interpretation_faithfulness.

Evaluation Harness runs separately from Production Runtime and supports regression against versioned datasets.

## 28. Security and data governance

Every context item and artifact may carry classification, tenant, provenance, retention, and permitted-processing metadata. Policy determines whether a model, tool, transport, or external agent may receive it.

## 29. Core pseudocode

```text
FUNCTION handleActivation(rawInput):

    activation = triggerAdapter.normalize(rawInput)

    admission = admissionController.evaluate(activation)

    IF admission is not executable:
        RETURN admission

    target = activationRouter.route(activation)
    run = executionController.start(target)

    WHILE NOT run.isTerminal():

        checkpoint.save(run)

        step = workflow.next(run)
        context = contextEngine.build(step, run.state)

        decision = agentRuntime.decide(step, context)

        IF decision.hasIndependentOperations:
            graph = dependencyPlanner.build(decision.operations)
            results = executionFabric.execute(graph)
            run.state = reducer.apply(run.state, results)

        ELSE IF decision.requiresHuman:
            task = humanRuntime.create(decision)
            run.waitFor(task)

        ELSE IF decision.requiresDelegation:
            child = delegationManager.delegate(decision)
            run.waitForOrContinue(child)

        ELSE IF decision.isComplete:
            validation = domainValidator.validate(run.state)

            IF validation.passed:
                run.complete(validation.result)
            ELSE:
                run.state = reducer.applyValidation(run.state, validation)

    audit.finalize(run)
    outcomes.record(run)

    RETURN run.result
```

## 30. Student exercises

Sales dashboard; churn analysis; cohort analysis; demand forecast; dynamic pricing analysis; attribution; anomaly investigation; capacity analysis.

## 31. Required diagrams

Each implementation must provide:
1. Context diagram.
2. Component diagram.
3. Activation sequence.
4. Main workflow DAG.
5. HITL sequence.
6. Failure/recovery sequence.
7. Agent delegation graph when applicable.

## 32. Required ADRs

At least:
- ADR-001 — Why domain logic remains outside Generic Core.
- ADR-002 — State/context/memory boundary.
- ADR-003 — Side-effect and idempotency strategy.
- ADR-004 — HITL authority boundary.
- ADR-005 — Parallelism/delegation strategy where applicable.

## 33. Definition of Done

A student implementation is complete only when it demonstrates:
- normalized non-UI activation;
- admission;
- explicit state;
- bounded context engineering;
- skills and typed capabilities;
- tool-result evaluation;
- safe parallel/dependent execution;
- policy enforcement;
- HITL;
- failure recovery;
- durable checkpoint/resume;
- audit;
- technical observability;
- business outcomes;
- evals;
- Architecture Constitution compliance.

## 34. Conformance rule

> If multiple domains require the same primitive, consider moving the primitive into the Generic Harness. If only this domain requires the behavior, keep it in the Domain Pack.

The Reference Harness must extend the Generic Harness; it must not fork it.
