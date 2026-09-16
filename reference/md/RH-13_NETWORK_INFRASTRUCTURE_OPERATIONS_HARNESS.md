# RH-13 — Network & Infrastructure Operations Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness

## 1. Mission
Operate heterogeneous networks through a governed closed loop while separating observed state, intent, authority, execution, and verified state.

```text
Architecture Constitution
→ Generic Agent Harness
→ Network Operations Domain Pack
→ Network & Infrastructure Operations Harness
→ Specific NOC / Telco / Enterprise Use Case
```

> Intent, observed state, proposed change, authorized change, executed change, and verified state are different objects.

## 2. Environments
ISP/Telco, FTTH, enterprise WAN/LAN, SD-WAN, data centers, Wi-Fi, BNG/AAA/RADIUS, OLT/ONT, routing, switching, firewalls, DNS/DHCP/IPAM and OSS/NMS.

## 3. Activations
SNMP/telemetry, syslog, controller events, OLT/BNG/AAA events, customer alarms, OSS events, tickets, topology changes, capacity thresholds, schedules, webhooks, queues, event buses, APIs and other agents.

```text
Raw Signals
→ Normalize
→ Deduplicate
→ Temporal + Topology + Service Correlation
→ Incident Candidates
→ Admission
→ Agent Activation
```

Raw alarms must not map 1:1 to model calls.

## 4. State contracts
```text
NetworkSignal {
  signalId, source, observedAt, deviceRef?, serviceRef?,
  customerScope?, metricOrEvent, severity?, rawArtifactRef?,
  correlationKeys[]
}

NetworkIncidentState {
  incidentId, signals[], affectedResources[], affectedServices[],
  customerImpact?, topologySnapshotRef, hypotheses[], actions[],
  approvals[], verification[], status
}

NetworkIntent {
  intentId, targetScope, desiredState, constraints[], policyRefs[]
}

NetworkAction {
  actionId, target, operation, preconditions[], expectedState,
  rollbackPlan?, riskLevel, authority, idempotencyKey?
}
```

## 5. Agents
Network Triage Agent; Topology Agent; Diagnostic Agent; Configuration Analysis Agent; Remediation Agent; Verification Agent; Field Coordination Agent; Customer Impact Agent.

## 6. Context engineering
Use bounded time windows and topology neighborhoods. Relevant context can include topology, inventory, telemetry, alarms, logs, configuration, recent changes, maintenance windows, customer/service mappings, capacity, historical incidents and runbooks.

## 7. Skills
`correlate-network-signals`, `map-service-impact`, `analyze-topology`, `diagnose-connectivity`, `compare-config`, `detect-drift`, `analyze-capacity`, `select-runbook`, `plan-remediation`, `verify-service`, `coordinate-field-work`.

## 8. Capabilities
`nms.query`, `telemetry.query`, `topology.read`, `inventory.read`, `config.read/change`, `radius.query`, `aaa.change`, `controller.read/change`, `olt.read/change`, `router.read/change`, `firewall.read/change`, `dns.read/change`, `dhcp.read/change`, `ipam.read/change`, `ticket.create/update`.

Vendor-specific SDKs, protocols and CLI syntax live behind adapters.

## 9. Closed loop
```text
Observe
  ↓
Correlate
  ↓
Orient / Context
  ↓
Diagnose
  ↓
Decide
  ↓
Authorize
  ↓
Act
  ↓
Observe Again
  ↓
Verify
  ├─ recovered → close / learn
  └─ not recovered → replan / escalate
```

The second observation is mandatory.

## 10. Parallelism and dependencies
```text
read topology ───────┐
query telemetry ─────┼─ parallel
read recent changes ─┤
query config ────────┘

apply change → wait convergence → measure → verify
```

The runtime owns scheduling.

## 11. Authority
Read-only diagnosis may be autonomous. Bounded low-risk changes are policy-dependent. Customer-affecting and wide-blast-radius changes require stronger authority. Destructive/irreversible actions are denied or routed through exceptional approval.

