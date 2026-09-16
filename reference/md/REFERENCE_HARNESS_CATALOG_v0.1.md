# Reference Harness Catalog
## 15 arquitecturas de referencia para enseñanza, práctica y validación del Generic Agent Harness
**Versión:** 0.1  
**Rol:** Catálogo académico y arquitectónico

---

# 1. Propósito

Este catálogo define **15 Reference Harnesses canónicos**. Un Reference Harness no es un caso de uso individual: es una especialización reusable del Generic Agent Harness para una familia de problemas.

```text
Generic Harness
      +
Domain Pack
      =
Reference Harness
      +
Specific Use Case
      =
Student / Enterprise Implementation
```

Ejemplo:

```text
Generic Harness
      +
Finance Domain Pack
      =
Finance & Accounting Harness
      +
Invoice Processing
      =
Invoice Processing Implementation
```

La intención es permitir que muchos estudiantes trabajen sobre problemas distintos sin fragmentar la arquitectura común.

---

# 2. Contrato común de todos los Reference Harnesses

Todo Reference Harness debe especificar:

1. Problem Domain
2. Actors
3. Business Outcomes
4. Activation Sources
5. Admission Rules
6. Agents and Responsibilities
7. State
8. Context Sources
9. Memory Strategy
10. Skills
11. Tools / Capabilities
12. External Integrations
13. Workflows
14. Parallelism and Dependencies
15. Agent Communication / Delegation
16. Protocol / Transport Requirements
17. Policies
18. Identity / Authorization
19. Human-in-the-Loop
20. Failure Semantics
21. Retry / Idempotency
22. Durable Execution
23. Events
24. Artifacts
25. Audit
26. Observability
27. Budgets / FinOps
28. Business Outcomes
29. Evals
30. Security / Data Governance
31. Architecture Diagram
32. Sequence Diagrams
33. Contracts / Interfaces
34. Pseudocode
35. Constitution Compliance
36. ADRs
37. Test Scenarios
38. Definition of Done

Todos deben respetar la Architecture Constitution del Generic Harness.

---

# 3. Anatomía de un Domain Pack

```text
DomainPack
│
├── Agents
├── Skills
├── Capabilities
├── State Types
├── Context Providers
├── Memory Policies
├── Activation Rules
├── Workflows
├── Policies
├── Validators
├── Evals
├── Business Outcomes
└── Optional UI / Interaction Adapters
```

El Domain Pack no debe duplicar primitives del Generic Harness.

---

# RH-01 — Book & Content Production Harness

## Propósito
Crear, revisar, validar, versionar y publicar contenido estructurado de larga duración.

## Actores
Autores, editores, revisores técnicos, diseñadores, profesores, publishers.

## Activaciones
Usuario, commit, modificación de capítulo, schedule, review request, build event.

## Agentes
- Book/Content Architect
- Author
- Technical Reviewer
- Pedagogical Reviewer
- Consistency Reviewer
- Editor

## State
`ContentProjectState`, `ChapterState`, `ContentManifest`, `ContractRegistry`, `Glossary`.

## Context
Contenido previo, estructura, guías editoriales, fuentes, ADRs, terminología.

## Skills
`write-section`, `define-concept`, `review-technical-content`, `review-pedagogy`, `revise`, `publish`.

## Capabilities
Files, search, diagrams, document generation, Web renderer, PDF renderer.

## Workflows
Plan → Draft → Review → Revise → Validate → Publish.

## HITL
Cambios de estructura, arquitectura, claims sensibles, publicación.

## Evals
Exactitud, consistencia, claridad, progresión pedagógica, referencias.

## Business outcomes
Tiempo de producción, defectos editoriales, retrabajo, calidad, publicación.

## Ejercicios
1. Libro técnico.
2. Manual de operaciones.
3. Curso universitario.
4. Knowledge playbook.
5. Documentación de API.
6. Manual de onboarding.
7. Reporte anual.
8. Policy handbook.

---

# RH-02 — Software Development Harness

## Propósito
Construir, modificar, depurar, probar y revisar software.

## Activaciones
Usuario, issue, pull request, CI failure, webhook, queue, scheduled maintenance.

## Agentes
Developer, Planner, Test Reviewer, Code Reviewer, opcional Architecture Agent.

