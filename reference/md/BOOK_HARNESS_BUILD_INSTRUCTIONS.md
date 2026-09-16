# Build Instructions — Book Production Harness
## A Self-Hosting Harness for Writing the Agent Harness Book

## 1. Mission

Build a **Book Production Harness** whose purpose is to create, validate, revise, and publish the technical book *Construyendo un Agent Harness*.

The system must itself become a reference implementation of the architectural ideas taught by the book.

Do **not** build a generic "AI writer".

Build a governed editorial runtime where:

- agents reason, write, review, and propose;
- skills encode reusable editorial and architectural procedures;
- deterministic scripts validate structural consistency;
- policies govern architectural and publishing changes;
- state tracks the evolving book;
- a canonical source produces both the Web book and PDF book;
- human approval is required for important architectural decisions.

Apply the constitutional rule:

> **Probabilistic systems may propose decisions. Deterministic systems must govern consequences.**

---

# 2. Primary Outputs

The harness must produce two synchronized outputs:

```text
Canonical Book Source
        │
        ├──────────────► Web Book
        │
        └──────────────► PDF Book
```

Web and PDF must **never** become independent content sources.

They are renderings of the same validated canonical book model.

Required final artifacts:

```text
dist/
├── web/
└── book.pdf
```

---

# 3. Core Architectural Model

Implement the system around the following flow:

```text
                         BOOK HARNESS
                              │
                    ┌─────────┴─────────┐
                    │                   │
              Book Controller      Book State
                    │
                    ▼
             Editorial Pipeline
                    │
     ┌──────────────┼──────────────────┐
     ▼              ▼                  ▼
 Research       Architecture        Writing
                    │                  │
     └──────────────┼──────────────────┘
                    ▼
              Quality Gates
                    │
                    ▼
             Canonical Source
                    │
                    ▼
                   BookIR
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Web Renderer         PDF Renderer
          │                   │
          ▼                   ▼
       Website             book.pdf
```

Do not place rendering, model-provider logic, UI, or publishing behavior inside the core editorial state.

---

# 4. Canonical Book Source

Use structured Markdown as the human-editable source.

Recommended structure:

```text
book/
├── book.yaml
├── frontmatter/
│   ├── preface.md
│   └── introduction.md
├── chapters/
│   ├── 00-architecture-constitution/
│   │   └── chapter.md
│   ├── 01-agent-harness/
│   │   └── chapter.md
│   ├── 02-agent-core/
│   │   └── chapter.md
│   └── ...
├── appendices/
│   ├── contract-registry.md
│   ├── component-registry.md
│   └── glossary.md
└── assets/
```

Create `book/book.yaml` as the canonical structural manifest.

Example:

```yaml
book:
  id: agent-harness-book
  version: 0.1

chapters:
  - id: CH-00
    file: chapters/00-architecture-constitution/chapter.md

  - id: CH-01
    file: chapters/01-agent-harness/chapter.md

  - id: CH-02
    file: chapters/02-agent-core/chapter.md
```

The harness must reason about the book through this structure rather than treating the repository as an arbitrary collection of Markdown files.

---

# 5. Book State

Create explicit book state.

```text
STRUCT BookState
    bookVersion: Version
    chapters: List<ChapterState>
    contracts: ContractRegistry
    components: ComponentRegistry
    adrs: ADRRegistry
    glossary: Glossary
    dependencies: ArchitectureDependencyGraph
    buildStatus: BuildStatus
END
```

Each chapter must have structured state:

```text
STRUCT ChapterState
    chapterId: ChapterId
    title: Text
    status: ChapterStatus

    introducesComponents: List<ComponentId>
    introducesContracts: List<ContractId>

    modifiesComponents: List<ComponentId>
    modifiesContracts: List<ContractId>

    constitutionalArticles: List<ArticleId>

    previousChapter: Optional<ChapterId>
    nextChapter: Optional<ChapterId>
END
```

Use this state for deterministic validation.

Example:

