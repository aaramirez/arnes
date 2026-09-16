# RH-14 — Business Process & Workflow Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness

## 1. Mission
Coordinate long-running enterprise processes combining deterministic workflow, bounded agentic reasoning, humans, systems, events, deadlines, compensations and cross-domain dependencies.

> **Workflow owns process truth; agents provide bounded reasoning inside workflow steps.**

```text
Architecture Constitution
→ Generic Agent Harness
→ Business Process Domain Pack
→ Business Process & Workflow Harness
→ Specific Enterprise Process
```

## 2. Applicable processes
Order-to-Cash, Procure-to-Pay, customer/vendor onboarding, claims, project intake, service provisioning, field service, change management and similar processes.

## 3. Activations
User, API, form, webhook, queue, business event, database change, schedule, file arrival, email, another workflow or another agent. All normalize to `ActivationRequest`.

## 4. Process definition
```text
ProcessDefinition {
  processType, version, entryConditions[], steps[], transitions[],
  dependencies[], timers[], policies[], compensationRules[],
  terminalStates[]
}
```

Running instances retain their process-definition version.

## 5. Process state
```text
ProcessInstance {
  processId, processType, definitionVersion, businessKey, status,
  currentSteps[], completedSteps[], pendingEvents[], humanTasks[],
  childProcesses[], artifacts[], deadlines[], variables, auditRef
}
```

## 6. Step types
```text
DETERMINISTIC_SERVICE
AGENT_DECISION
HUMAN_TASK
WAIT_EVENT
TIMER
SUBPROCESS
PARALLEL_FORK
JOIN
POLICY_GATE
COMPENSATION
```

This distinction is architectural, not cosmetic.

## 7. Agents
Process Intake Agent; Decision Agent; Exception Agent; Document/Data Agent; Process Monitor Agent; Coordination Agent.

Agents do not own the complete process lifecycle.

## 8. Context engineering
Each agentic step receives only the process variables, documents/data, policies, evidence, permitted history, capabilities and output schema required for that decision.

```text
Process State ≠ Prompt
```

The Context Engine projects canonical process state into bounded model context.

## 9. Canonical hybrid workflow
```text
Activation
 ↓
Deterministic Validation
 ↓
Agentic Classification
 ↓
┌───────────────┬───────────────┐
│ System Task A │ System Task B │
└───────┬───────┴───────┬───────┘
        └───────┬───────┘
                ↓
              Join
                ↓
        Agentic Decision
                ↓
           Policy Gate
          ┌─────┴─────┐
       Auto Path   Human Task
          └─────┬─────┘
                ↓
            Side Effect
                ↓
             Wait Event
                ↓
             Complete
```

## 10. Dependencies
Dependencies are machine-readable:
`B requires A.success`; `C requires A+B`; `D starts on event X`; `E starts at deadline T`.

Do not hide workflow dependencies inside prompts.

## 11. Parallelism
Fork/join semantics may be `ALL`, `ANY`, `QUORUM`, `CONDITION`, or `FIRST_SUCCESS`.

## 12. Durable waits
Explicit states include `WAITING_FOR_CUSTOMER`, `WAITING_FOR_APPROVAL`, `WAITING_FOR_PAYMENT`, `WAITING_FOR_VENDOR`, `WAITING_FOR_EXTERNAL_SYSTEM`, `WAITING_UNTIL_DATE`.

No model session remains open while waiting.

## 13. Human tasks
```text
HumanTask {
  taskId, processId, stepId, reason, inputRefs[],
  proposedDecision?, alternatives[], authorityRequired,
  dueAt, escalationPolicy
}
```

HITL is channel-independent.

## 14. Subprocesses and delegation
```text
Parent Process
├─ Child Process A
├─ Child Process B
└─ Agent Run C
```

Children receive bounded input, authority, deadline, correlation ID and result contract.

## 15. A2A vs system integration
```text
Workflow → ERP API        = deterministic tool/system integration
Workflow → Payment API    = deterministic tool/system integration
Workflow → External Agent = A2A candidate
Workflow → Human          = Human Interaction contract
```

Do not "agentize" every integration.

## 16. Policies
- Workflow state is canonical process state.
- Agents cannot skip mandatory deterministic gates.
- Process version is immutable unless migration is explicit.
- Consequential side effects require authority/idempotency.
- Human-task deadlines have escalation.
- Compensation is modeled for reversible distributed actions.

## 17. Saga / compensation
```text
Reserve Inventory → Charge Payment → Create Shipment
```
If shipment fails, compensation may refund payment and release inventory. Compensation is not equivalent to database rollback.

## 18. Failure exercise
A three-system onboarding succeeds in CRM and billing but fails in provisioning. The harness must know committed steps, avoid replaying successful non-idempotent actions, choose retry vs compensation, preserve state and resume from the correct boundary.

## 19. Durable execution
Persist process state, workflow position, timers, events, human tasks, child processes, side-effect receipts, idempotency keys, compensations and deadlines.

## 20. Event contract
```text
BusinessEvent {
  eventId, eventType, businessKey, occurredAt, producer,
  payloadRef, correlationId, causationId?, version
}
```

Correlation and causation are distinct.

## 21. Audit
The complete process timeline must be reconstructable, including agent decisions, human decisions, waits, events, side effects and compensations.

## 22. Observability
Technical: queue lag, workflow/step latency, failures, retries, human wait, cost.  
Business: cycle time, completion, exceptions, SLA, manual touches, rework, straight-through rate.

## 23. Evals
`process_routing`, `agent_decision_quality`, `dependency_compliance`, `policy_gate_compliance`, `human_escalation`, `event_correlation`, `idempotency`, `compensation_correctness`, `resume_correctness`, `process_completion`.

## 24. Pseudocode
```text
FUNCTION advanceProcess(processId, incomingEvent?):
  process = repository.load(processId)
  IF incomingEvent: process.apply(incomingEvent)

  WHILE process.hasRunnableSteps():
    runnable = workflowEngine.runnable(process)
    graph = dependencyPlanner.build(runnable)
    results = executionFabric.execute(graph)

    FOR result IN results:
      IF result.type == AGENT_DECISION:
        validateAgentResult(result)
      IF result.type == HUMAN_WAIT:
        humanRuntime.create(result.task)
      IF result.sideEffect:
        process.recordReceipt(result)
      process.apply(result)

    checkpoint.save(process)

  IF process.requiresCompensation():
    compensationEngine.execute(process)

  IF process.terminal:
    outcomes.record(process)

  RETURN process.status
```

## 25. Student exercises
Customer onboarding; vendor onboarding; procure-to-pay; order-to-cash; insurance claim; field service; project intake; service provisioning.

## 26. Required ADRs
Workflow vs agent boundary; durable execution technology; event/correlation model; Saga/compensation; human-task architecture; process versioning.

## 27. Definition of Done
Deterministic + agentic steps, multiple activation modes, explicit dependencies, fork/join, events, timers, HITL, subprocesses, bounded delegation, durable waits, idempotency, compensation, audit, outcomes and evals.

## 28. Constitution test
If the Agent Loop becomes the business-process engine, the design has failed. Workflow owns lifecycle/dependencies; Agent Runtime owns bounded reasoning.