## State
`ProjectState`, `RepositoryState`, `DevelopmentTask`, `DevelopmentPlan`, `BuildState`, `TestState`.

## Context
Repository map, código relevante, símbolos, dependencias, tests, conventions, architecture rules.

## Skills
`inspect-repository`, `implement-feature`, `debug`, `write-tests`, `refactor`, `review-code`.

## Capabilities
Filesystem, shell, Git, GitHub, build tools, test runners, package managers.

## Workflows
Inspect → Plan → Edit → Test → Fix → Build → Diff → Review.

## HITL
Push, major refactor, schema migration, deployment, breaking change.

## Evals
Task completion, test success, regression rate, diff quality, security, cost.

## Ejercicios
1. Feature implementation.
2. Bug fixing.
3. CI failure diagnosis.
4. Pull-request review.
5. Dependency upgrade.
6. API migration.
7. Frontend refactor.
8. Database migration.

---

# RH-03 — Research & Intelligence Harness

## Propósito
Investigar preguntas complejas y producir conclusiones trazables a evidencia.

## Activaciones
Usuario, API, scheduled intelligence request, event indicating new evidence.

## Agentes
Research Planner, Source Researcher, Evidence Analyst, Contradiction Reviewer, Synthesizer.

## State
`ResearchState`, `ResearchQuestion`, `Source`, `Evidence`, `Claim`, `Contradiction`, `Citation`.

## Context
Fuentes internas/externas, evidence store, research history, domain rules.

## Skills
`formulate-question`, `discover-sources`, `evaluate-source`, `extract-evidence`, `compare-claims`, `synthesize`.

## Capabilities
Web/search, enterprise knowledge, document readers, citation manager.

## Workflows
Question → Plan → Search → Evidence → Claims → Contradictions → Synthesis → Citation validation.

## HITL
Scope changes, ambiguous evidence, consequential recommendations.

## Evals
Source quality, citation coverage, claim support, contradiction detection, freshness.

## Ejercicios
1. Market research.
2. Competitive intelligence.
3. Legal research.
4. Technology landscape.
5. Policy research.
6. Vendor intelligence.
7. Academic literature review.
8. Due-diligence research.

---

# RH-04 — Data & Analytics Harness

## Propósito
Convertir preguntas de negocio en análisis reproducibles y validados.

## Activaciones
Usuario, API, schedule, dataset arrival, KPI anomaly.

## Agentes
Data Analyst, Data Engineer, Statistical Reviewer, Business Interpreter.

## State
`AnalysisState`, `DatasetState`, `AnalysisPlan`, `Transformation`, `Metric`, `ModelResult`.

## Context
Schemas, catalog, lineage, metric definitions, previous analyses.

## Skills
`discover-data`, `write-query`, `clean-data`, `analyze`, `validate-statistics`, `visualize`, `interpret`.

## Capabilities
SQL, Python sandbox, warehouse, files, visualization, notebook/artifact generation.

## Workflows
Question → Discover → Plan → Query → Transform → Analyze → Validate → Visualize → Interpret.

## HITL
Sensitive datasets, production writes, material business decisions.

## Evals
Correctness, reproducibility, leakage, statistical validity, query efficiency.

## Ejercicios
1. Sales dashboard.
2. Churn analysis.
3. Cohort analysis.
4. Demand forecasting.
5. Pricing analysis.
6. Marketing attribution.
7. Operational KPI analysis.
8. Anomaly investigation.

---

# RH-05 — Customer Service Harness

## Propósito
Resolver casos de clientes de manera omnicanal, contextual y gobernada.

## Activaciones
Chat, voice, email, WhatsApp, ticket, API, customer event.

## Agentes
Support Agent, Diagnostic Agent, Knowledge Agent, Escalation/Handoff Agent.

## State
`CustomerCaseState`, `ConversationState`, `ResolutionState`, `SLAState`.

## Context
CRM, account, products, orders, tickets, knowledge, prior interactions.

## Skills
`identify-intent`, `diagnose`, `retrieve-policy`, `resolve-case`, `handoff`, `follow-up`.

## Capabilities
CRM, ticketing, knowledge base, billing, messaging, diagnostics.

## Workflows
Identify → Context → Diagnose → Resolve/Act → Validate → Close or Handoff.

## HITL
Exceptions, high-value customers, refunds above threshold, unresolved cases.

