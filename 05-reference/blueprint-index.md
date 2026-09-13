# Blueprint Index

Navigation index for
[`knowhub_master_blueprint.html`](../../knowhub_master_blueprint.html) —
97 top-level sections (§0–§85) in a single HTML file.

This is an index, not a summary. Each row says what a section is for so
you can find it; the blueprint itself remains the content.

## How to read the blueprint

- Every top-level section has a stable anchor. Links below jump straight
  to it.
- Sub-section rows (marked `↳`) share their parent section's anchor —
  sub-headings carry no anchor of their own. Jump to the parent, then use
  the in-page search box for the sub-heading.
- The blueprint renders its own table of contents and a full-text search
  box at the top of the page. Use those for depth beyond this index.
- **Reading rule (§2).** The blueprint gives priority to current
  executable source over generated documentation or README descriptions
  where they differ, and badges material as `CURRENT PATH`,
  `RETAINED / HISTORIC` or `TARGET`. Honour those badges when reading any
  section that describes the upstream repositories.
- Sections marked **normative** below are labelled as such by the
  blueprint itself. Absence of the marker is not a classification — most
  sections carry no such label.
- Terminology: see [`glossary.md`](glossary.md) (§83).
  Build order: see
  [`../00-start-here/delivery-roadmap.md`](../00-start-here/delivery-roadmap.md) (§81).

---

## Product and scope

