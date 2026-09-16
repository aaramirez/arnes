# RH-07 — Marketing & Creative Reference Harness
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

Plan, create, review, approve, publish, and optimize marketing campaigns and creative assets while preserving brand, factual, legal, channel, and budget constraints.

## 4. Actors

Marketing requester, strategist, creative team, brand owner, legal/compliance reviewer, media buyer, channel owner, analytics team.

## 5. Activation sources

Campaign brief, content request, editorial schedule, performance anomaly, product launch event, asset request, API, another agent.

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

Strategy Agent; Audience/Research Agent; Copy Agent; Creative Agent; Brand Reviewer; Compliance Reviewer; Performance Analyst.

Agents exchange explicit `AgentTaskRequest`, `AgentProgress`, and `AgentTaskResult` contracts. Shared hidden memory is not an integration mechanism.

## 7. State

`CampaignState`, `AudienceState`, `ContentState`, `AssetState`, `ApprovalState`, `ChannelState`, `PerformanceState`, `ExperimentState`.

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

Brand kit, audience segments, product truth, approved claims, campaign history, channel rules, creative assets, performance metrics, legal constraints.

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

`build-campaign-strategy`, `research-audience`, `write-copy`, `create-creative-brief`, `review-brand`, `review-claims`, `plan-experiment`, `analyze-performance`, `recommend-optimization`.

Skills package reusable domain instructions and constraints and should be versioned independently from the runtime.

## 11. Capabilities

`brand.search`, `asset.read/write`, `creative.generate`, `cms.publish`, `social.publish`, `email.prepare/send`, `analytics.read`, `experiment.create`, `calendar.schedule`.

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
Brief → Research → Strategy → Content/Creative Production → Brand & Claim Review → Human Approval → Publish → Measure → Learn/Optimize.
```

## 13. Parallelism and dependencies

Audience research, copy variants, and visual concepts can run in parallel after strategy constraints exist. Publishing depends on brand/compliance/approval gates. Optimization depends on sufficient performance data.

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

External publication, paid-media spend, new product claims, high-risk audience targeting, brand exceptions, material campaign changes.

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

No fabricated product claims; approved brand assets/rules are authoritative; publication/spend permissions are separate; regulated claims require compliance review; audience data follows privacy/consent rules.

Policy outcomes are explicit: `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REDACT`, `LIMIT`.

## 18. Identity, authorization, secrets

Use least privilege, tenant isolation, delegated credentials, short-lived tokens, and a Credential Broker. Secrets are injected at execution time and must not be exposed in model context.

## 19. Domain-specific architecture

### Creative artifact lifecycle

```text
IDEA → DRAFT → BRAND_REVIEW → COMPLIANCE_REVIEW
     → APPROVED → SCHEDULED → PUBLISHED → MEASURED → ARCHIVED
```

### Campaign graph

```text
Campaign
├── Audience
├── Messages
├── Claims
├── Assets
├── Channels
├── Experiments
└── Metrics
```

Claims should reference product truth/evidence. Generated creative is an artifact, not state text.

### Learning loop

```text
Performance Event
      ↓
Analytics Context
      ↓
Performance Agent
      ↓
Hypothesis / Recommendation
      ↓
Experiment Plan
      ↓
Human/Budget Gate
      ↓
New Variant
```

The system must distinguish correlation from validated experiment results.

## 20. Failure semantics

Required scenario:

> A publish call times out after the platform may have accepted the post. The harness must reconcile publication status before retrying to avoid duplicate posts.

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

Campaign cycle time, content throughput, conversion contribution, engagement quality, cost per approved asset, brand defect rate, experiment learning velocity.

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

`brand_compliance`, `claim_grounding`, `audience_fit`, `copy_quality`, `creative_brief_quality`, `channel_compliance`, `publication_correctness`, `experiment_validity`.

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

Social campaign; product launch; email campaign; SEO program; creative A/B test; editorial calendar; brand compliance workflow; performance optimization.

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