```text
ToolRuntime introduced: Chapter 5
Current chapter: Chapter 3

Reference to ToolRuntime
→ INVALID
```

Do not rely only on prompts such as "do not use concepts that have not yet been introduced."

---

# 6. Agent Set

Start with a small set of specialized agents.

Do not begin with a large multi-agent architecture.

## 6.1 Book Architect Agent

Responsibilities:

```text
book structure
architecture consistency
chapter dependencies
concept progression
Constitution compliance
contract evolution
component evolution
```

This agent may propose architecture changes but must not automatically approve fundamental changes.

---

## 6.2 Chapter Author Agent

Responsibilities:

```text
write chapters
explain concepts
produce pseudocode
produce examples
produce diagrams as source
```

Provide a controlled context:

```text
Chapter Brief
+
Previous Architecture
+
Available Contracts
+
Available Components
+
Relevant Constitution Articles
+
Relevant ADRs
+
Editorial Rules
+
Relevant Skills
```

Do not dump the entire book into every authoring request.

---

## 6.3 Technical Reviewer Agent

Responsibilities:

```text
technical correctness
architecture reasoning
contract correctness
pseudocode correctness
component relationships
failure semantics
security implications
```

It proposes findings and corrections.

It does not silently change architecture.

---

## 6.4 Pedagogical Reviewer Agent

Responsibilities:

```text
reader comprehension
concept sequencing
undefined concepts
abstraction timing
example quality
chapter progression
explanation clarity
```

It should explicitly identify concepts used before being taught.

---

## 6.5 Consistency Reviewer Agent

Responsibilities:

```text
terminology consistency
cross-chapter consistency
architecture drift
contract drift
component naming
duplicated concepts
contradictory definitions
```

Where possible, implement equivalent checks deterministically in validators.

---

## 6.6 Editor Agent

Responsibilities:

```text
clarity
tone
redundancy
grammar
terminology
chapter flow
```

Constraint:

```text
Editor Agent
    CAN modify prose
    CANNOT modify architecture
    CANNOT redefine contracts
    CANNOT introduce components
```

Architectural changes must be routed back to the Book Architect workflow.

---

# 7. Skills

Create reusable procedural skills.

Initial structure:

```text
skills/
├── write-technical-chapter/
│   └── SKILL.md
├── define-contract/
│   └── SKILL.md
├── define-component/
│   └── SKILL.md
├── write-pseudocode/
│   └── SKILL.md
├── create-sequence-diagram/
│   └── SKILL.md
├── analyze-constitutional-impact/
│   └── SKILL.md
├── review-architecture/
│   └── SKILL.md
├── review-pedagogy/
│   └── SKILL.md
├── write-adr/
│   └── SKILL.md
└── revise-chapter/
    └── SKILL.md
```

Skills must encode procedure, not merely prompts.

Example rules for `write-pseudocode`:

```text
Never use undefined entities.

Define STRUCT before behavior.

Define INTERFACE before IMPLEMENTATION.

Use canonical names from registries.

Every FUNCTION declares:
- inputs
- output
- relevant errors

Use only the canonical pseudocode grammar.

Do not bypass established component boundaries.
```

---

# 8. Deterministic Scripts

Implement deterministic validators and builders.

Initial scripts:

```text
scripts/
├── validate-book
├── validate-chapter
├── validate-contracts
├── validate-components
├── validate-pseudocode
├── validate-links
├── validate-glossary
├── validate-sequence
├── build-dependency-graph
├── build-book-ir
├── build-web
└── build-pdf
```

`validate-chapter` must check at minimum:

```text
chapter metadata exists
referenced contracts exist
referenced components exist
no future component is referenced
required chapter sections exist
Constitutional Impact exists
Current Architecture exists
Architecture After This Chapter exists
failure semantics are documented
introduced entities are registered
modified entities are declared
```

The publishing pipeline must fail on structural validation errors.

---

# 9. Treat the Book System Like a Compiler

Use this mental model:

```text
                    BOOK SOURCE
                         │
                         ▼
                      Parser
                         │
                         ▼
                  Book Semantic Model
                         │
                         ▼
                  Semantic Validator
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Contracts   Components   References
              │          │          │
              └──────────┼──────────┘
                         ▼
                      BookIR
                         │
                 ┌───────┴───────┐
                 ▼               ▼
             Web Backend      PDF Backend
                 │               │
                 ▼               ▼
             Website          book.pdf
```

Do not implement the final pipeline as merely:

```text
Markdown → HTML
Markdown → PDF
```

Implement:

```text
Markdown
   ↓
Parsed Book Model
   ↓
Validated Book Model
   ↓
BookIR
   ↓
Multiple Renderers
```

---

# 10. Book Intermediate Representation

Create a renderer-independent `BookIR`.

```text
STRUCT BookIR
    metadata: BookMetadata
    chapters: List<ChapterIR>
    glossary: Glossary
    contracts: ContractRegistry
    components: ComponentRegistry
END
```

```text
STRUCT ChapterIR
    id: ChapterId
    sections: List<Section>
    diagrams: List<Diagram>
    codeBlocks: List<PseudocodeBlock>
    references: List<Reference>
    contractsIntroduced: List<ContractId>
    componentsIntroduced: List<ComponentId>
END
```

Renderers consume `BookIR`.

```text
WebRenderer.render(BookIR)

PDFRenderer.render(BookIR)
```

Renderers must not modify semantic content.

---

# 11. Web Renderer

The Web renderer should support:

```text
chapter navigation
previous / next navigation
architecture diagrams
expandable contracts
component references
ADR references
search
glossary
cross-references
responsive pseudocode blocks
```

Component names should be capable of linking to their canonical definitions.

Example:

```text
ToolRuntime
    ↓
CMP-004

Introduced:
Chapter 5

Depends on:
PolicyEngine
CapabilityRegistry

Consumes:
ToolCall

Produces:
ToolResult
```

---

# 12. PDF Renderer

The PDF renderer must consume the same `BookIR`.

Support:

```text
cover
copyright
table of contents
parts
chapters
headers
footers
page numbers
pseudocode blocks
diagrams
cross references
appendices
index
```

The PDF renderer controls presentation only.

It must not maintain an independent copy of chapter content.

---

# 13. Diagrams as Source

Do not make raster images the canonical diagram format.

Store diagram source:

```text
diagrams/
├── agent-loop.diagram
├── tool-runtime.diagram
├── hitl-flow.diagram
└── ...
```

Pipeline:

```text
Diagram Source
      ↓
Diagram Renderer
      ↓
Vector Asset
```

Prefer vector output.

Use:

```text
Web → responsive SVG
PDF → vector-compatible representation
```

---

# 14. Editorial Workflow

Implement this pipeline:

```text
                    Chapter Request
                          │
                          ▼
                    Book Architect
                          │
                          ▼
                    Chapter Brief
                          │
                          ▼
                    Author Agent
                          │
                          ▼
                       Draft
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
     Architecture      Technical       Pedagogical
       Review           Review           Review
          │               │                │
          └───────────────┼────────────────┘
                          ▼
                     Revision
                          │
                          ▼
                Deterministic Validators
                          │
                    ┌─────┴─────┐
                    │           │
                  FAIL         PASS
                    │           │
                    ▼           ▼
                 Revise      Editor
                                │
                                ▼
                           Canonical Source
                                │
                                ▼
                             BookIR
                          ┌─────┴─────┐
                          ▼           ▼
                        Web          PDF
```

Agents may iterate based on review findings.

Set explicit iteration and cost limits.

---

# 15. Human-in-the-Loop

Require human approval for important changes.

At minimum:

```text
Architecture Constitution modification
new fundamental core component
breaking contract change
major chapter restructuring
new architectural boundary
major ADR
removal of a canonical concept
```

Workflow:

```text
Agent proposes architecture change
        ↓
Architecture Policy
        ↓
REQUIRE_HUMAN_APPROVAL
        ↓
Persist proposal
        ↓
WAITING_FOR_HUMAN
        ↓
Approve / Reject / Modify
        ↓
Resume
```