| § | Section | Use when |
|---|---|---|
| **0** | [Build brief for a coding harness](../../knowhub_master_blueprint.html#harness-brief) — **normative** | You need the implementation orientation and the normative target: Python-first, API-first, not a Rust/Tauri transliteration. |
| ↳ 0.1 | Upstream repositories used as behavioral references | You need to know what Terrain and Litho/deepwiki-rs contribute and how KnowHub may use them. |
| ↳ 0.2 | What KnowHub is | You need the "is / is not" product definition. |
| ↳ 0.3 | The question KnowHub should answer | You need the product promise and the Intended / Implemented / Observed truth layers. |
| **1** | [Executive conclusion](../../knowhub_master_blueprint.html#executive) | You want the shortest statement of how the two upstream repositories relate. |
| **2** | [Audit method and an important reading rule](../../knowhub_master_blueprint.html#method) | Before reading any upstream-audit section: current source vs retained design vs target. |
| **3** | [What each repository is responsible for](../../knowhub_master_blueprint.html#comparison) | You need the division of responsibility between the two references. |
| **25** | [KnowHub enterprise target state — design principles](../../knowhub_master_blueprint.html#target-principles) | You need the principles that constrain every enterprise design decision. |
| **26** | [Enterprise inputs and outputs](../../knowhub_master_blueprint.html#target-input-output) | You need the enterprise-level input/output boundary. |
| **43** | [KnowHub final target-state mental model](../../knowhub_master_blueprint.html#final-architecture) | You want the whole-product mental model on one diagram. |
| **80** | [Scope boundary — MVP, Enterprise V1 & future](../../knowhub_master_blueprint.html#scope-boundary) | You need to know whether a capability belongs to MVP, Enterprise V1 or Future. |

## Application landscape

| § | Section | Use when |
|---|---|---|
| **0A** | [Full KnowHub application landscape](../../knowhub_master_blueprint.html#product-landscape) | You want the end-to-end flow from sources through compilation to experiences, with cross-cutting controls. |
| **0C** | [What the KnowHub web application must do](../../knowhub_master_blueprint.html#webapp-contract) | You need the web application's functional contract. |
| ↳ 0C.1 | Primary navigation model | You are designing top-level navigation. |
| ↳ 0C.2 | The default enquiry layout | You need the default enquiry screen structure. |
| ↳ 0C.3 | Principal user groups | You need the engineering/delivery and governance/enterprise audiences. |
| **44** | [Enterprise experience model and personas](../../knowhub_master_blueprint.html#enterprise-experience) | You need the persona set behind a UX or permission decision. |
| ↳ 44.1 | Primary personas | You need the persona definitions. |
| ↳ 44.2 | Scope selector is mandatory — **mandatory** | You are designing any scoped view; the scope selector is required. |
| **45** | [UI information architecture and required scenes](../../knowhub_master_blueprint.html#ui-information-architecture) | You need the required screen inventory and how scenes relate. |
| ↳ 45.1 | Scene catalogue | You need the enumerated list of scenes. |

## Architecture

| § | Section | Use when |
|---|---|---|
| **0G** | [Implementation boundaries and first build slice](../../knowhub_master_blueprint.html#implementation-boundaries) | You need the boundaries a build must respect and what is deferred. |
| ↳ 0G.3 | Architectural invariants for the code harness | You need the invariants every implementation must preserve (evidence first, permission before retrieval, provenance, determinism split, idempotency, versioning, API-first). |
| **27** | [Target architecture — Python enterprise knowledge platform](../../knowhub_master_blueprint.html#target-architecture) | You need the target platform architecture. |
| **29** | [Storage architecture](../../knowhub_master_blueprint.html#target-storage) | You need which store owns what. |
| **30** | [Backend codebase — knowhub-backend](../../knowhub_master_blueprint.html#target-python) | You are working in the backend repository. |
| ↳ 30.1 | Backend runtime processes from one backend codebase | You need the process model (API, workers, schedulers) from a single codebase. |
| ↳ 30.2 | Backend owns all security and knowledge truth | You are tempted to move a security or truth decision out of the backend. |
| **54** | [Implementation notes for the rewrite](../../knowhub_master_blueprint.html#appendix) | You want the rewrite guidance that accompanies the reverse-engineering appendix. |
| **61** | [Backend Python file-level target specification](../../knowhub_master_blueprint.html#python-file-spec) | You need the intended backend package and module layout. |
| ↳ 61.1 | Hard boundaries between packages | You are about to add a cross-package import. |
| ↳ 61.2 | Services that must stay deterministic | You are deciding whether a service may call an LLM. |
| ↳ 61.3 | Services allowed to infer/synthesize with an LLM | Same decision, from the other direction. |
| **71** | [Backend package boundaries & cross-repository dependency rules](../../knowhub_master_blueprint.html#package-dependency-rules) | You need the dependency direction and forbidden dependencies. |
| ↳ 71.1 | Dependency direction | You need the allowed direction of dependency. |
| ↳ 71.2 | Rules | You need the stated package rules. |
| ↳ 71.3 | Cross-repository rules | You are crossing the backend/frontend boundary. |
| ↳ 71.4 | Forbidden dependencies | You want the explicit prohibitions. |

## Technology stack

| § | Section | Use when |
|---|---|---|
| **0D** | [API-first product contract](../../knowhub_master_blueprint.html#api-contract) | You are designing or consuming the public API surface. |
| ↳ 0D.1 | Recommended API groups | You need the API grouping. |
| ↳ 0D.2 | Source registration example | You want a concrete request/response shape. |
| ↳ 0D.3 | Asynchronous job and event contract | You are designing job submission, polling or event streaming. |
| **0E** | [Recommended technology stack & implementation baseline](../../knowhub_master_blueprint.html#runtime-architecture) — **normative** | You need the normative implementation baseline for the Python rewrite. |
| ↳ 0E.1 | Technology decision matrix | You need the per-concern technology choice. |
| ↳ 0E.3 | Backend/service architecture | You need the backend service decomposition. |
| ↳ 0E.4 | Taskiq, LangGraph and PostgreSQL have different jobs | You are unsure which mechanism owns a piece of work. |
| ↳ 0E.5 | Logical storage responsibilities | You need which store is responsible for what. |
| ↳ 0E.6 | Retrieval stack — progressive, not all-at-once | You are considering adding retrieval infrastructure. |
| ↳ 0E.7 | Recommended V1 deployment topology | You are planning deployment. |
| ↳ 0E.8 | Two-repository delivery model — **normative** | You need the backend/frontend split, OpenAPI as shared contract, CI/CD and streaming boundary. |
| ↳ 0E.9 | Explicit non-choices for V1 | You are about to introduce something V1 deliberately excludes. |
| ↳ 0E.10 | Number intentionally unused | Nothing to read; the number is deliberately vacant. |
| **37** | [Suggested FastAPI surface](../../knowhub_master_blueprint.html#target-api) | You are adding or reviewing an endpoint. |
| **38** | [Job orchestration and idempotency](../../knowhub_master_blueprint.html#target-jobs) | You are implementing background work. |
| **69** | [Error, retry & failure semantics](../../knowhub_master_blueprint.html#failure-semantics) | You need the error taxonomy and retry behaviour. |
| ↳ 69.1 | Error taxonomy | You need to classify an error. |
| ↳ 69.2 | Idempotency | You are making an operation safely retryable. |
| ↳ 69.3 | Partial success | An operation can partly succeed. |
| ↳ 69.4 | Dead-letter/quarantine model | You need the failure-of-last-resort path. |
| **70** | [Canonical API & Pydantic contracts](../../knowhub_master_blueprint.html#api-contracts) | You need the canonical request/response contracts. |
| ↳ 70.1 | Resource envelopes | You need the standard envelope shape. |
| ↳ 70.2 | Sync/run contracts | You are modelling a sync or run. |
| ↳ 70.3 | Inquiry contract | You are implementing Ask. |
| ↳ 70.4 | Impact contract | You are implementing change impact. |
| ↳ 70.5 | API behavior rules | You need the cross-cutting API rules. |
| **76** | [Environment & deployment matrix](../../knowhub_master_blueprint.html#environment-matrix) | You need the environment set and what differs between them. |
| ↳ 76.1 | Promotion rules | You are promoting a build between environments. |
| **77** | [Configuration model & feature flags](../../knowhub_master_blueprint.html#configuration-model) | You are adding configuration or a flag. |
| ↳ 77.1 | Configuration domains | You need where a setting belongs. |
| ↳ 77.2 | Precedence | Two configuration sources disagree. |

## Identity and security

Authentication, session and RBAC contracts are defined in §0E.13–§0E.16
(§79 states this explicitly).

| § | Section | Use when |
|---|---|---|
| ↳ 0E.13 | Enterprise identity, authentication, token & session architecture (in [§0E](../../knowhub_master_blueprint.html#runtime-architecture)) — **normative** | You are touching authentication, tokens or sessions. Covers auth modes, IdP abstraction, browser/BFF flow, first-run bootstrap, local credential security, OIDC contract, token ownership, token validation, session lifecycle, CSRF/XSS/CORS hardening. |
| ↳ 0E.14 | Authorization architecture, RBAC/ABAC, user & role management — **normative** | You are touching authorization. Covers the decision formula, permission taxonomy, logical roles, module-access matrix, user and role management, break-glass, implementation pattern, identity/access APIs. |
| ↳ 0E.15 | Frontend authentication, authorization & administration implementation — **normative** | You are implementing auth or admin in the frontend. |
| ↳ 0E.16 | Security verification / penetration-test readiness — **normative** | You are preparing for security verification. |
| ↳ 0E.18 | Security references for implementation | You want the external security references. |
| **35** | [Enterprise security and governance requirements](../../knowhub_master_blueprint.html#target-security) | You need the enterprise security and governance requirement set. |
| **49** | [Enterprise authorization, source ACL propagation and safe inquiry](../../knowhub_master_blueprint.html#enterprise-permissions) | You are making evidence or answers permission-aware. |
| ↳ 49.1 | Security model | You need the authorization model for evidence. |
| ↳ 49.2 | Permission-aware derived knowledge | Derived knowledge must inherit source permissions. |
| ↳ 49.3 | Enterprise audit requirements | You need what must be audited. |
| **72** | [Security threat model & abuse cases](../../knowhub_master_blueprint.html#threat-model) | You are threat-modelling a change. |
| ↳ 72.1 | Trust boundaries | You need the trust boundaries. |
| ↳ 72.2 | Required abuse cases | You need the abuse cases that must be covered. |
| ↳ 72.3 | Security validation gates | You need the security gates a release must pass. |
| **79** | [Personas, RBAC & stewardship](../../knowhub_master_blueprint.html#personas-rbac) | You are mapping personas to roles and stewardship scope. |
| ↳ 79.1 | Separation of duties | You need the separation-of-duties constraints. |

## Persistence and data

| § | Section | Use when |
|---|---|---|
| **28** | [Canonical enterprise data model](../../knowhub_master_blueprint.html#target-data) | You need the canonical enterprise entities. |
| **62** | [Canonical enterprise data contracts](../../knowhub_master_blueprint.html#canonical-contracts) | You need the contract shape for a canonical concept. |
| ↳ 62.1 | Core identities | You are modelling application, source or principal identity. |
| ↳ 62.2 | Evidence and claims | You are modelling evidence or knowledge claims. |
| ↳ 62.3 | Graph contract | You are modelling entities and relationships. |
| ↳ 62.4 | Artifact contract | You are modelling a generated artifact. |
| ↳ 62.5 | Change and freshness contracts | You are modelling change sets, baselines or freshness. |
| ↳ 62.6 | Research-run contract | You are modelling a Foundry research run. |
| **66** | [Canonical logical & physical data model](../../knowhub_master_blueprint.html#canonical-data-model) | You are writing a migration or designing tables. |
| ↳ 66.1 | Core relationship model | You need the relationships between core tables. |
| ↳ 66.2 | Canonical table families | You need which table family a new table belongs to. |
| ↳ 66.3 | Persistence invariants | You must not break a stated persistence invariant. |
| ↳ 66.4 | Recommended database constraints | You are adding constraints or indexes. |
| **67** | [Lifecycle state machines](../../knowhub_master_blueprint.html#state-machines) | You are implementing or changing a state transition. |
| ↳ 67.1 | Application readiness | Application-level states. |
| ↳ 67.2 | Source lifecycle | Source states from registration onward. |
| ↳ 67.3 | Run lifecycle | Run states. |
| ↳ 67.4 | Knowledge artifact lifecycle | Artifact states. |
| ↳ 67.5 | Knowledge claim lifecycle | Claim states, including promotion to approved. |
| ↳ 67.6 | UI behavior | How states must surface in the UI. |
| **78** | [Schema, ontology, parser, prompt & model versioning](../../knowhub_master_blueprint.html#versioning-migrations) | You are changing anything that affects generated knowledge. |
| ↳ 78.1 | Rebuild hierarchy | You need to know what a version bump forces to rebuild. |

## Sources and ingestion

| § | Section | Use when |
|---|---|---|
| **0B** | [What data is given to KnowHub and how](../../knowhub_master_blueprint.html#source-landscape) | You need the canonical source types and how they arrive. |
| ↳ 0B.1 | Canonical source types | You need the source families and what is retained per family. |
| ↳ 0B.2 | Source onboarding sequence | You are implementing onboarding for a source. |
| ↳ 0B.3 | Preserve native structure: do not flatten everything into chunks | You are about to chunk a structured source. |
| **47** | [Enterprise multi-source ingestion architecture](../../knowhub_master_blueprint.html#enterprise-ingestion) | You are adding or designing a connector. |
| ↳ 47.1 | Source families KnowHub should support | You need the target connector estate. |
| ↳ 47.2 | Connector contract | You need what every connector must implement. |
| **48** | [Canonical source model, document parsing and relationship resolution](../../knowhub_master_blueprint.html#source-normalization) | You are normalizing a source into canonical form. |
| ↳ 48.1 | Canonical revision model | You are modelling revisions for a new source type. |
| ↳ 48.2 | Source-specific normalization | You need per-source normalization rules. |
| ↳ 48.3 | Relationship resolution hierarchy | You are resolving links between sources. |
| **50** | [Connector synchronization, deletion, freshness and semantic refresh](../../knowhub_master_blueprint.html#sync-freshness) | You are implementing sync, deletion or refresh. |
| ↳ 50.1 | Generalized source baseline | You need the baseline concept for a non-Git source. |
| ↳ 50.2 | Change event pipeline | You are handling change events. |
| ↳ 50.3 | Deletion and access-revocation are first-class | A source item is deleted or access is revoked. |
| ↳ 50.4 | Freshness dimensions | You need the dimensions freshness is measured on. |

## Knowledge Foundry

| § | Section | Use when |
|---|---|---|
| — | [KnowHub internal naming — Knowledge Foundry](../../knowhub_master_blueprint.html#knowhub-naming) | Before naming anything in the Foundry subsystem; states the naming rule and recommended runtime identifiers. |
| **31** | [Knowledge Foundry — Python knowledge synthesis engine](../../knowhub_master_blueprint.html#target-litho) | You are implementing preprocess → research → compose → validate. |
| ↳ — | Recommended research output contract | You need the shape of a research output. |
| **34** | [Freshness and semantic incremental regeneration](../../knowhub_master_blueprint.html#target-freshness) | You are implementing freshness or incremental regeneration. |
| ↳ — | State decision | You need the refresh-state decision rule. |

Foundry-adjacent: deterministic vs LLM-permitted services are split in
§61.2/§61.3 (Architecture); the failure-and-resume sequence is §68.7
(Retrieval and Ask); the upstream behavioural reference is §§5–11 and
§§57–59 (Upstream reference).

## AI and model architecture

| § | Section | Use when |
|---|---|---|
| ↳ 0E.11 | Model gateway, provider portability & local inference (in [§0E](../../knowhub_master_blueprint.html#runtime-architecture)) — **normative** | You are calling a model. Covers the gateway boundary, normalized contracts, provider portability and local inference. |
| ↳ 0E.12 | DSPy — selective AI-programming & optimization layer | You are considering DSPy; states its scope and placement. |
| **73** | [LLM & agent interaction contract](../../knowhub_master_blueprint.html#llm-agent-contract) | You are designing any model-assisted step. |
| ↳ 73.1 | When an LLM may be used | You need permission to use a model for a task. |
| ↳ 73.2 | When an LLM must not replace deterministic logic | You are tempted to replace parsing or ACL logic with a prompt. |
| ↳ 73.3 | Tool contract | You are giving a model tools. |
| ↳ 73.4 | Structured output and publication | You are publishing model output. |
| ↳ 73.5 | Evidence sufficiency | You need the bar for answering at all. |

## Retrieval and Ask

| § | Section | Use when |
|---|---|---|
| **0F** | [End-to-end behavior the implementation must support](../../knowhub_master_blueprint.html#end-to-end-flows) | You need the behaviours the product must demonstrate end to end. |
| ↳ 0F.1 | Initial application onboarding | First-run flow for an application. |
| ↳ 0F.2 | Incremental change | Behaviour when a source changes. |
| ↳ 0F.3 | Ask | The Ask behaviour at a glance. |
| ↳ 0F.4 | Change-impact request | The impact-request behaviour. |
| ↳ 0F.5 | Knowledge generation | The generation behaviour. |
| **33** | [Retrieval and Q&A](../../knowhub_master_blueprint.html#target-retrieval) | You are implementing retrieval or the macro/meso/micro escalation. |
| ↳ — | Tool surface | You need the retrieval tool surface. |
| **46** | [Ask workspace — the primary inquiry scene](../../knowhub_master_blueprint.html#ask-workspace) | You are building or changing Ask. |
| ↳ 46.1 | Recommended three-pane layout | You are laying out the Ask scene. |
| ↳ 46.2 | What every material answer should expose | You are deciding what an answer must show. |
| ↳ 46.3 | Inquiry modes | You need the modes: Explain, Locate, Trace, Impact, Compare, Review. |
| ↳ 46.4 | Example cross-source question | You want a worked cross-source example. |
| **52** | [Enterprise inquiry journeys and revised target state](../../knowhub_master_blueprint.html#enterprise-inquiry-flows) | You want the journeys the platform must support, and the revised target architecture. |
| ↳ 52.1 | Journey: requirement to implementation | Tracing intent to code. |
| ↳ 52.2 | Journey: architecture decision conformance | Comparing an ADR against current code. |
| ↳ 52.3 | Journey: cross-application change impact | Impact across applications. |
| ↳ 52.4 | Journey: "ask the enterprise" without violating boundaries | Broad inquiry under permission constraints. |
| ↳ 52.5 | Revised enterprise target architecture | The architecture after the enterprise journeys. |
| ↳ 52.6 | Additional backend Python packages required | Packages the enterprise journeys add. |
| ↳ 52.7 | Enterprise acceptance criteria added to the blueprint | The acceptance criteria these journeys introduce. |
| **68** | [End-to-end sequence specifications](../../knowhub_master_blueprint.html#sequence-specifications) | You need the step-by-step sequence for a named flow. |
| ↳ 68.1 | Git repository onboarding | Onboarding a Git source. |
| ↳ 68.2 | Jira incremental synchronization | Incremental Jira sync. |
| ↳ 68.3 | Uploaded project document | Handling an uploaded document. |
| ↳ 68.4 | Grounded inquiry | The full Ask sequence. |
| ↳ 68.5 | Permission change | A permission changes upstream. |
| ↳ 68.6 | Deletion/revocation | Source content is deleted or access revoked. |
| ↳ 68.7 | Knowledge Foundry failure and resume | A Foundry run fails and must resume. |

## Graph and traceability

| § | Section | Use when |
|---|---|---|
| **32** | [Code graph and semantic relationship model](../../knowhub_master_blueprint.html#target-graph) | You are modelling entities, relationships or traversals. |
| **51** | [Enterprise knowledge governance, authority and contradiction handling](../../knowhub_master_blueprint.html#enterprise-knowledge-governance) | Sources disagree, or a claim needs an authority level. |
| ↳ 51.1 | Suggested authority classes | You need the authority classes. |
| ↳ 51.2 | Contradiction is a feature, not an error to hide | You are tempted to merge conflicting sources. |
| ↳ 51.3 | Human stewardship | You need the stewardship model. |

## Frontend / UX

| § | Section | Use when |
|---|---|---|
| ↳ 0E.2 | UI/UX implementation baseline (in [§0E](../../knowhub_master_blueprint.html#runtime-architecture)) | You need the baseline for portfolio navigation, Ask, evidence-first interaction, graph, long-running work, accessibility and responsiveness. |
| ↳ 0E.17 | Premium bright UI/UX visual specification | You are making a visual decision. Covers visual direction, application shell, Ask, graph/traceability, administration UX, interaction polish, accessibility/responsive behaviour, prohibited anti-patterns and design-system deliverables. |
| **30A** | [Frontend codebase — knowhub-frontend](../../knowhub_master_blueprint.html#target-frontend) | You are working in the frontend repository. |
| ↳ 30A.1 | Frontend responsibility boundary | You are deciding whether logic belongs in the frontend. |
| ↳ 30A.2 | Contract update workflow | The backend contract changed. |
| **61A** | [Frontend file-level target specification](../../knowhub_master_blueprint.html#frontend-file-spec) | You need the intended frontend structure. |
| ↳ 61A.1 | Route map | You are adding a route. |
| ↳ 61A.2 | Frontend layering | You need the layer a module belongs to. |
| ↳ 61A.3 | UI state classes | You are handling loading, empty, partial or error states. |
| ↳ 61A.4 | Frontend quality gates | You need the frontend quality bar. |

## Testing and evaluation

| § | Section | Use when |
|---|---|---|
| **36** | [Observability, evaluation and reliability](../../knowhub_master_blueprint.html#target-observability) | You are adding telemetry, evaluation or reliability behaviour. |
| **41** | [Acceptance criteria for "core Terrain + Litho parity"](../../knowhub_master_blueprint.html#acceptance) | You need the parity acceptance checklist. |
| **60** | [KnowHub complete behavioral parity matrix](../../knowhub_master_blueprint.html#parity-matrix) | You need per-behaviour parity expectations. |
| **63** | [Porting strategy and definition of done](../../knowhub_master_blueprint.html#porting-dod) | You need what to port, redesign or skip, and what "done" means. |
| ↳ 63.1 | Do not line-by-line port Rust | You are deciding how to treat upstream code. |
| ↳ 63.2 | Golden parity corpus | You are building the test corpus. |
| ↳ 63.3 | Definition of P0 completion | You need the P0 completion checklist. |
| ↳ 63.4 | Definition of enterprise target completion | You need the enterprise completion checklist. |
| ↳ 63.5 | Recommended implementation gates | You want the gate sequence A–F. |
| **65** | [Non-functional requirements & engineering constraints](../../knowhub_master_blueprint.html#non-functional-requirements) | You need latency, capacity, compliance, accessibility or cost constraints. |
| ↳ 65.1 | Availability, latency and responsiveness | You need the performance targets. |
| ↳ 65.2 | Initial capacity envelope | You need the sizing assumptions. |
| ↳ 65.3 | Security, privacy and compliance constraints | You need the compliance constraints. |
| ↳ 65.4 | Accessibility, browser and UX constraints | You need the accessibility and browser bar. |
| ↳ 65.5 | Cost and model-usage guardrails | You need the cost guardrails. |
| **74** | [Golden reference application](../../knowhub_master_blueprint.html#golden-reference-application) | You are writing tests against the reference estate. |
| ↳ 74.1 | Reference estate | You need the reference application's repositories, Jira project and documents. |
| ↳ 74.2 | Required canonical facts | You need the facts the system must derive. |
| ↳ 74.3 | Golden questions | You need the questions Ask must answer correctly. |
| ↳ 74.4 | Golden change scenarios | You need the change scenarios to exercise. |
| **75** | [Acceptance criteria & executable test catalogue](../../knowhub_master_blueprint.html#acceptance-tests) | You need the test catalogue and acceptance IDs. |
| ↳ 75.1 | Test layers | You need which layer a test belongs to. |

## Implementation sequencing

| § | Section | Use when |
|---|---|---|
| **81** | [Implementation dependency graph & execution order](../../knowhub_master_blueprint.html#implementation-dependency-graph) | You need the authoritative build order. Mirrored in [`../00-start-here/delivery-roadmap.md`](../00-start-here/delivery-roadmap.md). |
| ↳ 81.1 | Milestone exit rule | You are deciding whether a milestone is finished. |
| **39** | [Build roadmap — from useful clone to enterprise platform](../../knowhub_master_blueprint.html#target-roadmap) | You want the P0–P7 phase view with exit criteria. |
| **40** | [Feature-parity ledger](../../knowhub_master_blueprint.html#parity) | You need the MUST / ADAPT / IMPROVE / SKIP PORT verdict for a feature. |
| **42** | [What I would build first — concrete 12-week engineering sequence](../../knowhub_master_blueprint.html#first-build) | You want the week-by-week view. |

Also relevant: §0G.1 First useful vertical slice and §0G.2 Explicitly
defer from the initial Python rewrite, both inside
[§0G](../../knowhub_master_blueprint.html#implementation-boundaries);
§63.5 implementation gates (Testing and evaluation); §80 scope boundary
(Product and scope). The blueprint does not state how these views
reconcile with §81.

## ADRs

| § | Section | Use when |
|---|---|---|
| **82** | [Architectural Decision Record register](../../knowhub_master_blueprint.html#adr-register) | Before proposing any architectural change. Lists ADR-001 to ADR-025 as decisions already made, and the rule that reversing one requires a superseding ADR. |

## Governance

| § | Section | Use when |
|---|---|---|
| **83** | [Glossary & terminology contract](../../knowhub_master_blueprint.html#glossary) — **normative** | You need the normative meaning of a term. Extracted to [`glossary.md`](glossary.md). |
| **84** | [Coding-harness implementation instructions](../../knowhub_master_blueprint.html#coding-harness-instructions) | You are implementing anything; this is implementation governance. |
| ↳ 84.1 | Mandatory working style | You need the working rules, including implementing in §81 order. |
| ↳ 84.2 | Definition of a completed implementation task | You are about to call something done. |
| ↳ 84.3 | Repository hygiene expected from the harness | You need the expected directory layout of the two implementation repositories. |
| ↳ 84.4 | Stop conditions | You have hit a conflict and need to know whether to stop. |

Release-governance material sits in §85.7 and §85.8 (Future backlog).

## Future backlog

| § | Section | Use when |
|---|---|---|
| **85** | [Future enterprise backlog & product readiness](../../knowhub_master_blueprint.html#future-enterprise-backlog) | You are asked for a capability that is deliberately deferred. |
| ↳ 85.1 | Backlog interpretation | You need how to read this backlog. |
| ↳ 85.2 | P0 — Enterprise production gates | The deferred items that still gate production. |
| ↳ 85.3 | P1 — Scale, quality and operational maturity | Scale and maturity items. |
| ↳ 85.4 | P2 — Strategic product differentiation | Differentiation items. |
| ↳ 85.5 | Suggested horizon sequencing | You need the horizon ordering. |
| ↳ 85.6 | How backlog items should become delivery epics | You are moving a backlog item into a tracker. |
| ↳ 85.7 | Product-readiness scorecard to revisit at each major release | You are preparing a release review. |
| ↳ 85.8 | External control frameworks worth using when these epics are activated | You need the external frameworks referenced. |

## Upstream reference — Terrain and Litho/deepwiki-rs

These sections describe the two read-only reference repositories. Apply
the §2 reading rule: prefer current executable source over generated
documentation, and honour the `CURRENT PATH` / `RETAINED / HISTORIC` /
`TARGET` badges.

### Litho / deepwiki-rs

| § | Section | Use when |
|---|---|---|
| **4** | [Litho/deepwiki-rs — complete functional model](../../knowhub_master_blueprint.html#litho-overview) | You want the whole Litho model. |
| **5** | [Litho preprocessing — what the current path actually does](../../knowhub_master_blueprint.html#litho-preprocess) | You are implementing Foundry preprocessing. |
| **6** | [Litho memory, context and declarative agent contract](../../knowhub_master_blueprint.html#litho-memory) | You are designing Foundry state or the agent contract. |
| **7** | [Litho research agents — responsibilities and dependency chain](../../knowhub_master_blueprint.html#litho-research) | You are implementing a specialist researcher. |
| **8** | [Litho composition, document tree and output validation](../../knowhub_master_blueprint.html#litho-compose) | You are implementing composition or validation. |
| **9** | [Litho external knowledge ingestion and routing](../../knowhub_master_blueprint.html#litho-knowledge) | You are routing external knowledge into research. |
| **10** | [Litho LLM execution, caching and operational controls](../../knowhub_master_blueprint.html#litho-llm) | You are designing model execution, caching or budgets. |
| **11** | [Litho subtleties and wiring issues worth fixing rather than porting](../../knowhub_master_blueprint.html#litho-subtleties) | You want the known upstream problems not to reproduce. |
| **57** | [Litho/deepwiki-rs reverse-engineering inventory](../../knowhub_master_blueprint.html#litho-file-inventory) | You need the file-level Litho inventory. |
| **58** | [Litho execution contracts and subtleties](../../knowhub_master_blueprint.html#litho-runtime-contracts) | You need Litho's runtime contracts and production heuristics. |
| **58A** | [Final upstream delta audit — capabilities that must survive the rewrite](../../knowhub_master_blueprint.html#final-upstream-delta-audit) | You are checking nothing was lost in the rewrite. |
| **59** | [Current code vs documented / retained design](../../knowhub_master_blueprint.html#current-vs-documented) | A documented behaviour and the current source disagree. |

### Terrain

| § | Section | Use when |
|---|---|---|
| **12** | [Terrain — complete functional model](../../knowhub_master_blueprint.html#terrain-overview) | You want the whole Terrain model. |
| **13** | [Terrain core — deterministic services and overlooked utilities](../../knowhub_master_blueprint.html#terrain-core) | You are implementing deterministic services. |
| **14** | [Terrain repository scan, structured ingestion and source pack](../../knowhub_master_blueprint.html#terrain-scan-pack) | You are implementing scan, pack or OpenAPI ingestion. |
| **15** | [Terrain developer metadata and domain knowledge inputs](../../knowhub_master_blueprint.html#terrain-meta) | You are designing repo metadata or domain hints. |
| **16** | [Terrain machine context — the macro/meso contract](../../knowhub_master_blueprint.html#terrain-context) | You are generating agent context. |
| **17** | [Terrain's current Litho integration](../../knowhub_master_blueprint.html#terrain-litho) | You want how Terrain actually invokes Litho, and its workspace contract. |
| **18** | [Terrain knowledge search, Q&A tools, citations and sessions](../../knowhub_master_blueprint.html#terrain-search) | You are implementing search, citations or sessions. |
| **19** | [Terrain freshness — trust is a runtime behavior](../../knowhub_master_blueprint.html#terrain-freshness) | You are implementing freshness or trust. |
| **20** | [Terrain incremental refresh — conservative diff-driven editing](../../knowhub_master_blueprint.html#terrain-incremental) | You are implementing incremental refresh and preservation guards. |
| **21** | [Terrain initialization versus quick refresh](../../knowhub_master_blueprint.html#terrain-refresh) | You need the difference between init and quick refresh. |
| **22** | [CodeGraph, RTK, skills, AGENTS.md and environment management](../../knowhub_master_blueprint.html#terrain-codegraph-env) | You are designing graph integration, skills or environment handling. |
| **23** | [Terrain SDD — requirements to code review](../../knowhub_master_blueprint.html#terrain-sdd) | You are designing the SDD workflow. |
| **24** | [Litho → Terrain evolution map](../../knowhub_master_blueprint.html#evolution) | You want how the two systems relate as an evolution. |
| **55** | [Terrain reverse-engineering inventory](../../knowhub_master_blueprint.html#terrain-file-inventory) | You need the file-level Terrain inventory. |
| **56** | [Terrain execution contracts and call graphs](../../knowhub_master_blueprint.html#terrain-runtime-contracts) | You need Terrain's runtime contracts and call graphs. |

### Source ledger

| § | Section | Use when |
|---|---|---|
| **53** | [Source ledger](../../knowhub_master_blueprint.html#sources) | You need the pinned commits and the primary-source citations (T1–T21, L1–L17). |
| **64** | [Additional primary sources for the reverse-engineering appendix](../../knowhub_master_blueprint.html#appendix-sources) | You need the supplementary sources used to verify the appendix. |