## Evals
Resolution accuracy, FCR, escalation quality, hallucination, policy compliance.

## Business outcomes
FCR, AHT, CSAT, cost per resolution, escalation rate.

## Ejercicios
1. Billing support.
2. Technical support.
3. Order support.
4. Returns.
5. Telecom support.
6. SaaS help desk.
7. Travel support.
8. Insurance customer service.

---

# RH-06 — Sales & CRM Harness

## Propósito
Gestionar leads y oportunidades mediante inteligencia, seguimiento y automatización.

## Activaciones
Lead form, CRM event, inactivity event, email response, schedule, sales request.

## Agentes
Lead Qualification Agent, Opportunity Agent, Account Research Agent, Follow-up Agent.

## State
`LeadState`, `OpportunityState`, `AccountState`, `EngagementState`.

## Context
CRM, communications, account intelligence, products, pricing, sales policies.

## Skills
`qualify-lead`, `research-account`, `recommend-next-action`, `draft-outreach`, `update-crm`.

## Capabilities
CRM, email, calendar, research, quoting, analytics.

## HITL
Commercial commitments, discounts, sensitive outreach, contract changes.

## Evals
Qualification accuracy, next-action quality, CRM correctness, conversion impact.

## Business outcomes
Conversion, pipeline velocity, revenue, seller productivity.

## Ejercicios
1. Lead qualification.
2. Dormant opportunity recovery.
3. Account planning.
4. Renewal assistance.
5. Sales outreach.
6. Meeting preparation.
7. Cross-sell recommendation.
8. Pipeline hygiene.

---

# RH-07 — Marketing & Creative Harness

## Propósito
Planificar, producir, revisar y medir campañas y activos creativos.

## Activaciones
Campaign brief, schedule, performance event, content request.

## Agentes
Strategist, Research Agent, Copy Agent, Creative Agent, Brand Reviewer, Performance Analyst.

## State
`CampaignState`, `ContentState`, `AudienceState`, `PerformanceState`.

## Context
Brand kit, audience, channels, historical campaigns, analytics, products.

## Skills
`build-strategy`, `write-copy`, `create-creative-brief`, `review-brand`, `analyze-performance`.

## Capabilities
Creative tools, analytics, social platforms, CMS, email platforms.

## HITL
Publishing, paid spend, brand-sensitive content, legal claims.

## Evals
Brand compliance, factual accuracy, engagement quality, conversion contribution.

## Ejercicios
1. Social campaign.
2. Product launch.
3. Email campaign.
4. SEO content.
5. Creative testing.
6. Brand review.
7. Campaign optimization.
8. Editorial calendar.

---

# RH-08 — Document & Knowledge Harness

## Propósito
Ingerir, entender, clasificar, enriquecer y recuperar conocimiento empresarial.

## Activaciones
File upload, repository event, email attachment, schedule, knowledge request.

## Agentes
Document Analyst, Classifier, Knowledge Curator, Retrieval Agent.

## State
`DocumentState`, `KnowledgeItem`, `IndexState`, `ProvenanceState`.

## Context
Documents, metadata, taxonomy, access controls, existing knowledge.

## Skills
`extract`, `classify`, `summarize`, `link-knowledge`, `index`, `retrieve`.

## Capabilities
Document parsers, storage, search/index, OCR where necessary, metadata systems.

## HITL
Low-confidence extraction, restricted classification, records disposition.

## Evals
Extraction accuracy, retrieval precision/recall, provenance, classification accuracy.

## Ejercicios
1. Contract ingestion.
2. Policy knowledge base.
3. Technical documentation index.
4. Email attachment processing.
5. Enterprise search.
6. Records classification.
7. Knowledge consolidation.
8. FAQ generation.

---

# RH-09 — Finance & Accounting Harness

## Propósito
Automatizar y asistir procesos financieros bajo controles estrictos.

## Activaciones
Invoice event, ERP event, file, API, payment exception, schedule.

## Agentes
Finance Operations Agent, Reconciliation Agent, Exception Analyst, Approval Coordinator.

## State
`FinancialCaseState`, `InvoiceState`, `ReconciliationState`, `ApprovalState`.

## Context
ERP, ledger, invoices, vendors, policies, authority matrices.

## Skills
`validate-invoice`, `reconcile`, `analyze-exception`, `prepare-approval`, `explain-variance`.

