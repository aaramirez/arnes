# RH-12 — IT & Cloud Operations Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness  
**Category:** Event-Driven Operational Automation

---

## 1. Mission

The IT & Cloud Operations Harness detects, investigates, coordinates, remediates, verifies, and learns from application, platform, infrastructure, cloud, CI/CD, and operational incidents.

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
IT & Cloud Operations Domain Pack
        ↓
IT & Cloud Operations Harness
```

Central invariant:

> **Detection, diagnosis, authority, execution, and verification are separate concerns.**

---

## 2. Actors

- SRE / Operations engineer
- Developer
- Incident commander
- Service owner
- Security team
- Cloud platform
- Observability platform
- CI/CD platform
- ITSM
- Automated remediation agent

---

## 3. Business outcomes

```text
MTTD
MTTA
MTTR
diagnosis_accuracy
auto_remediation_rate
rollback_rate
change_failure_rate
repeat_incident_rate
human_pages_avoided
service_availability
```

---

## 4. Activation sources

This harness should strongly exercise non-user activation.

```text
Metrics Alert
Log Alert
Trace Anomaly
Cloud Event
Kubernetes Event
CI Failure
Deployment Event
Ticket
Schedule
Webhook
Queue
Event Bus
Another Agent
        ↓
ActivationGateway
```

---

## 5. Admission and event control

Before starting an expensive agent run:

- validate source;
- deduplicate;
- correlate related alerts;
- suppress known noise;
- identify service/tenant;
- determine severity;
- apply maintenance windows;
- enforce rate/budget limits.

```text
100 raw alerts
      ↓
Correlation / Deduplication
      ↓
3 incident candidates
      ↓
Agent activation
```

This prevents an "agent storm."

---

## 6. Agents

### Incident Triage Agent
Classifies severity and affected service.

### Diagnostic Agent
Builds and tests hypotheses.

### Topology/Dependency Agent
Understands service relationships and blast radius.

### Remediation Agent
Selects approved runbooks/actions.

### Change Reviewer
Evaluates risk before consequential remediation.

### Verification Agent
Determines whether service recovered.

### Incident Communication Agent
Produces stakeholder updates from verified state.

---

## 7. State

```text
IncidentState {
  incidentId
  serviceRefs[]
  severity
  symptoms[]
  correlatedSignals[]
  hypotheses[]
  evidence[]
  actions[]
  approvals[]
  recoveryState
  status
}
```

```text
Hypothesis {
  hypothesisId
  statement
  evidenceFor[]
  evidenceAgainst[]
  confidence
  testPlan[]
  status
}
```

```text
RemediationAction {
  actionId
  runbookRef
  target
  riskLevel
  expectedEffect
  rollbackPlan
  requiredAuthority
  idempotencyKey?
  status
}
```

---

## 8. Context engineering

Operational context may include:

- metrics;
- logs;
- traces;
- topology;
- recent deployments;
- configuration;
- incidents;
- runbooks;
- service ownership;
- SLOs;
- cloud resource state;
- change calendar.

Use time windows and service boundaries aggressively to prevent context explosion.

---

## 9. Skills

```text
triage-incident
correlate-signals
analyze-topology
form-hypothesis
test-hypothesis
select-runbook
estimate-blast-radius
execute-remediation
verify-recovery
prepare-incident-update
prepare-postmortem
```

---

## 10. Capabilities

```text
metrics.query
logs.query
traces.query
topology.read
cloud.read
cloud.change
kubernetes.read
kubernetes.change
cicd.read/trigger
itsm.read/update
runbook.execute
notification.send
```

Read and change capabilities must have different authority classes.

---

## 11. Investigation workflow

```text
Operational Event
      ↓
Correlation / Admission
      ↓
Create / Attach Incident
      ↓
Collect Signals
      ↓
┌────────────┬─────────────┬──────────────┐
│ Logs       │ Metrics     │ Recent Change│
└─────┬──────┴──────┬──────┴──────┬───────┘
      └─────────────┼──────────────┘
                    ↓
              Hypotheses
                    ↓
           Test / Gather Evidence
                    ↓
              Root Cause?
        ┌───────────┴────────────┐
        │                        │
       NO                       YES
        ↓                        ↓
  Continue bounded         Remediation Plan
   investigation                  ↓
                           Policy / Human Gate
                                  ↓
                              Execute
                                  ↓
                              Verify
```

---

## 12. Parallelism

Excellent domain for parallel tool use:

```text
query logs ─────────┐
query metrics ──────┼── parallel
read deployment ───┤
read topology ──────┘
```

But:

```text
change config
   ↓
wait propagation
   ↓
verify health
   ↓
next action
```

must be sequential where causality matters.

---

## 13. Event-driven execution fabric

```text
Event Bus
   ↓
Correlation Service
   ↓
Activation
   ↓
Incident Workflow
   ↓
Agent Runs
   ↓
Tool Runtime
   ↓
Cloud / K8s / CI
```

Long-running workflows should not require a permanently open model session.

---

## 14. Agent-to-agent communication

An incident may spawn specialists:

```text
Incident Coordinator
 ├── Application Agent
 ├── Database Agent
 ├── Kubernetes Agent
 ├── Cloud Agent
 └── Security Agent
