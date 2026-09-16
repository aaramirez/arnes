# RH-11 — Legal, Risk & Compliance Reference Harness
**Version:** 0.1  
**Extends:** Generic Agent Harness  
**Category:** Evidence, Obligation, Control & Governance

---

## 1. Mission

The Legal, Risk & Compliance Harness analyzes documents, obligations, risks, controls, evidence, and regulatory change while preserving provenance, uncertainty, authority, jurisdiction, and human accountability.

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
Legal / Risk / Compliance Domain Pack
        ↓
Reference Harness
```

Central invariant:

> **A model's interpretation is not itself a legal authority, risk acceptance, compliance approval, or regulatory filing.**

---

## 2. Actors

- Legal counsel
- Compliance officer
- Risk owner
- Control owner
- Auditor
- Business owner
- Vendor/third party
- Regulator-facing team
- Security/privacy teams
- Human approver

---

## 3. Business outcomes

```text
review_cycle_time
obligation_coverage
control_evidence_coverage
risk_detection_rate
false_positive_rate
regulatory_change_latency
audit_preparation_time
human_review_effort
```

---

## 4. Activation sources

```text
Contract Upload
Policy Update
Regulatory Update
Control Event
Audit Request
Risk Event
Vendor Event
User Request
Scheduled Review
Another Agent
        ↓
ActivationRequest
```

---

## 5. Admission

Evaluate:

- jurisdiction;
- matter/workspace;
- confidentiality;
- privilege handling;
- user authority;
- permitted external research;
- data residency;
- risk category;
- conflict/access restrictions;
- required human counsel.

---

## 6. Agents

### Legal Research Agent
Finds and structures relevant authority/evidence.

### Contract Analyst
Extracts clauses, obligations, deviations, and risks.

### Regulatory Change Agent
Compares new requirements to current policy/control state.

### Risk Analyst
Creates risk hypotheses and evidence-backed assessments.

### Compliance Evidence Agent
Maps controls to evidence.

### Challenge / Counterargument Agent
Actively searches for alternative interpretation or contradictory evidence.

### Human Review Coordinator
Packages findings for authorized review.

---

## 7. Core state models

```text
MatterState {
  matterId
  jurisdiction
  classification
  documents[]
  issues[]
  obligations[]
  risks[]
  findings[]
  humanDecisions[]
}
```

```text
Obligation {
  obligationId
  authorityRef
  subject
  requirement
  effectiveDate
  jurisdiction
  applicability
  evidenceRefs[]
}
```

```text
Risk {
  riskId
  statement
  likelihood?
  impact?
  evidenceRefs[]
  controlRefs[]
  owner
  status
}
```

```text
Control {
  controlId
  objective
  owner
  frequency
  evidenceRequirements[]
  mappedObligations[]
}
```

---

## 8. Context engineering

Context may include:

- matter documents;
- applicable jurisdiction;
- authoritative sources;
- contracts;
- internal policies;
- control library;
- evidence;
- precedent/memos when authorized;
- known interpretations;
- counter-evidence.

Every material source should preserve provenance/version.

---

## 9. Knowledge hierarchy

Do not flatten all sources into equivalent chunks.

```text
Authority / Regulation
        ↓
Obligation
        ↓
Internal Policy
        ↓
Control
        ↓
Evidence
        ↓
Finding
```

A generated summary is not an authority.

---

## 10. Skills

```text
review-contract
extract-clause
map-obligation
research-authority
assess-risk
identify-control-gap
collect-evidence
compare-regulatory-change
prepare-review-memo
challenge-conclusion
```

---

## 11. Capabilities

```text
document.read
legal_research.search
regulatory_source.fetch
grc.read/update
evidence_store.read/write
contract_repository.read
policy_repository.read
artifact.generate
notification.send
```

Writes to GRC or filing systems are higher-risk than read/research operations.

---

## 12. Contract review workflow

```text
Contract Received
      ↓
Classify / Determine Jurisdiction
      ↓
Extract Clauses
      ↓
Compare to Playbook
      ↓
┌──────────────┬─────────────────┐
│ Risk Analysis│ Obligation Map  │
└───────┬──────┴────────┬────────┘
        └────────┬───────┘
                 ↓
          Deviations / Findings
                 ↓
          Human Legal Review
                 ↓
        Approved Position / Action
```

---

## 13. Regulatory change workflow

```text
Regulatory Event
      ↓
Source Verification
      ↓
Extract Requirements
      ↓
Applicability Analysis
      ↓
Map to Existing Policies / Controls
      ↓
Gap Analysis
      ↓
Human Review
      ↓
Remediation Workflow
      ↓
