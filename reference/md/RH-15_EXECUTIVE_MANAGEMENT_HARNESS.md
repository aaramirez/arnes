# RH-15 — Executive & Management Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness

## 1. Mission
Consolidate enterprise evidence, detect material changes, frame decisions, challenge assumptions, coordinate follow-through and measure outcomes without replacing accountable leadership.

> **The harness improves the decision system; it does not become the accountable executive.**

```text
Architecture Constitution
→ Generic Agent Harness
→ Executive & Management Domain Pack
→ Executive & Management Reference Harness
→ Specific Management Operating Model
```

## 2. Activations
Scheduled operating review, KPI threshold, forecast change, risk event, project milestone, budget variance, customer/market signal, executive request, decision deadline or output from another harness.

## 3. Materiality
```text
Signals
 ↓
Validation / Reconciliation
 ↓
Materiality Filter
 ↓
Management Issue
 ↓
Executive Workflow
```

Materiality can consider financial/customer impact, strategic priority, risk, variance, confidence, urgency and reversibility.

## 4. Agents
Management Intelligence Agent; Performance Agent; Strategy Agent; Finance Agent; Risk/Challenge Agent; Decision Preparation Agent; Follow-Through Agent.

## 5. State
```text
ManagementCycleState {
  cycleId, period, priorities[], metrics[], issues[],
  risks[], decisions[], actions[], outcomes[]
}

DecisionRecord {
  decisionId, question, owner, alternatives[], evidenceRefs[],
  assumptions[], risks[], recommendation?, humanDecision,
  rationale?, decidedAt, reviewDate?
}

ActionCommitment {
  actionId, decisionRef, owner, dueDate,
  successCriteria[], status, evidenceRefs[]
}
```

## 6. Context engineering
Use hierarchical, drillable context rather than one enormous enterprise prompt:

```text
Enterprise Summary
  ↓
Issue Summary
  ↓
Evidence
  ↓
Source System / Artifact
```

Include strategic priorities, KPI/OKR deltas, variances, project changes, customer/market signals, risks, previous decisions and open commitments.

## 7. Evidence hierarchy
```text
Source Data
 ↓
Validated Metric / Fact
 ↓
Finding
 ↓
Interpretation
 ↓
Alternative Explanations
 ↓
Recommendation
 ↓
Human Decision
```

These layers remain distinguishable.

## 8. Skills
`prepare-operating-review`, `analyze-kpi-variance`, `summarize-portfolio`, `frame-decision`, `generate-alternatives`, `challenge-assumptions`, `analyze-scenario`, `prepare-executive-brief`, `track-commitments`, `review-decision-outcome`.

## 9. Capabilities
Primarily read-oriented: analytics, finance, CRM, portfolio, risk, operations, research and documents. Bounded writes can create/update tasks, calendar items and notifications. Consequential business writes remain explicit.

## 10. Operating review workflow
```text
Scheduled Review
 ↓
Collect Enterprise Signals
 ↓
Validate / Reconcile
 ↓
Materiality Filter
 ↓
┌─────────────┬─────────────┬──────────────┐
│ Performance │ Finance     │ Risk/Challenge│
└──────┬──────┴──────┬──────┴──────┬───────┘
       └─────────────┼──────────────┘
                     ↓
               Issue Synthesis
                     ↓
             Decision Required?
              ├─ no → Brief
              └─ yes
                     ↓
              Decision Package
                     ↓
                Human Decision
                     ↓
              Action Commitments
                     ↓
                Follow-Through
                     ↓
                 Outcome Review
```

## 11. Decision package
```text
DecisionPackage {
  question, whyNow, decisionOwner, evidence[], uncertainties[],
  assumptions[], alternatives[], optionTradeoffs[], risks[],
  reversibility, recommendation?, dissentingView?, requiredBy
}
```

Uncertainty is visible rather than replaced by synthetic confidence.

## 12. Challenge function
A structurally separate challenge role asks:
- Which assumption can fail?
- What evidence contradicts this?
- What are second-order effects?
- Is correlation being treated as causation?
- What would change the decision?
- What is the cost of waiting?

## 13. Scenarios
Use Base, Upside, Downside and Stress cases with explicit assumptions/evidence. Scenario output is not prediction certainty.

## 14. Human authority
Executives retain authority for strategic decisions, material resource allocation, organizational changes, risk acceptance, major commitments, policy changes and other high-impact decisions.

