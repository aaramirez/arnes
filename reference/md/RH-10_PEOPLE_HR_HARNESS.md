# RH-10 — People & HR Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness  
**Category:** Human-Centered Enterprise Operations

---

## 1. Mission

The People & HR Harness coordinates employee-service, recruiting-operations, onboarding, offboarding, learning, and HR case workflows while preserving privacy, fairness, authority boundaries, and human accountability.

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
People & HR Domain Pack
        ↓
People & HR Reference Harness
        ↓
Specific HR Use Case
```

A central invariant is:

> **The harness may assist a people decision, but model output does not itself constitute employment authority.**

---

## 2. Actors

- Employee
- Candidate
- Hiring manager
- Recruiter
- HR Operations
- HR Business Partner
- Benefits administrator
- IT / Facilities
- Payroll
- Compliance / Legal
- Human approver

---

## 3. Business outcomes

```text
onboarding_cycle_time
case_resolution_time
employee_self_service_rate
handoff_rate
policy_answer_accuracy
process_completion_rate
manual_coordination_hours
privacy_incidents
human_override_rate
```

The harness should optimize operational work without optimizing away required human judgment.

---

## 4. Activation sources

```text
Employee Request
Candidate Event
HRIS Event
ATS Event
Manager Request
Onboarding Event
Offboarding Event
Schedule
API
Another Workflow
        ↓
Trigger Adapter
        ↓
ActivationRequest
```

Examples:

```text
EMPLOYEE_HELP_REQUEST
NEW_HIRE_CREATED
CANDIDATE_STAGE_CHANGED
EMPLOYEE_TERMINATION_CONFIRMED
BENEFITS_ENROLLMENT_WINDOW
TRAINING_OVERDUE
```

---

## 5. Admission

Evaluate:

- identity;
- employee/candidate relationship;
- tenant/legal entity;
- requested operation;
- data classification;
- authority;
- consent where required;
- jurisdiction;
- duplicate workflow;
- risk category.

Possible outcomes:

`ACCEPT`, `REJECT`, `ROUTE`, `REQUIRE_APPROVAL`, `REDACT`, `DEFER`.

---

## 6. Agents

### Employee Service Agent
Handles policy and service requests.

### HR Knowledge Agent
Retrieves authoritative HR policy and benefits information.

### Onboarding/Offboarding Agent
Coordinates deterministic cross-functional tasks.

### Recruiting Operations Agent
Supports scheduling, process tracking, communications, and preparation.

### Case Routing Agent
Routes sensitive or exceptional matters to qualified humans.

### Process Monitor Agent
Detects missing tasks, SLA breaches, and blocked dependencies.

Agents do not make autonomous high-consequence employment decisions merely because a model can generate a recommendation.

---

## 7. State

```text
EmployeeCaseState {
  caseId
  employeeRef
  caseType
  jurisdiction
  classification
  status
  assignedAuthority
  tasks[]
  evidenceRefs[]
  humanDecisions[]
}
```

```text
OnboardingState {
  employeeRef
  startDate
  roleRef
  managerRef
  requiredTasks[]
  dependencies[]
  completedTasks[]
  blockedTasks[]
  accessProvisioningState
}
```

```text
CandidateProcessState {
  candidateRef
  requisitionRef
  stage
  scheduledEvents[]
  permittedDataRefs[]
  communications[]
  humanDecisionRefs[]
}
```

---

## 8. Context engineering

Potential context:

- applicable HR policy;
- employee/candidate permitted profile;
- jurisdiction;
- role/requisition;
- benefit plan;
- process state;
- prior case information when authorized;
- manager/organization data;
- deadlines;
- authority matrix.

Context must be minimized by task.

A scheduling agent does not need compensation history. A benefits FAQ agent does not need performance records.

---

## 9. Memory boundaries

Separate:

```text
HRIS / ATS System of Record
Case State
Workflow State
Knowledge / Policy
Permitted Interaction Memory
Artifacts
```

Conversational memory must never become an unofficial personnel file.

---

## 10. Skills

```text
answer-hr-policy
route-employee-case
coordinate-onboarding
coordinate-offboarding
schedule-interview
prepare-interview-package
track-process
prepare-human-review
assign-training
follow-up-task
```

---

## 11. Capabilities

```text
hris.read
ats.read/update
calendar.read/create
email.draft/send
ticket.create/update
knowledge.search
identity.lookup
training.assign
workflow.task.create
document.generate
```

Capabilities expose risk and side-effect metadata.

---

## 12. Canonical onboarding workflow

```text
NEW_HIRE_CREATED
      ↓
Admission
      ↓
Load Role / Manager / Start Date
      ↓
Create Onboarding Plan
      ↓
┌─────────────┬──────────────┬──────────────┐
│ IT Access   │ HR Documents │ Manager Tasks│
└──────┬──────┴──────┬───────┴──────┬───────┘
       └─────────────┼───────────────┘
                     ↓
             Dependency Checks
                     ↓
             Missing / Blocked?
               ├─ YES → Escalate
               └─ NO
                     ↓
                Start-Date Ready
```

Independent preparation tasks can execute in parallel.

---

## 13. Recruiting workflow boundary

```text
Candidate Information
        ↓
Operational Assistance
        ↓
Human Evaluation / Decision
        ↓
