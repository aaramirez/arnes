# RH-06 — Sales & CRM Reference Harness
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
RH-06 — Sales & CRM
        ↓
Specific Use Case
```

This specification must reuse Generic Harness primitives. Domain-specific behavior belongs in agents, skills, policies, workflows, capabilities, validators, and context providers—not in the core loop.

## 2. Purpose

Coordinate lead, account, opportunity, and follow-up work while keeping commercial authority and customer communication governed.

## 3. Actors

Lead, Account Executive, SDR/BDR, Sales Manager, CRM, Marketing, Customer.

## 4. Activation sources

Lead form, CRM stage change, inactivity event, inbound email, meeting event, schedule, another agent.

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

Lead Qualification Agent; Account Research Agent; Opportunity Agent; Follow-up Agent; Sales Manager/Decision Support Agent.

Agents communicate through explicit task/result contracts. They do not depend on shared implicit memory.

## 7. State

LeadState, AccountState, OpportunityState, EngagementState, NextActionState, ApprovalState.

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

CRM records, communications, product catalog, pricing, account intelligence, sales playbooks, territory and discount policies.

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

qualify-lead, research-account, summarize-engagement, recommend-next-action, draft-outreach, prepare-meeting, update-crm.

Skills are reusable domain instruction packages and may be versioned independently.

## 11. Capabilities

crm.read/update, email.draft/send, calendar.read/create, research.search, quote.prepare, analytics.read.

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
Sales Event → Load CRM Context → Research/Qualify → Recommend Next Action → Human/Policy Gate → Communicate/Update CRM → Observe Outcome.
```

## 13. Parallelism and dependencies

Account research and engagement summarization can run concurrently; outreach waits for qualification, policy, and required human approval.

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

External outreach under configured conditions, discounts, commercial commitments, opportunity close/loss decisions, sensitive account changes.

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

CRM is system of record; no invented customer facts; discount authority is delegated explicitly; communication frequency limits; external messages preserve approved claims.

Policy decisions are explicit and auditable: `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REDACT`, `LIMIT`.

## 18. Identity, authorization, and secrets

Use least privilege, tenant isolation, short-lived delegated credentials, and a Credential Broker. Raw secrets must not be placed in model context.

## 19. Failure semantics

Required exercise:

> An email send returns an uncertain status. The harness must reconcile delivery before retrying to prevent duplicate customer outreach.

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
Conversion rate, pipeline velocity, seller productivity, revenue influence, follow-up latency, CRM completeness.

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

qualification_accuracy, crm_update_accuracy, next_action_quality, outreach_grounding, policy_compliance, duplicate_action_prevention.

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

Lead qualification; dormant opportunity recovery; account planning; renewal assistance; meeting preparation; cross-sell; pipeline hygiene; outbound follow-up.

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