Evidence of Completion
```

---

## 14. HITL and authority

Required human authority for configured actions such as:

- legal advice/sign-off;
- risk acceptance;
- control exception;
- contractual commitment;
- regulatory filing;
- material compliance interpretation;
- waiver;
- privileged matter decisions.

```text
DecisionAuthority {
  decisionType
  authorizedRoles[]
  threshold?
  jurisdiction?
  requiredApprovals
}
```

---

## 15. Policies

```text
POL-LRC-001:
  Material findings preserve source/evidence references.

POL-LRC-002:
  Jurisdiction must be resolved before applying jurisdiction-specific rules.

POL-LRC-003:
  Privileged/restricted matter content cannot be delegated externally without authorization.

POL-LRC-004:
  Risk acceptance requires authorized human authority.

POL-LRC-005:
  Regulatory filing cannot be inferred from model completion.

POL-LRC-006:
  Contradictory authority/evidence is surfaced.
```

---

## 16. Agent communication

Specialized external legal/regulatory agents may use A2A, but their outputs enter as untrusted structured findings until validated locally.

```text
Local Matter Agent
      ↓
Bounded Delegation
      ↓
Specialist Agent
      ↓
FindingPackage
      ↓
Local Provenance / Policy Validation
```

---

## 17. Evidence model

```text
EvidenceItem {
  evidenceId
  sourceRef
  sourceVersion
  proposition
  locationRef
  collectedAt
  collector
  classification
  integrityHash?
}
```

Evidence lineage should survive transformations.

---

## 18. Failure scenario

**Scenario:** a regulatory source changes after an analysis has been completed but before a remediation plan is approved.

The harness must:

1. detect source/version change where configured;
2. mark dependent findings potentially stale;
3. identify affected obligations/controls;
4. re-run only impacted analysis;
5. require renewed review when material;
6. preserve the old analysis for audit.

---

## 19. Durable execution

Matters and remediation programs may run for months.

Persist:

- matter state;
- source versions;
- obligations;
- risk/control mappings;
- findings;
- human decisions;
- remediation tasks;
- deadlines;
- evidence.

---

## 20. Events

```text
MatterOpened
ContractReceived
RegulatoryChangeDetected
ObligationIdentified
RiskFindingRaised
ControlGapIdentified
HumanReviewRequested
RiskAccepted
RemediationCreated
EvidenceCollected
MatterClosed
```

---

## 21. Audit

Audit must distinguish:

```text
Source fact
Extracted evidence
Agent interpretation
Human decision
Executed remediation
```

This separation is essential for defensibility.

---

## 22. Observability

Technical:
- research latency;
- document processing;
- agent/tool failures;
- cost;
- human wait.

Domain:
- findings by severity;
- evidence coverage;
- open obligations;
- remediation aging;
- stale analyses;
- review backlog.

---

## 23. Evals

```text
clause_extraction_accuracy
obligation_mapping
jurisdiction_selection
source_provenance
risk_issue_detection
false_positive_rate
control_mapping
contradiction_detection
stale_source_detection
human_escalation_accuracy
```

---

## 24. Security / governance

This harness may require:

- matter-level ACL;
- legal privilege labels;
- restricted external model use;
- data residency;
- immutable evidence;
- retention/legal hold;
- ethical walls;
- detailed access audit.

---

## 25. Pseudocode

```text
FUNCTION analyzeMatter(activation):

  matter = matterRepository.loadOrCreate(activation)

  sources = contextEngine.resolveAuthoritativeSources(matter)

  analyses = executionFabric.executeParallel([
      extractObligations(sources),
      identifyRisks(matter),
      mapControls(matter)
  ])

  findings = findingEngine.consolidate(analyses)

  challenged = challengeAgent.test(findings)

  validation = provenanceValidator.validate(
      challenged,
      sources
  )

  IF validation.materialIssues:
      humanTask = humanRuntime.create(validation)
      checkpoint.save(matter.waitFor(humanTask))
      WAIT

  matter.apply(validation)
  audit.record(matter)

  RETURN matter.reviewPackage
```

---

## 26. Student exercises

1. Contract review.
2. Regulatory change analysis.
3. Compliance evidence collection.
4. Policy gap analysis.
5. Third-party risk.
6. Control testing.
7. Legal research.
8. Obligation mapping.

---

## 27. Required ADRs

- Authority/source hierarchy.
- Privileged/restricted context handling.
- Human legal/risk authority boundary.
- Evidence lineage.
- Regulatory source versioning.

---

## 28. Definition of Done

The implementation demonstrates evidence lineage, source/version awareness, jurisdiction-aware context, explicit human authority, challenge/counter-evidence, durable matter state, policy enforcement, audit separation, and domain evals.

---

## 29. Constitution test

The Generic Core must know nothing about contracts, regulations, legal privilege, controls, or risk acceptance. Those are Domain Pack concerns.
