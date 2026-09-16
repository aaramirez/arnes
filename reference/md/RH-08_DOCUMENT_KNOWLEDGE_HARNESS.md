# RH-08 — Document & Knowledge Reference Harness
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

Ingest, extract, classify, enrich, govern, index, retrieve, and lifecycle-manage enterprise documents and knowledge while preserving provenance and access controls.

## 4. Actors

Knowledge worker, records manager, document owner, compliance officer, search user, source system, knowledge curator.

## 5. Activation sources

File upload, email attachment, repository event, folder/watch event, scheduled crawl, API, document update/delete, knowledge query.

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

Document Intake Agent; Extraction Agent; Classifier; Knowledge Curator; Retrieval Agent; Quality/Provenance Reviewer.

Agents exchange explicit `AgentTaskRequest`, `AgentProgress`, and `AgentTaskResult` contracts. Shared hidden memory is not an integration mechanism.

## 7. State

`DocumentState`, `ExtractionState`, `ClassificationState`, `KnowledgeItem`, `IndexState`, `ProvenanceState`, `RetentionState`, `AccessState`.

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

Source document, metadata, taxonomy, schemas, ACLs, related documents, retention policy, authoritative knowledge sources, index metadata.

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

`extract-structure`, `extract-fields`, `classify-document`, `summarize`, `link-knowledge`, `detect-duplicates`, `index`, `retrieve`, `validate-provenance`, `apply-retention`.

Skills package reusable domain instructions and constraints and should be versioned independently from the runtime.

## 11. Capabilities

`document.read`, `parser.extract`, `ocr.extract` when necessary, `storage.read/write`, `index.upsert/delete`, `search.query`, `metadata.read/write`, `acl.resolve`.

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
Document Event → Admission/ACL → Parse/Extract → Classify → Validate → Enrich/Link → Index → Quality Gate → Available for Retrieval → Lifecycle Events.
```

## 13. Parallelism and dependencies

Independent extraction of tables, metadata, and sections may run in parallel. Indexing waits for required validation/classification. Retrieval filters authorization before context assembly.

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

Low-confidence extraction, ambiguous records classification, retention/disposition, restricted-data reclassification, authoritative-source conflicts.

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

Access control travels with knowledge; source provenance cannot be removed; generated summaries are derivative artifacts; deletion/retention events propagate to indexes; retrieval must enforce current ACLs.

Policy outcomes are explicit: `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REDACT`, `LIMIT`.

## 18. Identity, authorization, secrets

Use least privilege, tenant isolation, delegated credentials, short-lived tokens, and a Credential Broker. Secrets are injected at execution time and must not be exposed in model context.

## 19. Domain-specific architecture

### Knowledge lineage

```text
Source Document v7
   ↓ extraction
Extracted Facts
   ↓ validation
Knowledge Items
   ↓ indexing
Search Index
   ↓ retrieval
ContextBundle
```

Every derivative keeps a lineage reference to the source/version.

### Retrieval security

Correct order:

```text
Query
 ↓
Identity / Authorization
 ↓
Permitted Search Scope
 ↓
Retrieve
 ↓
Re-rank
 ↓
ContextBundle
```

Not:

```text
Retrieve everything → filter after model sees it
```

### Lifecycle

```text
INGESTED → EXTRACTED → CLASSIFIED → VALIDATED
         → INDEXED → ACTIVE → SUPERSEDED / RETAINED / DELETED
```

Supersession and deletion are first-class events.

## 20. Failure semantics

Required scenario:

> A source document is deleted or access-revoked while an indexed derivative remains. The harness must invalidate/remove unauthorized derived knowledge and prevent stale retrieval.

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

Search success, knowledge reuse, document processing time, manual classification reduction, stale-knowledge rate, retrieval quality.

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

`extraction_accuracy`, `classification_accuracy`, `retrieval_precision`, `retrieval_recall`, `acl_enforcement`, `provenance_integrity`, `duplicate_detection`, `deletion_propagation`.

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

Contract ingestion; policy knowledge base; technical-document index; email attachment processing; enterprise search; records classification; knowledge consolidation; FAQ knowledge service.

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