Human interaction must be channel-independent.

Do not couple approval semantics to a TUI.

---

# 16. Policies

Create explicit policies.

Recommended structure:

```text
policies/
├── architecture.yaml
├── contracts.yaml
├── terminology.yaml
├── chapters.yaml
└── publishing.yaml
```

Example:

```yaml
contracts:
  undefinedReferences: deny
  breakingChanges: require_approval

architecture:
  newCoreComponent: require_approval
  constitutionChange: require_approval

publishing:
  unresolvedValidationErrors: deny
```

Policies must be deterministic wherever possible.

---

# 17. Evals

Evaluate generated chapters on multiple dimensions.

Required dimensions:

```text
Technical Accuracy
Architecture Consistency
Contract Consistency
Pedagogical Clarity
Concept Progression
Pseudocode Validity
Terminology Consistency
Constitution Compliance
Cross-Chapter Consistency
```

Store evaluation results by chapter and book version.

Do not reduce quality evaluation to a single generic "good writing" score.

---

# 18. Repository Layout

Use this as the initial target structure:

```text
agent-harness-book/
│
├── constitution/
│   └── ARCHITECTURE_CONSTITUTION.md
│
├── book/
│   ├── book.yaml
│   ├── frontmatter/
│   ├── chapters/
│   ├── appendices/
│   └── assets/
│
├── registry/
│   ├── contracts.yaml
│   ├── components.yaml
│   ├── glossary.yaml
│   └── adrs.yaml
│
├── agents/
│   ├── book-architect.md
│   ├── chapter-author.md
│   ├── technical-reviewer.md
│   ├── pedagogical-reviewer.md
│   ├── consistency-reviewer.md
│   └── editor.md
│
├── skills/
│   ├── write-technical-chapter/
│   ├── define-contract/
│   ├── define-component/
│   ├── write-pseudocode/
│   ├── create-sequence-diagram/
│   ├── analyze-constitutional-impact/
│   ├── review-architecture/
│   ├── review-pedagogy/
│   ├── write-adr/
│   └── revise-chapter/
│
├── policies/
│   ├── architecture.yaml
│   ├── contracts.yaml
│   ├── terminology.yaml
│   ├── chapters.yaml
│   └── publishing.yaml
│
├── scripts/
│   ├── validate-book
│   ├── validate-chapter
│   ├── validate-contracts
│   ├── validate-components
│   ├── validate-pseudocode
│   ├── validate-links
│   ├── validate-glossary
│   ├── validate-sequence
│   ├── build-dependency-graph
│   ├── build-book-ir
│   ├── build-web
│   └── build-pdf
│
├── packages/
│   ├── book-core/
│   ├── book-parser/
│   ├── book-validator/
│   ├── book-ir/
│   ├── web-renderer/
│   └── pdf-renderer/
│
├── apps/
│   ├── authoring/
│   └── web-book/
│
├── evals/
│
├── docs/
│   └── adr/
│
└── dist/
    ├── web/
    └── book.pdf
```

---

# 19. Self-Hosting Principle

The Book Harness must progressively reuse the same generic harness primitives taught by the book.

Target relationship:

```text
                    GENERIC HARNESS CORE
                           │
        ┌──────────────────┼───────────────────┐
        │                  │                   │
        ▼                  ▼                   ▼
   Book Agents       Coding Agents       Enterprise Agents
        │
        ▼
   Book Harness
```

Do not create an editorial architecture that cannot later be expressed through the generic harness.

As the generic harness evolves, migrate the Book Harness to use:

```text
AgentLoop
ContextEngine
ModelGateway
ToolRuntime
PolicyEngine
SessionManager
EventBus
ExecutionController
HumanInteractionService
Skills
Extensions
Evals
```

The Book Harness should become a practical proof that these primitives work.

---

# 20. Initial Milestone

Do not implement the complete architecture in the first iteration.

Build a small vertical slice.