Authority belongs in policy/configuration, not prompts.

## 12. Agent communication
```text
Network Coordinator
├─ Access Agent
├─ IP Core Agent
├─ Transport Agent
├─ AAA Agent
├─ Security Agent
└─ Customer Impact Agent
```

Independent agents may use A2A. Deterministic network systems remain tool/API/event integrations.

## 13. Policies
- Read and mutation capabilities are separated.
- Every autonomous change has explicit scope and verification criteria.
- An agent cannot expand its own privileges.
- Wide blast radius requires stronger authority.
- Repeated unsuccessful remediation opens a circuit breaker.
- Actual network state is reconciled after uncertain writes.

## 14. Failure exercise
A controller times out after receiving a configuration change. Do not blindly resend. Read actual state, compare intended vs actual, determine whether the change committed, verify convergence/service, and retry only when safe.

## 15. Circuit breakers
`maxChangesPerIncident`, `maxDevicesAffected`, `maxCustomersAffected`, `maxBlastRadius`, `maxRemediationAttempts`, `maxAutonomousRisk`, `maxEstimatedCostImpact`.

## 16. Durable execution
Persist waits, timers, approvals, topology snapshots, correlation IDs, action results and verification state. Network convergence or field work may span hours/days.

## 17. Events
`NetworkSignalReceived`, `IncidentCorrelated`, `ServiceImpactDetected`, `HypothesisCreated`, `ChangeProposed`, `ChangeApproved`, `ChangeExecuted`, `ConvergenceObserved`, `RecoveryVerified`, `FieldDispatchRequested`, `IncidentResolved`.

## 18. Audit and observability
Audit reconstructs observed state, hypothesis, authority, target/scope, exact change, result and post-change state.

Technical telemetry: run/tool latency, queue depth, retries, cost.  
Network outcomes: availability, latency, loss, utilization, customer impact, MTTR, change success.

## 19. Evals
`alarm_correlation`, `topology_reasoning`, `impact_analysis`, `diagnosis_accuracy`, `configuration_analysis`, `tool_selection`, `change_safety`, `blast_radius_estimation`, `reconciliation`, `recovery_verification`, `policy_compliance`.

Digital twins/simulators are preferred for destructive-path evaluation.

## 20. Pseudocode
```text
FUNCTION processNetworkSignals(signals):
  normalized = normalize(signals)
  candidates = correlationEngine.correlate(normalized)

  FOR candidate IN candidates:
    IF admission.suppress(candidate): CONTINUE
    incident = repository.loadOrCreate(candidate)

    WHILE NOT incident.terminal:
      context = networkContext.build(incident)
      decision = agentRuntime.decide(context)

      IF decision.investigation:
        graph = dependencyPlanner.build(decision.readOperations)
        incident.apply(executionFabric.execute(graph))

      IF decision.change:
        risk = blastRadius.evaluate(decision.change)
        authority = policy.authorize(decision.change, risk)
        IF authority.requiresHuman:
          WAIT humanRuntime.request(decision.change)

        preState = verifier.snapshot(decision.change.target)
        result = toolRuntime.execute(decision.change)
        actualState = verifier.reconcile(decision.change)
        incident.record(preState, result, actualState)

        IF verifier.recovered(actualState):
          incident.resolve()
        ELSE:
          incident.replanOrEscalate()

      checkpoint.save(incident)
```

## 21. Student exercises
FTTH access outage; BNG/AAA degradation; OLT capacity; WAN routing incident; Wi-Fi degradation; configuration drift; DNS/DHCP outage; automated capacity response.

## 22. Definition of Done
Multi-source activation, correlation, topology-aware context, parallel diagnosis, bounded authority, safe changes, reconciliation, closed-loop verification, circuit breakers, durable execution, audit, network outcomes and evals.

## 23. Constitution test
SNMP, RADIUS, NETCONF/RESTCONF, vendor SDKs, controllers and device-specific logic belong behind adapters/capabilities—not inside Generic Core.