## 15. Cross-harness integration
```text
           RH-15 Executive Harness
             ↑   ↑   ↑   ↑
             │   │   │   │
          Finance Sales Ops Risk Projects
```

Prefer versioned events/artifacts/contracts over access to hidden agent state.

```text
ManagementSignal {
  signalId, sourceHarness, signalType, subject,
  materiality, confidence, evidenceRefs[],
  occurredAt, expiresAt?
}
```

If broadly useful, `ManagementSignal` becomes a candidate Generic Enterprise primitive.

## 16. A2A
Use A2A when independent agents must delegate/negotiate. For most cross-harness reporting, event/artifact contracts are simpler and more governable.

## 17. Policies
- Recommendations identify evidence and assumptions.
- Material decisions have explicit human owner.
- Recommendation never silently becomes commitment.
- Conflicting material evidence is surfaced.
- Cross-domain context respects source-domain access policy.
- Commitments have owner, due date and success criteria.

## 18. Failure exercise
A brief reports a major revenue decline, but two systems disagree because one daily load is incomplete. Detect freshness/reconciliation conflict, mark the metric unresolved, prevent unsupported causal explanation, delay/escalate material decisions and resume when authoritative data arrives.

## 19. Durable decision cycle
```text
Signal → Issue → Decision → Commitment → Execution → Outcome → Learning
```

Preserve this chain over weeks/months.

## 20. Events
`MaterialSignalDetected`, `ManagementIssueCreated`, `DecisionRequired`, `DecisionPackagePrepared`, `DecisionRecorded`, `ActionCommitted`, `ActionOverdue`, `OutcomeObserved`, `DecisionReviewDue`, `ManagementCycleClosed`.

## 21. Decision memory
Record what was known at the time, assumptions, alternatives, decision owner, rationale, expected outcome and actual outcome. This enables institutional learning without hindsight distortion.

## 22. Observability
Technical: data/agent latency, cost, source failures, human wait.  
Management: decision latency, action completion, overdue commitments, forecast accuracy, review rate, outcome realization.

## 23. Evals
`metric_fidelity`, `materiality_detection`, `evidence_grounding`, `variance_explanation`, `alternative_generation`, `assumption_identification`, `challenge_quality`, `decision_package_completeness`, `commitment_tracking`, `outcome_attribution`.

## 24. Pseudocode
```text
FUNCTION runManagementCycle(trigger):
  signals = signalGateway.collect(trigger)
  validated = metricValidator.reconcile(signals)
  issues = materialityEngine.detect(validated)

  FOR issue IN issues:
    perspectives = executionFabric.executeParallel([
      performanceAgent.analyze(issue),
      financeAgent.analyze(issue),
      riskAgent.challenge(issue),
      strategyAgent.frame(issue)
    ])

    package = decisionAgent.synthesize(issue, perspectives)

    IF package.requiresDecision:
      decision = WAIT humanRuntime.request(package)
      decisionRepository.record(decision)
      commitments = actionPlanner.create(decision)
      workflow.start(commitments)
    ELSE:
      artifactStore.save(package)

  checkpoint.save()
  RETURN managementCycle.summary
```

## 25. Student exercises
Weekly operating review; portfolio review; budget variance; strategy decision; product investment; risk committee; sales performance; executive KPI cockpit.

## 26. Required ADRs
Materiality filtering; cross-harness integration; recommendation vs decision authority; decision memory; challenge-agent independence; metric reconciliation.

## 27. Definition of Done
Multi-domain signal ingestion, materiality, evidence hierarchy, parallel perspectives, challenge, explicit decision ownership, durable decisions/actions/outcomes, cross-harness contracts, audit, management outcomes and evals.

## 28. Constitution test
RH-15 must not become an unrestricted super-agent. It is constrained by the same identity, policy, delegation, context, audit and human-authority rules as every harness.

## 29. Reference Harness Suite Completion
The 15 Reference Harnesses now serve both as teaching examples and as an **Architecture Conformance Suite**:

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
15 Reference Harnesses
        ↓
Conformance Findings
        ↓
Generic Primitive Refinement
        ↓
ADRs / Constitution Updates
```

If a reference domain requires modifying Generic Core with domain-specific behavior, investigate whether a generic primitive is missing or the boundary is wrong.