Recorded Decision
        ↓
Authorized Process Action
```

The harness can coordinate and summarize. It must not silently convert model scoring into an employment decision.

---

## 14. HITL

Required for configured high-consequence decisions, including:

- hiring decision;
- rejection when governed by human decision policy;
- compensation;
- performance action;
- termination;
- accommodation/sensitive cases;
- policy exception;
- sensitive employee investigations.

```text
HumanDecision {
  decisionId
  decisionType
  authority
  evidenceRefs[]
  decision
  rationale?
  timestamp
}
```

---

## 15. Policies

```text
POL-HR-001:
  Access only data required for the current HR task.

POL-HR-002:
  Sensitive personnel data is not delegated to unauthorized agents.

POL-HR-003:
  Employment authority remains with authorized humans/systems.

POL-HR-004:
  Jurisdiction-specific policy is resolved before advice/action.

POL-HR-005:
  Offboarding access revocation follows deterministic dependencies.

POL-HR-006:
  Model inference must not create unsupported employee facts.
```

---

## 16. Fairness and decision support

Separate:

```text
Process Automation
Decision Support
Decision Authority
```

Evaluation should detect whether outputs systematically vary improperly across protected or proxy attributes when such analysis is legally and ethically appropriate.

The safest design is to exclude unnecessary sensitive attributes from model context.

---

## 17. Agent communication

Example onboarding:

```text
HR Orchestrator
 ├── Task → IT Provisioning Agent
 ├── Task → Facilities Agent
 ├── Task → Learning Agent
 └── Task → Manager Coordination Agent
```

Delegation includes only required identity/resource scope.

External independent systems may communicate through A2A or conventional APIs/events. A2A is not required for ordinary deterministic system integration.

---

## 18. Failure scenario

**Scenario:** onboarding creates an IT provisioning request, but the acknowledgement is lost.

The harness must:

1. preserve the intended task;
2. query provisioning status using correlation/idempotency identifiers;
3. avoid creating duplicate accounts;
4. resume downstream tasks only when dependencies are satisfied;
5. escalate if reconciliation cannot establish state.

---

## 19. Durable execution

HR workflows can span days or weeks.

Persist:

```text
workflow position
pending tasks
dependencies
human tasks
external correlation IDs
deadlines
SLA timers
decisions
artifacts
```

---

## 20. Events

```text
EmployeeCaseOpened
NewHireCreated
OnboardingTaskCreated
OnboardingTaskCompleted
OnboardingBlocked
HumanDecisionRequired
HumanDecisionReceived
OffboardingStarted
AccessRevocationConfirmed
CaseClosed
```

---

## 21. Audit

Audit should reconstruct:

- activation;
- identity;
- accessed data classes;
- policies used;
- delegations;
- communications;
- human decisions;
- external side effects;
- final process outcome.

Access to audit itself must be controlled.

---

## 22. Observability

Technical:
- workflow latency;
- integration failures;
- retries;
- human wait time;
- cost.

HR outcomes:
- onboarding readiness;
- case SLA;
- self-service resolution;
- blocked task count;
- handoff rate;
- process completion.

---

## 23. Evals

```text
policy_answer_accuracy
case_routing_accuracy
privacy_minimization
jurisdiction_selection
workflow_completion
handoff_quality
unsupported_inference_detection
fairness_regression
side_effect_safety
```

---

## 24. Security and governance

HR data typically requires stronger controls:

- field-level access;
- tenant/legal-entity isolation;
- classification;
- retention;
- purpose limitation;
- model/provider restrictions;
- redaction;
- regional processing constraints;
- detailed access audit.

---

## 25. Pseudocode

```text
FUNCTION processHRActivation(raw):

  activation = normalize(raw)
  admission = admission.evaluate(activation)

  IF NOT admission.executable:
      RETURN admission

  state = hrState.loadOrCreate(activation)

  WHILE NOT state.terminal:

      step = workflow.next(state)
      context = contextEngine.buildMinimumNecessary(step, state)

      policy = policyEngine.evaluate(step, context)

      IF policy.requiresHuman:
          task = humanRuntime.create(step, policy)
          checkpoint.save(state.waitFor(task))
          WAIT

      operations = agentRuntime.plan(step, context)

      graph = dependencyPlanner.build(operations)
      results = executionFabric.execute(graph)

      state = reducer.apply(state, results)
      validator.validate(state)
      checkpoint.save(state)

  RETURN state.outcome
```

---

## 26. Student exercises

1. Employee help desk.
2. New-hire onboarding.
3. Offboarding.
4. Interview coordination.
5. Benefits Q&A.
6. Training assignment.
7. HR case routing.
8. Recruiting operations.

---

## 27. Required ADRs

- HR data minimization strategy.
- Human decision-authority boundary.
- Cross-functional workflow orchestration.
- Jurisdiction/policy resolution.
- External side-effect reconciliation.

---

## 28. Definition of Done

The implementation demonstrates non-chat activation, privacy-aware context construction, durable workflows, explicit human authority, cross-system dependencies, bounded delegation, safe side effects, audit, HR outcome metrics, and an eval suite.

---

## 29. Constitution test

If the Generic Core contains concepts such as `candidate`, `employee`, `benefits`, or `onboarding`, the boundary is probably wrong.

Those belong to the People & HR Domain Pack.
