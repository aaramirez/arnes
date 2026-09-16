# RH-09 — Finance & Accounting Reference Harness
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
Reference Harness
        ↓
Specific Use Case
```

Domain behavior belongs in Domain Packs, agents, skills, policies, workflows, validators, context providers, and capability adapters. It must not be embedded in the Generic Core.

## 2. Common runtime path

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
Context + State + Policy
      ↓
Model / Skills / Capabilities
      ↓
Validation / HITL / Side Effects
      ↓
Checkpoint + Audit + Outcomes
```

The model is replaceable. Protocols and transports are adapters. Human interaction is a runtime contract, not a UI dependency.


## 3. Purpose

Automate and assist financial operations under strict controls for authority, segregation of duties, reconciliation, idempotency, evidence, and immutable audit.

## 4. Actors

Finance analyst, AP/AR specialist, accountant, approver, controller, auditor, vendor/customer, ERP/payment systems.

## 5. Activation sources

Invoice arrival, ERP event, expense submission, payment exception, reconciliation schedule, close event, budget anomaly, API, another workflow.

All sources normalize to the same `ActivationRequest`. Admission evaluates identity, tenant, authorization, risk, duplication, budget, data classification, and workflow availability.

Possible decisions:

```text
ACCEPT
REJECT
DEFER
DEDUPLICATE
ROUTE
REQUIRE_APPROVAL
```

## 6. Agents

Finance Operations Agent; Invoice/Document Agent; Reconciliation Agent; Exception Analyst; Approval Coordinator; Financial Reviewer.

Agents exchange explicit `AgentTaskRequest`, `AgentProgress`, and `AgentTaskResult` contracts. Shared hidden memory is not an integration mechanism.

## 7. State

`FinancialCaseState`, `InvoiceState`, `ExpenseState`, `ReconciliationState`, `ApprovalState`, `PaymentState`, `LedgerActionState`, `EvidenceState`.

Common envelope:

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

## 8. Context engineering

ERP/ledger data, invoice/expense documents, vendor master, purchase orders, approval matrix, budgets, accounting policies, prior transactions, reconciliation evidence.

Context is assembled intentionally and carries provenance, classification, version, and access metadata. The complete enterprise corpus is never assumed to fit in a prompt.

## 9. Memory boundaries

Keep separate:

```text
Execution State
Durable Domain State
Retrievable Knowledge
Permitted Long-Term Memory
Artifacts / Evidence
```

The model context is not the system of record.

## 10. Skills

`validate-invoice`, `match-po`, `detect-duplicate`, `reconcile`, `analyze-exception`, `prepare-approval`, `explain-variance`, `prepare-journal`, `verify-payment-status`.

Skills package reusable domain instructions and constraints and should be versioned independently from the runtime.

## 11. Capabilities

`erp.read/write`, `ledger.read`, `invoice.extract`, `vendor.lookup`, `po.lookup`, `payment.status/execute`, `approval.request`, `artifact.write`, `analytics.query`.

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

The Tool Runtime, not the model, owns authorization and execution.

## 12. Canonical workflow

```text
Financial Event → Extract/Load → Validate → Match/Reconcile → Exception Analysis → Policy/Authority Decision → Human Approval if required → Side Effect → Reconcile Result → Audit.
```

## 13. Parallelism and dependencies

Document extraction, vendor lookup, PO lookup, and duplicate checks can run in parallel. Payment/journal actions wait for validation, segregation-of-duties, authority, and approval dependencies.

The Execution Fabric enforces the DAG. Model output may propose work but cannot bypass dependencies.

## 14. Agent communication and federation

```text
Agent A
  ↓
AgentCommunicationGateway
  ├── Internal Adapter → Managed Agent
  └── A2A Adapter      → Independent Agent
```

A2A is optional and appropriate for interoperating with independent agent systems. Webhooks, HTTP, queues, WebSockets, SSE, gRPC, Kafka, or service buses may be transports depending on requirements. Semantic contracts remain transport-independent.

## 15. Delegated authority

Child agents receive bounded authority:

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

Authority is never inherited implicitly.

## 16. Human-in-the-loop

Payment authorization, journal posting, exception overrides, threshold approvals, vendor-master changes, material write-offs, policy exceptions.

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

The run checkpoints and waits. The approval may arrive through web, mobile, email, chat, ticketing, or an API.

## 17. Policies

Segregation of duties is mandatory; agent cannot both create and approve a controlled action when policy forbids it; approval thresholds are deterministic; payment authority is delegated explicitly; financial side effects require idempotency/reconciliation; audit evidence is immutable.

Policy outcomes are explicit: `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REDACT`, `LIMIT`.

## 18. Identity, authorization, secrets

Use least privilege, tenant isolation, delegated credentials, short-lived tokens, and a Credential Broker. Secrets are injected at execution time and must not be exposed in model context.

## 19. Domain-specific architecture

### Segregation of duties

Authority is modeled independently from reasoning quality.

```text
Prepare Action
     ↓
Policy Engine
     ↓
Authority Matrix
  ┌──┴──────────────┐
  │                 │
Auto-permitted   Approval Required
  │                 ↓
  │             Authorized Human
  └──────────┬──────┘
             ↓
       Execute Side Effect
             ↓
          Reconcile
             ↓
       Immutable Audit
```

Example deterministic thresholds:

```text
amount < configured_auto_limit
    → policy may allow automation

amount >= approval_limit
    → authorized approver required

high_risk / vendor_change / exception
    → enhanced or dual approval
```