## Milestone BH-v0.1

Implement:

```text
1 Book Architect Agent
1 Chapter Author Agent

Skills:
- write-technical-chapter
- define-contract
- define-component
- write-pseudocode
- analyze-constitutional-impact

Registries:
- contracts
- components
- glossary

Deterministic validators:
- validate-chapter
- validate-contracts
- validate-components

Canonical Markdown source

Minimal BookIR

Minimal Web Renderer

Minimal PDF Renderer
```

Required end-to-end flow:

```text
Chapter Request
      ↓
Book Architect
      ↓
Chapter Brief
      ↓
Chapter Author
      ↓
Draft
      ↓
Validators
      ↓
Canonical Source
      ↓
BookIR
   ┌──┴──┐
   ▼     ▼
 Web    PDF
```

The milestone is complete only when one real chapter can pass through this entire pipeline and produce both outputs from the same canonical source.

---

# 21. Subsequent Evolution

After BH-v0.1 works, evolve incrementally.

Suggested sequence:

```text
BH-v0.1
Basic authoring + validation + dual rendering

BH-v0.2
Technical and pedagogical reviewers

BH-v0.3
Consistency reviewer + dependency graph

BH-v0.4
Policies + architecture approval workflow

BH-v0.5
Durable HITL + persisted editorial runs

BH-v0.6
Evals + quality scorecards

BH-v0.7
Extension model + richer rendering

BH-v0.8
Automation for scheduled builds/reviews

BH-v0.9
Selective multi-agent orchestration

BH-v1.0
Book Harness running on the Generic Agent Harness
```

Do not introduce a later-stage capability merely because it appears in the target architecture.

Each version must solve a demonstrated limitation of the previous one.

---

# 22. Implementation Rules

1. Keep the core small.
2. Prefer deterministic validators over prompt instructions when a rule can be encoded.
3. Agents propose; policies and validators govern.
4. Do not use multi-agent complexity until the single-agent pipeline is reliable.
5. Do not allow renderer-specific content to leak into canonical chapter semantics.
6. Do not allow an editor agent to modify architecture.
7. Do not allow undefined contracts or components.
8. Do not allow future concepts to appear before their declared introduction unless explicitly marked as a preview.
9. Keep Web and PDF synchronized through `BookIR`.
10. Persist architectural decisions as ADRs.
11. Make important runs observable and traceable.
12. Track model usage, cost, iterations, failures, and validation results.
13. Every important agent action must operate under an explicit execution budget.
14. Every architectural change must identify its impact on the Architecture Constitution.
15. Prefer a working vertical slice over broad scaffolding with no end-to-end output.

---

# 23. Definition of Done

The system is not complete merely because agents can generate Markdown.

A production-ready Book Harness must demonstrate:

```text
[ ] Canonical source exists.
[ ] Book structure is machine-readable.
[ ] BookState is explicit.
[ ] Contracts and components are registered.
[ ] Chapter dependencies are validated.
[ ] Agents use controlled context.
[ ] Skills encode reusable procedures.
[ ] Deterministic quality gates exist.
[ ] Architectural changes can require human approval.
[ ] BookIR is renderer-independent.
[ ] Web and PDF derive from the same BookIR.
[ ] Cross-chapter terminology is consistent.
[ ] Pseudocode references only defined entities.
[ ] Architecture evolution is traceable through ADRs.
[ ] Editorial runs are observable.
[ ] Evals can measure chapter quality.
[ ] One command/pipeline can build the complete book.
```

---

# 24. Final Design Principle

Do not optimize first for the number of agents.

Optimize for the integrity of the production system:

```text
Agents
    reason and propose

Skills
    encode procedures

Registries
    define canonical truth

Scripts
    validate deterministically

Policies
    govern change

State
    tracks reality

Humans
    approve consequential decisions

BookIR
    separates semantics from presentation

Renderers
    produce Web and PDF
```

The final system should demonstrate the central thesis of the book by its own operation:

> **The book is produced by the same architectural principles that the book teaches.**