## Capabilities
ERP, accounting systems, document extraction, payments under policy.

## HITL
Threshold approvals, payment, journal adjustments, exceptions.

## Policies
Segregation of duties, delegated authority, dual approval, immutable audit.

## Evals
Financial correctness, duplicate prevention, policy compliance, exception accuracy.

## Ejercicios
1. Invoice processing.
2. Expense approval.
3. Reconciliation.
4. Collections prioritization.
5. Variance analysis.
6. Payment exception.
7. Month-end assistance.
8. Budget monitoring.

---

# RH-10 — People & HR Harness

## Propósito
Orquestar procesos de personas manteniendo privacidad, equidad y control humano.

## Activaciones
HRIS event, employee request, onboarding event, application event.

## Agentes
Employee Service Agent, Onboarding Agent, Recruiting Operations Agent, HR Knowledge Agent.

## State
`EmployeeCaseState`, `OnboardingState`, `CandidateProcessState`.

## Context
HRIS, policies, benefits, role information, organizational data.

## Skills
`answer-policy`, `coordinate-onboarding`, `route-request`, `prepare-interview`, `track-process`.

## Capabilities
HRIS, calendar, email, document systems, ticketing.

## HITL
Hiring decisions, performance actions, compensation, sensitive employee matters.

## Evals
Policy accuracy, privacy compliance, routing, process completion.

## Ejercicios
1. Employee help desk.
2. Onboarding.
3. Offboarding workflow.
4. Interview coordination.
5. Benefits Q&A.
6. Training assignment.
7. HR case routing.
8. Recruiting operations.

---

# RH-11 — Legal, Risk & Compliance Harness

## Propósito
Analizar obligaciones, riesgos, controles y documentos con evidencia y supervisión.

## Activaciones
Document upload, regulatory update, control event, risk event, user request.

## Agentes
Legal Research Agent, Contract Analyst, Risk Analyst, Compliance Evidence Agent.

## State
`MatterState`, `RiskState`, `ControlState`, `EvidenceState`.

## Context
Contracts, regulations, policies, controls, precedents, evidence.

## Skills
`review-contract`, `map-obligation`, `assess-risk`, `collect-evidence`, `identify-gap`.

## Capabilities
Document systems, regulatory research, GRC platforms, evidence stores.

## HITL
Legal advice, material risk acceptance, regulatory submissions, contractual commitments.

## Evals
Evidence coverage, issue detection, false positives, policy alignment, provenance.

## Ejercicios
1. Contract review.
2. Compliance evidence collection.
3. Policy gap analysis.
4. Regulatory change analysis.
5. Third-party risk.
6. Control testing.
7. Legal research.
8. Obligation mapping.

---

# RH-12 — IT & Cloud Operations Harness

## Propósito
Diagnosticar y remediar incidentes de aplicaciones, infraestructura y cloud.

## Activaciones
Monitoring alert, ticket, CI/CD event, cloud event, schedule.

## Agentes
Incident Agent, Diagnostic Agent, Cloud Operations Agent, Change Reviewer.

## State
`IncidentState`, `SystemState`, `RemediationState`, `ChangeState`.

## Context
Logs, metrics, traces, topology, runbooks, deployment history, CMDB.

## Skills
`triage`, `diagnose`, `correlate-signals`, `execute-runbook`, `verify-recovery`, `rollback`.

## Capabilities
Cloud APIs, Kubernetes, observability, CI/CD, ticketing, shell under sandbox.

## HITL
Production changes, destructive actions, security-sensitive remediation.

## Evals
MTTR, diagnosis accuracy, remediation success, rollback success, incident recurrence.

## Ejercicios
1. Application incident.
2. Cloud resource failure.
3. Kubernetes troubleshooting.
4. CI/CD incident.
5. Release rollback.
6. Capacity alert.
7. Certificate expiry.
8. Infrastructure drift.

---

# RH-13 — Network & Infrastructure Operations Harness

## Propósito
Operar redes y telecomunicaciones mediante telemetría, eventos y acciones gobernadas.

## Activaciones
Telemetry, SNMP/event stream, alarms, NMS, customer degradation, schedule.

## Agentes
NOC Agent, Topology Agent, Diagnostic Agent, Capacity Agent, Remediation Agent.

## State
`NetworkIncidentState`, `TopologyState`, `DeviceState`, `RemediationState`.