Threshold values belong in configuration/policy, not prompts.

### Financial action contract

```text
FinancialAction {{
  actionId
  actionType
  amount?
  currency?
  resourceRefs[]
  preparedBy
  requiredAuthority
  approvals[]
  idempotencyKey
  status
}}
```

### Critical invariant

> A model recommendation never constitutes financial authority.

The harness must prove who authorized every consequential financial side effect.

## 20. Failure semantics

Required scenario:

> A payment request times out after submission and its commit status is unknown. The harness must query payment status/reconcile by idempotency key before deciding whether another attempt is safe.

Distinguish runtime failure, domain-validation failure, policy denial, human rejection, and uncertain side effects.

## 21. Retry, idempotency, and delivery

Use retry only when error semantics allow it. Consequential side effects require idempotency or reconciliation.

For asynchronous delivery consider:

```text
acknowledgement
deduplication
bounded retry
ordering requirements
backpressure
dead-letter queue
status reconciliation
```

## 22. Durable execution

Checkpoint before and after consequential boundaries and before waiting on humans, child agents, external systems, schedules, or asynchronous callbacks.

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

## 23. Events

Base vocabulary:

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

Domain events extend this vocabulary without modifying the core event primitive.

## 24. Artifacts

Large results, evidence, generated assets, reports, and snapshots are versioned artifacts referenced from state.

## 25. Audit

Audit must reconstruct activation, principal, context provenance, versions, delegations, policy decisions, tool calls, side effects, human decisions, and terminal outcome. Operational telemetry is not a substitute for immutable audit evidence.

## 26. Observability and business outcomes

Technical telemetry includes latency, tokens, cost, queue wait, tool failures, retries, child runs, and checkpoint/resume behavior.

Domain outcomes:

Straight-through processing rate, exception rate, reconciliation time, close-cycle time, duplicate-payment prevention, cost per transaction, audit readiness.

## 27. Budgets / FinOps

Bound executions using combinations of:

```text
maxTurns
maxToolCalls
maxChildRuns
maxDelegationDepth
maxTokens
maxCost
maxWallClock
```

Budgets may be allocated to workflow branches or specialized agents.

## 28. Evals

Minimum domain eval suite:

`invoice_extraction`, `duplicate_detection`, `matching_accuracy`, `reconciliation_accuracy`, `policy_compliance`, `approval_routing`, `payment_idempotency`, `financial_explanation_accuracy`.

Evaluation runs outside Production Runtime against versioned regression datasets.

## 29. Security and data governance

Context and artifacts may carry tenant, classification, provenance, retention, geographic, and permitted-processing metadata. Policy decides whether a model, tool, protocol, transport, or external agent can receive them.

## 30. Generic orchestration pseudocode

```text
FUNCTION handleActivation(rawInput):

    activation = triggerAdapter.normalize(rawInput)
    admission = admissionController.evaluate(activation)

    IF NOT admission.executable:
        RETURN admission

    run = executionController.start(
        activationRouter.route(activation)
    )

    WHILE NOT run.isTerminal():

        checkpoint.save(run)

        step = workflow.next(run)
        context = contextEngine.build(step, run.state)
        decision = agentRuntime.decide(step, context)

        SWITCH decision.type:

          CASE OPERATIONS:
            graph = dependencyPlanner.build(decision.operations)
            results = executionFabric.execute(graph)
            run.state = reducer.apply(run.state, results)

          CASE DELEGATE:
            child = delegationManager.delegate(decision)
            workflow.registerChild(run, child)

          CASE HUMAN:
            task = humanRuntime.create(decision)
            run.waitFor(task)

          CASE COMPLETE:
            validation = domainValidator.validate(run.state)

            IF validation.passed:
                run.complete(validation.result)
            ELSE:
                run.state = reducer.applyValidation(
                    run.state, validation
                )

    audit.finalize(run)
    outcomes.record(run)

    RETURN run.result
```

## 31. Student exercises

Invoice processing; expense approval; bank reconciliation; collections prioritization; payment exception; variance analysis; month-end assistance; budget monitoring.

## 32. Required failure/recovery demonstration

Students must implement the failure scenario from §20 and prove:
1. state remains consistent;
2. side effects are not duplicated;
3. retry/reconciliation follows explicit semantics;
4. audit reconstructs what happened;
5. the workflow either resumes safely, escalates, or terminates explicitly.

## 33. Required diagrams

1. Context diagram.
2. Component diagram.
3. Activation sequence.
4. Main workflow DAG.
5. HITL sequence.
6. Failure/recovery sequence.
7. Agent delegation graph when applicable.

## 34. Required ADRs

At least:
- ADR-001 — Domain vs Generic Core boundary.
- ADR-002 — State/context/memory boundary.
- ADR-003 — Side-effect/idempotency strategy.
- ADR-004 — HITL authority boundary.
- ADR-005 — Parallelism/delegation strategy.

## 35. Definition of Done

The implementation demonstrates normalized activation, admission, explicit state, bounded context, typed capabilities, tool-result evaluation, dependency-aware execution, policy enforcement, HITL, durable checkpoint/resume, side-effect control, audit, observability, business outcomes, evals, and Architecture Constitution compliance.

## 36. Conformance rule

> If multiple domains require the same primitive, evaluate moving that primitive into the Generic Harness. If only this domain requires the behavior, keep it in the Domain Pack.

The Reference Harness extends the Generic Harness; it does not fork it.