```

Specialists return structured evidence/hypotheses, not merely prose.

External organizational agents may communicate via A2A; internal agents can use queues/events/in-process messaging.

---

## 15. HITL and operational authority

Examples:

```text
Read telemetry                 → autonomous
Restart stateless dev service  → policy-dependent
Restart production workload    → risk-dependent
Scale production               → threshold/policy
Change firewall/IAM            → strong approval
Delete resource/data           → deny or exceptional approval
Production rollback            → policy/incident authority
```

Risk is based on target, environment, blast radius, action, and incident severity—not simply on which model proposed it.

---

## 16. Policies

```text
POL-OPS-001:
  Read capabilities are distinct from mutation capabilities.

POL-OPS-002:
  Production changes require configured authority.

POL-OPS-003:
  Every remediation has verification criteria.

POL-OPS-004:
  High-risk action requires rollback strategy when possible.

POL-OPS-005:
  Agent cannot expand its own cloud permissions.

POL-OPS-006:
  Repeated failed remediation triggers escalation/circuit breaker.
```

---

## 17. Failure scenario

**Scenario:** an agent issues a production scale operation; the cloud API times out after possibly applying it.

Required behavior:

1. do not immediately repeat;
2. read actual resource state;
3. reconcile desired vs actual state;
4. record whether side effect committed;
5. continue verification if successful;
6. retry only if reconciliation proves it did not apply.

---

## 18. Circuit breakers

Operational autonomy requires explicit limits.

```text
maxRemediationAttempts
maxActionsPerIncident
maxAffectedResources
maxCostIncrease
maxBlastRadius
maxAutonomousRisk
```

Exceeded limits route to human authority.

---

## 19. Durable execution

Incident workflows persist through:

- waits for propagation;
- CI jobs;
- deployment;
- human approval;
- external remediation;
- service recovery windows.

Checkpoint after every consequential action.

---

## 20. Events

```text
SignalReceived
IncidentCreated
IncidentCorrelated
HypothesisCreated
HypothesisRejected
RootCauseIdentified
RemediationProposed
ApprovalRequested
RemediationExecuted
RecoveryVerified
IncidentResolved
PostmortemRequested
```

---

## 21. Artifacts

- incident timeline;
- query results;
- logs/metric snapshots;
- topology snapshot;
- remediation plan;
- command/tool results;
- verification report;
- stakeholder updates;
- postmortem.

---

## 22. Audit vs telemetry

Telemetry answers:

> What is the system doing?

Audit answers:

> Who/what authorized and executed this production change, with which evidence and policy?

Both are required.

---

## 23. Observability

Harness-level:

```text
agent_run_latency
tool_latency
tool_error_rate
queue_depth
retry_count
model_cost
checkpoint_resume_count
```

Operational outcome:

```text
MTTR
false_remediation_rate
recovery_success
human_escalation
incident_recurrence
blast_radius
```

---

## 24. Evals

```text
incident_classification
signal_correlation
hypothesis_quality
root_cause_accuracy
tool_selection
runbook_selection
remediation_safety
blast_radius_estimation
recovery_verification
policy_compliance
```

Simulation environments are especially valuable for this harness.

---

## 25. Security

Use:

- workload identity;
- short-lived credentials;
- environment-specific roles;
- sandbox for commands;
- network egress policy;
- command/capability policy;
- protected production resources;
- immutable action audit.

Never expose broad cloud credentials to the model.

---

## 26. Pseudocode

```text
FUNCTION handleOperationalEvent(event):

  normalized = eventAdapter.normalize(event)

  candidate = correlationEngine.correlate(normalized)

  IF candidate.suppressed OR candidate.duplicate:
      RETURN

  incident = incidentRepository.loadOrCreate(candidate)

  WHILE NOT incident.terminal:

      context = opsContext.build(
          incident,
          boundedTimeWindow=True
      )

      decision = incidentAgent.decide(context)

      IF decision.type == INVESTIGATE:
          graph = dependencyPlanner.build(
              decision.readOperations
          )
          evidence = executionFabric.execute(graph)
          incident.apply(evidence)

      IF decision.type == REMEDIATE:
          risk = riskEngine.evaluate(decision.action)

          authority = policyEngine.authorize(
              decision.action, risk
          )

          IF authority.requiresHuman:
              WAIT humanRuntime.request(decision.action)

          result = toolRuntime.execute(decision.action)
          incident.record(result)

          actual = verifier.reconcileActualState(
              decision.action
          )

          incident.apply(actual)

      checkpoint.save(incident)

  RETURN incident.outcome
```

---

## 27. Student exercises

1. Application outage.
2. Kubernetes failure.
3. CI/CD incident.
4. Cloud resource degradation.
5. Deployment rollback.
6. Capacity alert.
7. Certificate expiration.
8. Infrastructure drift.

---

## 28. Required ADRs

- Alert correlation before agent activation.
- Read vs mutation capability boundary.
- Operational risk/authority model.
- Uncertain side-effect reconciliation.
- Multi-agent incident delegation.

---

## 29. Definition of Done

The implementation demonstrates event-driven activation, correlation/deduplication, parallel investigation, explicit hypotheses, risk-aware remediation, HITL, reconciliation, circuit breakers, durable waits, recovery verification, audit, operational metrics, and evals.

---

## 30. Constitution test

Cloud, Kubernetes, CI/CD, ITSM, and observability integrations are adapters/capabilities. None should become dependencies of the Generic Agent Core.