## Context
Topology, telemetry, alarms, configuration, inventory, historical incidents.

## Skills
`correlate-alarms`, `diagnose-network`, `analyze-topology`, `recommend-remediation`, `verify-service`.

## Capabilities
NMS, OSS/BSS, routers, OLT/BNG, telemetry, ticketing, configuration systems.

## Agent communication
Especialmente apropiado para delegación, ejecución paralela y A2A/federación.

## HITL
Configuration changes, service-impacting remediation, mass actions.

## Evals
Detection accuracy, root-cause accuracy, MTTR, false remediation, service recovery.

## Ejercicios
1. Fiber outage.
2. BNG degradation.
3. Capacity saturation.
4. OLT alarm correlation.
5. Customer-impact analysis.
6. Configuration drift.
7. Preventive maintenance.
8. Automated incident triage.

---

# RH-14 — Business Process & Workflow Harness

## Propósito
Combinar workflow determinístico, razonamiento agentic y decisiones humanas en procesos empresariales.

## Activaciones
API, event, form, queue, schedule, another workflow.

## Agentes
Process Agent, Exception Agent, Research Agent, Approval Coordinator.

## State
`ProcessInstance`, `StepState`, `ApprovalState`, `ExceptionState`.

## Context
Process data, policies, enterprise systems, case history.

## Skills
`evaluate-case`, `handle-exception`, `collect-information`, `recommend-decision`.

## Capabilities
ERP, CRM, BPM, email, documents, ticketing, external APIs.

## Workflows
Este harness debe demostrar explícitamente:

```text
Deterministic Step
      ↓
Agentic Step
      ↓
Deterministic Validation
      ↓
Human Decision
      ↓
Deterministic Side Effect
```

## HITL
Configurado por proceso, riesgo y autoridad.

## Evals
Cycle time, automation rate, exception resolution, policy compliance.

## Ejercicios
1. Procurement.
2. Vendor onboarding.
3. Order management.
4. Supply-chain exception.
5. Field-service dispatch.
6. Approval workflow.
7. Customer onboarding.
8. Claims workflow.

---

# RH-15 — Executive & Management Harness

## Propósito
Consolidar información, monitorear objetivos y apoyar decisiones gerenciales sin sustituir autoridad humana.

## Activaciones
Schedule, KPI event, meeting, executive request, project event.

## Agentes
Executive Briefing Agent, Portfolio Analyst, Project Intelligence Agent, Decision Support Agent.

## State
`ManagementContext`, `PortfolioState`, `DecisionState`, `ActionState`.

## Context
KPIs, projects, financials, risks, meetings, communications, strategic plans.

## Skills
`synthesize-brief`, `identify-risk`, `compare-options`, `track-actions`, `prepare-decision`.

## Capabilities
BI, project systems, calendar, documents, communications, finance data.

## HITL
Decisiones estratégicas, commitments, personnel actions, material financial decisions.

## Evals
Signal quality, decision relevance, factual consistency, action follow-through.

## Business outcomes
Decision latency, action completion, risk detection, management time saved.

## Ejercicios
1. Daily executive brief.
2. Project portfolio review.
3. Board preparation.
4. Strategic initiative tracking.
5. Meeting intelligence.
6. Risk briefing.
7. KPI anomaly briefing.
8. Decision memo generation.

---

# 4. Matriz de cobertura arquitectónica

Leyenda: `●` central, `○` relevante, `–` secundario.

| Harness | Events | HITL | Durable | A2A | Data Gov | Audit | Artifacts | Business Outcomes |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Book & Content | ○ | ● | ○ | ○ | ○ | ○ | ● | ○ |
| Software Development | ● | ● | ○ | ○ | ○ | ● | ● | ○ |
| Research & Intelligence | ○ | ○ | ○ | ○ | ● | ○ | ● | ○ |
| Data & Analytics | ● | ○ | ○ | ○ | ● | ● | ● | ● |
| Customer Service | ● | ● | ● | ○ | ● | ● | ○ | ● |
| Sales & CRM | ● | ● | ● | ○ | ● | ● | ○ | ● |
| Marketing & Creative | ○ | ● | ○ | ○ | ○ | ○ | ● | ● |
| Document & Knowledge | ● | ● | ● | ○ | ● | ● | ● | ○ |
| Finance & Accounting | ● | ● | ● | ○ | ● | ● | ● | ● |
| People & HR | ● | ● | ● | ○ | ● | ● | ○ | ● |
| Legal/Risk/Compliance | ● | ● | ● | ○ | ● | ● | ● | ● |
| IT & Cloud Operations | ● | ● | ● | ● | ○ | ● | ○ | ● |
| Network Operations | ● | ● | ● | ● | ● | ● | ○ | ● |
| Business Workflow | ● | ● | ● | ● | ● | ● | ○ | ● |
| Executive & Management | ● | ● | ● | ○ | ● | ● | ● | ● |

---

# 5. Niveles académicos

## Level 1 — Foundation
El estudiante implementa un Agent Loop, tools, context, state y validación básica.

Casos recomendados:
- Employee Help Desk
- Lead Qualification
- Document Classification
- Executive Brief
- Simple Research

## Level 2 — Operational
Agregar activation, multiple tools, policies, artifacts y observability.

Casos:
- Book Production
- Coding
- Data Analytics
- Customer Support
- Marketing
- Knowledge Management

## Level 3 — Enterprise
Agregar queues/events, durable execution, HITL, identity, audit e integraciones.

Casos:
- Finance
- HR
- Compliance
- Procurement
- IT Operations

## Level 4 — Autonomous Enterprise
Agregar agent communication, delegation, parallelism, A2A/federation, reliability y business outcomes.

Casos:
- NOC
- Cloud Operations
- Complex Business Workflow
- Cross-functional enterprise process

---

# 6. Plantilla de asignación para estudiantes

Cada estudiante recibe:

```text
Reference Harness:
Specific Use Case:
Difficulty Level:

Required Primitives:
Optional Primitives:
Forbidden Shortcuts:

Activation Scenario:
Primary Business Outcome:
Required HITL Scenario:
Required Failure Scenario:
Required Recovery Scenario:
Required Audit Scenario:
Required Eval Scenario:
```

Ejemplo:

```text
Reference Harness:
    Finance & Accounting

Specific Use Case:
    Invoice Processing

Difficulty:
    Level 3

Required:
    Activation
    Workflow
    ToolRuntime
    PolicyEngine
    HITL
    Idempotency
    AuditLedger
    Evals

Required failure:
    ERP timeout after invoice validation

Required recovery:
    resume without duplicating invoice creation

Required HITL:
    invoice exceeds delegated approval threshold
```

---

# 7. Reglas de evaluación

No evaluar únicamente si "el agente funciona".

Evaluar:

```text
Architecture
    25%

Contracts and State
    15%

Execution / Tool Use
    15%

Reliability
    10%

Security / Governance
    10%

HITL
    5%

Observability / Audit
    5%

Evals
    5%

Business Outcome
    5%

Documentation / ADRs
    5%
```

El estudiante debe demostrar tanto **happy path** como **failure path**.

---

# 8. Conformance

Cada Reference Harness debe demostrar que utiliza el Generic Harness sin modificar innecesariamente su core.

Regla:

> **Si una necesidad puede resolverse mediante un Domain Pack, adapter, skill, policy, capability o extension, no debe introducirse como lógica específica dentro del Generic Core.**

La práctica académica debe evaluar precisamente esa separación.

---

# 9. Uso como Architecture Conformance Suite

Los 15 Reference Harnesses también funcionan como pruebas de generalidad de la arquitectura.

```text
Architecture Principle
        ↓
Reference Harnesses
        ↓
Implementation Exercises
        ↓
Conformance Evidence
```

Si una primitive del Generic Harness solo funciona para un dominio, debe revisarse.

Si todos los dominios necesitan duplicar la misma primitive, esa primitive probablemente pertenece al Generic Harness.

Si solo uno o pocos dominios la requieren, probablemente pertenece al Domain Pack.

---

# 10. Resultado esperado

El catálogo permite construir:

```text
15 Reference Harnesses
        ×
8 ejercicios promedio
        =
~120 ejercicios posibles
```

sin convertir cada caso de uso en una arquitectura distinta.

La jerarquía canónica queda:

```text
Architecture Constitution
        ↓
Generic Agent Harness
        ↓
Reference Harness
        ↓
Domain Pack
        ↓
Specific Use Case
        ↓
Implementation
        ↓
Evals / Conformance
```
