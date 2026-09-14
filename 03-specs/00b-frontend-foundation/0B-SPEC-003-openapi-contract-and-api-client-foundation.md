# 0B-SPEC-003 — OpenAPI contract and API client foundation

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository:** `knowhub-frontend`
- **Status:** Approved
- **§81 0B items owned:** `pinned OpenAPI generation pipeline`

## 1. Purpose

Establish how `knowhub-frontend` obtains backend types and how it talks to the
backend, before any product API exists: the governance of the pinned contract
artifact, a reproducible generation pipeline, the freshness property that keeps
generated output honest, the single API access boundary, correlation
propagation, safe error normalization, the streaming boundary and the
server-state foundation.

The milestone constraint this specification is built around: on the **normal
0B path no backend product contract exists**, and 0B is deliberately designed to
complete and prove its pipeline without one. None may be hand-authored by the
frontend to fill the gap. R7 permits one exception — a legitimate
backend-generated artifact that independently becomes available through the
approved workflow before 0B completes — and that exception changes what may sit
in `contracts/`, not what the evidence means: pipeline verification does not
become compatibility verification. 0B therefore proves the **pipeline**, and
never claims to have proved compatibility with `knowhub-backend`. §3.2 and §4
carry that distinction normatively rather than leaving it to reviewer memory.

## 2. Sources

**Blueprint**

- [§30A Frontend codebase — knowhub-frontend](../../../knowhub_master_blueprint.html#target-frontend),
  including §30A.1 (frontend responsibility boundary) and §30A.2 (contract
  update workflow)
- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**: OpenAPI is the shared contract; generate, do not hand-copy;
  frontend CI verifies contract compatibility; the authentication and streaming
  boundary
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Client data/state row
- [§61A.2 Frontend layering](../../../knowhub_master_blueprint.html#frontend-file-spec)
- [§61A.3 UI state classes](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — server-state and URL-state rows
- [§61A.4 Frontend quality gates](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — type-safety, contract and security rows
- [§70.5 API behavior rules](../../../knowhub_master_blueprint.html#api-contracts)
  — the safe error structure, streaming, `/api/v1` versioning, and the rule that
  the backend owns and publishes OpenAPI while the frontend pins and generates
- [§69.1 Error taxonomy](../../../knowhub_master_blueprint.html#failure-semantics)
  — recorded as a forward constraint in §11
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
- [§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph),
  including the parallelization rule and §81.1
- [§84.1](../../../knowhub_master_blueprint.html#coding-harness-instructions),
  [§84.2 and §84.4](../../../knowhub_master_blueprint.html#coding-harness-instructions)

**ADRs** — referenced by number and link only; decision text lives in the
canonical files.

- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
  (primary)
- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
- [ADR-001 — KnowHub is Python-first and API-first](../../02-adrs/ADR-001-python-first-api-first.md)
- [ADR-002 — Application is the primary enterprise scope](../../02-adrs/ADR-002-application-is-primary-scope.md)
  (constrains R49 — Application is the scope shape a cache key must be able to
  carry; no domain scope field is defined here)
- [ADR-022 — Hybrid RBAC/ABAC authorization with default deny](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
  (constrains R49 — a cached result must never be served across an
  authorization scope boundary; no authorization logic is defined here)
- [ADR-021 — Authentication is provider-pluggable](../../02-adrs/ADR-021-provider-pluggable-authentication.md)
  (forward constraint on §8 and §12 only — what the server-side path injects is
  owned by 0B-SPEC-004)

**Approved specifications**

- [0B-SPEC-001](0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md)
  — owns the repository, the §61A.2 layering rule, and the existence of
  `src/lib/api/`, `src/lib/query/`, `contracts/` and `scripts/`
- [M00-SPEC-003](../00-foundation/M00-SPEC-003-backend-observability-baseline.md)
  R8–R10 — the backend correlation contract this specification propagates into

**Dependency on a specification not yet written.** The API boundary receives its
configured backend origin through the configuration interface owned by
0B-SPEC-004. This specification states how the API layer *consumes*
configuration; 0B-SPEC-004 states how configuration is *produced*. That is an
interface obligation, not a circular dependency: nothing here requires
0B-SPEC-004 to exist first.

## 3. Normative requirements — contract artifact governance

### 3.1 The canonical product contract

**R1.** `contracts/` is the governed location for the pinned backend contract
artifact, and holds the governance note recording the rules below. On the normal
0B path that note is its only content. Under the R7 exception it may
additionally hold the legitimate pinned backend-generated artifact. In both
cases `contracts/` remains the governed location, the governance note remains
present, and R2–R4 bind whatever it holds.
*(§30A; 0B-SPEC-001 R10; R7)*

**R2.** A canonical product contract artifact **originates from the backend**.
It is generated and published by `knowhub-backend` through the contract-update
workflow of §30A.2 and is never produced by frontend developers.
*(§30A.2; §0E.8 — "Backend owns the contract"; §70.5 — "The backend repository
owns and publishes OpenAPI"; ADR-017)*

**R3.** A pinned contract artifact is **never hand-edited** in
`knowhub-frontend`, identifies the backend revision or version it was produced
from, and is changed only through an explicit contract-update change that is
reviewable as a contract change in its own right.
*(§30A — "pinned, generated from backend; never hand-edited"; §30A.2; ADR-017)*

**R4.** The pinned backend-generated artifact is the **sole authoritative
product-contract input**: the only input from which production generated API
artifacts are derived. No second product contract exists — no hand-written
OpenAPI document, no shadow contract, and no manually maintained TypeScript
transcription of backend models.
*(§0E.8 — "Generate, do not hand-copy"; §71.3 — "No source-level shared
package"; ADR-017; §84.4)*

**R5.** The synthetic fixture of §4 is permitted **only as a test-only
pipeline-verification input**. Output generated from it is not a product API
contract and not a product generated client, and the fixture can never
substitute for the real backend-generated artifact.
*(§0E.8; ADR-017)*

**R6.** The real product API is versioned under `/api/v1`, and a breaking change
requires a version transition rather than a silent redefinition. This is
recorded as the forward rule governing a future pinned artifact; 0B introduces
no versioned path of its own.
*(§70.5 — "Version APIs under /api/v1; breaking changes require a version
transition plan"; §0E.8)*

### 3.2 The 0B exit rule and the compatibility non-claim

**R7.** At normal 0B exit, **`contracts/knowhub-openapi.json` does not exist.**
The backend publishes no product contract at this point in the §81 sequence, and
none may be manufactured to fill the gap. If, before 0B completes, a legitimate
backend-generated product contract becomes available through the §30A.2
workflow, it may be introduced under R2–R4.
*(§81 — 0B "waits on/stubs versioned API contract"; §30A.2; ADR-017)*

**R8.** **Pipeline verification is not compatibility verification.** Evidence
produced at 0B demonstrates that the generation pipeline is reproducible and
that its failure modes are detected. It demonstrates nothing about
`knowhub-backend`: not compatibility, not the existence of any endpoint, and not
a frontend/backend version pair. No acceptance evidence, gate result or
document in this repository may be presented as showing otherwise.
*(§0E.8 — compatibility is verified against "the backend contract selected for
release"; §70.5; §81.1)*

## 4. Normative requirements — the synthetic contract fixture

**R9.** 0B proves the generation pipeline against a **synthetic, test-only
OpenAPI fixture**. The fixture exists solely to exercise this specification's
pipeline and its failure behaviour, so its contract-governance properties are
owned here; the harness and automated gate that check those properties are owned
by 0B-SPEC-005.
*(§0E.8; §84.1; ADR-017)*

**R10.** The fixture is **non-product**. It describes an obviously synthetic
surface and contains no KnowHub resource vocabulary, no M1 endpoint name, no
`/api/v1` or other product-like path adopted to resemble KnowHub, no
authentication schema and no business schema. Nothing about it may be read as
describing what the backend has, will have, or is being asked to have.
*(§84.1; §84.4; ADR-017)*

**R11.** The fixture is **contained**. It is test-only material; it does not
live under `contracts/` and does not live under `src/`; and it is neither
reachable nor importable from production application code.
*(§30A; §84.3; 0B-SPEC-001 R9–R11)*

**R12.** The fixture **can never become the product contract**. It is not
promoted, copied or renamed into `contracts/`, and a client generated from it is
never shipped as a product client.
*(R4; R5; ADR-017)*

*Implementation guidance (non-normative): the fixture's filesystem path and
filename are an implementation-plan decision, provided R10–R12 hold. A name that
states what it is, in a location that is unmistakably test material, satisfies
them most directly.*

## 5. Normative requirements — the generation pipeline

**R13.** A single deterministic generation entry point produces the TypeScript
contract artifacts from an OpenAPI document. The blueprint refers to it as
`generate:api` in both §0E.8 and §30A.2, and that name is used unless the
implementation plan records a reason to differ.
*(§0E.8 local development; §30A.2)*

**R14.** The pipeline takes its OpenAPI document as an **explicit input**. It
does not discover one implicitly, and pointing it at a different document is a
configuration change rather than a code change — so substituting a genuine
artifact for the fixture, whenever one becomes available, requires no rewrite.
*(§30A.2; §0E.8)*

**R15.** Generation is **deterministic**: the same input and the same committed
generator configuration produce byte-identical output.
*(§0E.8 — "Generation must produce a clean build"; §61A.4 contract row)*

**R16.** Invalid or unparseable contract input **fails clearly and loudly**,
with a message identifying the problem. It never produces partial, empty or
silently stale output.
*(§69.1 — a malformed input is a validation error, failed immediately; §84.2)*

**R17.** Generation requires **no `knowhub-backend` checkout, no running
backend, no network access and no workstation-specific state**. It runs from the
pinned local document alone.
*(§71.3 — Independent CI; §0E.8 — "Frontend developers must be able to build
using a pinned OpenAPI contract without importing the backend repository as a
source dependency"; ADR-016)*

**R18.** Generated output is written to a location distinct from handwritten
frontend source, so that derived and authored code are never interleaved.
*(§30A — `src/lib/api/generated/`; §0E.8)*

*Implementation guidance (non-normative): §0E.8 names **openapi-typescript +
openapi-fetch** and permits "an equivalent approved generator". The named pair
is therefore the default; selecting the equivalent, and pinning versions, are
implementation-plan decisions. This specification fixes the outcome in R13–R18,
not the tool.*

## 6. Normative requirements — generated-code governance

**R19.** Generated artifacts are **derived, not authoritative**. The OpenAPI
input selected for a generation run is authoritative for that run's output, and
the generated tree is a reproducible projection of it. For **product** generated
artifacts that authoritative input is the pinned backend-generated contract
under R4. The synthetic fixture of §4 is a test input only and never becomes an
authoritative product contract.
*(§0E.8; ADR-017; R4, R5)*

**R20.** Generated artifacts are **never hand-edited**. A change to generated
output is made by changing the input or the generator configuration and
regenerating.
*(§0E.8 — "Generate, do not hand-copy"; §30A)*

**R21.** Handwritten code — the API boundary of §7, view-models and hooks —
lives outside the generated tree and is never mixed into it.
*(§30A — "Frontend UI-only view models stay separate"; §61A.2)*

**R22.** Backend representations are **not re-declared by hand**. A handwritten
type that duplicates a generated contract type, rather than composing or
narrowing it for presentation, is not introduced.
*(§0E.8; §71.3 — no manually maintained TypeScript copies of Pydantic models;
§84.1 — use typed contracts; ADR-017)*

## 7. Normative requirements — the freshness property

**R23.** **Freshness.** Given the pinned input and the committed generator
configuration, regenerating produces no unexpected diff. Generated output that
does not match what the input and configuration produce is stale, and stale
output is a defect rather than a tolerated state.
*(§0E.8 — "generated-client freshness" as a required frontend CI gate; §61A.4
contract row)*

**R24.** The eventual freshness gate detects at least: stale generated output;
hand-edited generated output; generator or configuration drift that changes
output; and invalid contract input. This specification owns the property; the
gate's wiring is owned by 0B-SPEC-005.
*(§0E.8; §61A.4; §84.2)*

**R25.** The synthetic fixture is the **mandatory** freshness demonstration,
so that the property is proved independently of whether any real backend
contract exists. On the normal 0B path it is the only demonstration available.
Under the R7 exception the genuine pinned artifact **may additionally** be
subjected to the same regeneration and freshness property.

Every such result — fixture or genuine artifact — is **freshness evidence
only**. It is reported as pipeline evidence under R8 and is not evidence about
`knowhub-backend`: it claims no endpoint compatibility and no validated
frontend/backend version pair.
*(R8; R7; §0E.8; §81)*

## 8. Normative requirements — the single API access boundary

**R26.** All access from `knowhub-frontend` to the KnowHub backend API crosses
**one approved boundary**, composed of the generated contract artifacts beneath
a handwritten API layer.
*(§61A.2; §30A — `src/lib/api/client.ts`, `server-client.ts`; §0E.8)*

**R27.** The boundary distinguishes a **server-side path** and a **browser-safe
path**. The server-side path is where authenticated backend access is later
performed; the browser-safe path never carries backend bearer credentials.
This specification fixes the separation; the authentication and session
mechanics injected into the server-side path are owned by 0B-SPEC-004, and no
token, session or credential handling is defined here.
*(§0E.8 authentication and streaming boundary; §30A.1; §0E.15; §61A.2)*

**R28.** Consumers above the boundary — server-state hooks, view-models,
feature components and routes — obtain backend data **through** it and never
beneath it.
*(§61A.2; §30A.1)*

**R29.** No page or feature component implements an **ad-hoc direct HTTP
integration with the KnowHub backend API**: constructing its own backend
request, applying its own headers, or parsing its own backend response. This
constrains direct integration with the FastAPI backend. It does not constrain
legitimate Next.js-internal requests, route handlers, server actions, asset
requests or third-party browser requests the architecture requires.
*(§61A.2; §30A.1; §0E.8)*

**R30.** The boundary is the **only place** that converts a backend transport
outcome into an application-level result: components never inspect an HTTP
status or parse a backend error body themselves.
*(§61A.2; §61A.4 security row; R38–R41)*

**R31.** 0B defines **no product operation** at the boundary. The boundary is
built and exercised generically over whatever operations its input describes.
*(§81 — business APIs belong to M1 and later; §84.1)*

## 9. Normative requirements — the configuration interface

**R32.** The API boundary obtains the backend origin it targets as a
**validated, configured value supplied through the frontend configuration
interface owned by 0B-SPEC-004**. It is never hard-coded.
*(§77.1 — configuration by domain; §0E.8; §61A.2)*

**R33.** The boundary does **not read environment variables directly** and does
not parse, validate or default configuration itself. It consumes a value already
validated by the configuration layer. This specification names no environment
variable, defines no server-only or browser-exposed split, and defines no secret
handling — all of which are owned by 0B-SPEC-004.
*(§77.2 — secrets are references, never ordinary settings values; §0E.16;
§61A.4 security row)*

## 10. Normative requirements — correlation propagation

**R34.** Every outbound request the boundary makes to the backend carries a
**correlation identifier**, so that a request observed in the browser can be
located in backend telemetry.
*(M00-SPEC-003 R8 and R10 — an inbound identifier is honoured and the
identifier is returned to the caller; §84.1 — correlation identifiers flow
across boundaries; §65.4)*

**R35.** An identifier already present in the enclosing request context is
**preserved and forwarded only when it conforms** to the authoritative backend
correlation binding. A non-conforming value is **never forwarded**: the boundary
does not silently propagate an invalid inbound identifier. Where no identifier
is present, or where the present one does not conform, the boundary originates a
conforming identifier instead.
*(M00-SPEC-003 R8 — an inbound identifier is honoured, one is generated when
absent; M00-SPEC-003 R9 — a caller-supplied value is accepted only when it
conforms, and a non-conforming value is replaced rather than propagated
unvalidated)*

**R36.** The propagation mechanism is **parameterized over a correlation
binding** — the header and the identifier representation — and fixes neither.
M00-SPEC-003 R9 leaves the concrete representation and its generator to the
implementation plan, so no authoritative product binding exists to adopt.

**At 0B the product binding is therefore unbound.** It becomes bound only when
an authoritative backend-published contract or documented transport binding
exists, and it is resolved from that source. Backend implementation source may
be inspected as a **consistency check only**; it is never the normative source,
and no backend module, package or symbol name is depended upon here.

0B may prove the mechanism using a **clearly synthetic, test-only binding**.
That binding is not a product correlation contract, establishes no future header
name and no future identifier representation, and is subject to the containment
rules of R11 and R12.
*(M00-SPEC-003 R9 and its non-normative guidance; §71.3; ADR-016; R11, R12)*

**R37.** Correlation propagation introduces **no browser telemetry system**: no
tracing SDK, no vendor integration and no metrics framework. It also never
causes session, token or credential material to be exposed to browser code.
*(§0E.13.9 — no secrets in telemetry; §84.1; §0E.16)*

## 11. Normative requirements — safe error normalization

**R38.** Failures reaching the boundary are converted into **one safe internal
representation** used by every consumer.
*(§61A.2; §0E.8)*

**R39.** That representation carries the fields §70.5 fixes for a KnowHub safe
error — **code, message, correlation identifier, retryable flag and optional
remediation hint** — together with the HTTP status class where it is meaningful
to a caller. It is a frontend-internal presentation-facing type mapped **from**
whatever the pinned contract exposes; this specification defines **no backend
error schema**, which the backend owns.
*(§70.5 — "Errors use a common safe structure: code, message, correlation ID,
retryable flag and optional remediation hint"; §70.2 — `SafeError`; ADR-017)*

**R40.** Retryability is represented **only when the contract states it**. The
frontend does not infer or invent retry semantics; §69.1's taxonomy is backend
policy.
*(§70.5; §69.1)*

**R41.** **Nothing unsafe survives normalization.** Raw backend exception text,
stack traces, sensitive implementation or diagnostic detail, sensitive or
internal configuration, credentials, secrets and tokens are never carried into
the representation and so cannot reach a rendering surface.
*(§61A.4 security row; §0E.13.9; §0E.17.8)*

**R42.** The normalized representation is **produced** here and **rendered** by
0B-SPEC-002. This specification defines no error presentation, and 0B-SPEC-002
defines no error shape.
*(§61A.3; §0E.17.6)*

## 12. Normative requirements — the streaming boundary

**R43.** Authenticated streaming from the backend crosses the **same approved
boundary**. A feature component never establishes a backend stream directly.
*(§0E.8 — the BFF proxies the authenticated backend stream to the browser as a
same-origin readable stream; §30A — `src/lib/api/streaming.ts`; §61A.2)*

**R44.** Stream establishment and failure follow the same rules as any other
backend access: configured origin (R32), correlation propagation (R34), safe
normalization of a failure (R38–R41), and bearer credentials remaining
server-side under the mechanics owned by 0B-SPEC-004.
*(§0E.8; §61A.4 security row)*

**R45.** 0B defines **no streaming protocol content**: no SSE or websocket event
schema, no run event model, no session or resumption token format and no product
event name. §70.5's direction — SSE or equivalent server-supported event
streaming with resumable run and session identifiers — is recorded as the shape
the future contract is expected to take, not as a contract defined here. No
stream implementation is built at 0B.
*(§70.5; §0D.3; §81; §84.1)*

## 13. Normative requirements — the server-state foundation

**R46.** Server state is held by **TanStack Query**, with local React state for
transient interaction state.
*(§0E.1 Client data/state row; §30A.1; §61A.3 server-state row)*

**R47.** One **query-client configuration** exists for the application, composed
at the single provider-composition point 0B-SPEC-001 establishes.
Application-level query defaults are defined centrally there, and no feature or
query module defines its own query-client default policy. Exact default values
are implementation-plan decisions where the blueprint does not mandate them.
*(§0E.1; §61A.3; 0B-SPEC-001 R13)*

**R48.** One **cache-key convention** governs every query key; keys are
constructed through it rather than assembled ad hoc at call sites.
*(§61A.3 — "cache keys always include application/source scope where
relevant"; §0E.1)*

**R49.** A cache key **carries every authoritative scope dimension** needed to
prevent a cached result being served across a scope boundary. The convention
must be able to express such dimensions; 0B introduces none of them, because the
capabilities that define scope do not yet exist.
*(§61A.3; §49 — permission-aware evidence; ADR-002; ADR-022; §84.1)*

**R50.** Backend server state is **not duplicated** into a second client store.
Where a value is server state, the query layer holds it.
*(§30A.1 — "Do not create a second copy of backend workflow state in the
browser"; §0E.1)*

**R51.** Cache invalidation, when product mutations exist, flows **through the
query layer** rather than through ad-hoc cross-component signalling.
*(§61A.3; §0E.1 — "optimistic mutations only where safe")*

**R52.** 0B defines **no product query and no product cache key**. No
application, user, source, Ask, graph or other domain query is introduced.
*(§81; §84.1)*

## 14. Forward constraints recorded, not implemented at 0B-SPEC-003

- **The first real contract — normally at M1.** M1, delivering a stable
  backend API surface, is the expected first genuine product-contract point; the
  R7 exception may cause the first legitimate pin to occur during 0B instead.
  Whenever it happens the sequence and its governance are the same: the backend
  generates and publishes the artifact → the frontend pins it under `contracts/`
  through the §30A.2 contract-update change → generation runs from it → the
  generated diff is reviewed as a contract change. R2–R4 govern it regardless of
  timing. A compatibility gate becomes meaningful only once the real backend and
  frontend evidence it requires exists; **merely receiving the artifact proves no
  compatibility**. Pinning it changes the pipeline's input, not the pipeline.
  *(§30A.2; §0E.8; ADR-017; R7)*
- **The compatibility gate is not this specification's.** §0E.8 and §70.5 place
  breaking-change detection before backend release, and frontend compatibility
  tests against the backend contract selected for release. That gate spans both
  repositories and a deployed or published artifact; it is not satisfied by
  anything 0B produces. *(§0E.8; §70.5; §81.1)*
- **Backend error policy.** §69.1's taxonomy — validation, authentication,
  transient dependency, rate limiting, content, model, integrity, security —
  and its retry and backoff defaults are backend policy. The frontend renders
  what the contract exposes and infers none of it. *(§69.1; §70.5)*
- **List and mutation conventions.** §70.5's cursor pagination for high-volume
  lists and stable resource and run identifiers on mutations presuppose APIs
  that do not exist at 0B; the query conventions here must accommodate them
  without pre-declaring them. *(§70.5; §81)*
- **Configuration production.** The configuration model, its validation, the
  server-only versus browser-exposed split and secret containment are owned by
  **0B-SPEC-004**. *(§77.1; §77.2)*
- **Session and credential mechanics.** What the server-side path injects, and
  every cookie, token, CSRF and header concern, are owned by **0B-SPEC-004**.
  *(§0E.13; §0E.15; ADR-021)*
- **Gates, harnesses and CI.** The contract test layer, the freshness gate's
  wiring and every other gate are owned by **0B-SPEC-005**. Verification by
  0B-SPEC-005 does not transfer normative ownership of any requirement here.
  *(§0E.8; §61A.4)*

## 15. Acceptance criteria

| ID | Criterion |
|---|---|
| **0B-AC-040** | **Normal path:** `contracts/knowhub-openapi.json` is absent at 0B exit and `contracts/` holds only the governance note, which records the R2–R4 rules and the `/api/v1` versioning rule. **R7 exception path:** if the file exists, it is demonstrated to have originated from `knowhub-backend`, to have arrived through the approved §30A.2 contract-update workflow, to identify its backend revision or version, and not to have been hand-authored or hand-edited by frontend developers; it is the sole authoritative product-contract input and no shadow or second product contract accompanies it. **In either path:** 0B has itself manufactured no product API contract, no frontend-authored product API path or schema has been introduced, and no pipeline evidence is represented as backend compatibility evidence. *(R1, R2, R3, R4, R6, R7, R8)* |
| **0B-AC-041** | No document, gate result or acceptance record in the repository states or implies that 0B evidence demonstrates compatibility with `knowhub-backend`, the existence of any backend endpoint, or a frontend/backend version pair. *(R8, R25)* |
| **0B-AC-042** | The fixture used to exercise the pipeline contains no KnowHub resource vocabulary, no M1 endpoint name, no `/api/v1` or other product-like path, no authentication schema and no business schema. *(R9, R10)* |
| **0B-AC-043** | The fixture is test-only: it resides outside `contracts/` and outside `src/`, and no production application module can import or reach it. *(R11)* |
| **0B-AC-044** | No client generated from the fixture is shipped as a product client, and the fixture has not been promoted, copied or renamed into `contracts/`. *(R5, R12)* |
| **0B-AC-045** | The generation entry point accepts an explicitly specified local OpenAPI document, and pointing it at a different document is a configuration change requiring no code change. *(R13, R14)* |
| **0B-AC-046** | Running generation twice from the same input and committed configuration produces byte-identical output. *(R15)* |
| **0B-AC-047** | Generation from a malformed or unparseable document fails with a message identifying the problem, and leaves no partial, empty or silently stale output behind. *(R16)* |
| **0B-AC-048** | Generation completes with only `knowhub-frontend` cloned, with no running backend, with network access unavailable, and with no workstation-specific state. *(R17)* |
| **0B-AC-049** | Generated output is reproducibly derived from the OpenAPI input selected for the run; it is not treated as an independent authoritative contract, and altering it is not a source-of-truth operation — a change originates in the input or the generator configuration and is realised by regenerating. Generated output occupies a location distinct from handwritten source; no handwritten module lives inside the generated tree; and no handwritten type re-declares a generated contract type rather than composing or narrowing it. *(R18, R19, R21, R22)* |
| **0B-AC-050** | Each drift case R24 enumerates is detected rather than silently absorbed — generated output stale with respect to its input, generated output that has been hand-edited, a generator or configuration change that alters output, and invalid contract input — and regeneration restores the expected state. Invalid input fails the gate rather than being absorbed into a stale-but-passing state; AC-047 separately verifies R16's generation-time failure behaviour. *(R20, R23, R24)* |
| **0B-AC-051** | Exactly one approved KnowHub-backend API boundary exists. Server-state hooks, view-models, feature components and routes each obtain backend data through it: none constructs an ad-hoc backend request, none applies backend transport headers of its own, and none inspects a backend HTTP status or parses a backend response or error body. Conversion of a backend transport outcome into an application-level result occurs only at that boundary. Unaffected: Next.js-internal requests, asset requests, unrelated third-party browser requests, and route-handler or server-action mechanics that form part of the approved boundary — but no route handler or server action constitutes a **second** independent backend integration boundary. *(R26, R28, R29, R30)* |
| **0B-AC-052** | The boundary exposes distinct server-side and browser-safe paths, the browser-safe path carries no backend bearer credential, and no product operation is defined at the boundary. *(R27, R31)* |
| **0B-AC-053** | The boundary obtains its backend origin from the configuration interface; no backend origin is hard-coded, and the boundary reads no environment variable and performs no configuration parsing, validation or defaulting of its own. *(R32, R33)* |
| **0B-AC-054** | Every outbound backend request carries a correlation identifier. Exercised against the synthetic test-only binding: a conforming identifier already present in the request context is preserved and forwarded; an absent identifier is originated; and a non-conforming inbound identifier is not propagated but replaced by a conforming one. The mechanism is parameterized over the binding and fixes no real KnowHub header name and no product identifier representation, the product binding remaining unbound at 0B. *(R34, R35, R36)* |
| **0B-AC-055** | No browser tracing SDK, vendor telemetry integration or metrics framework is introduced, and no session, token or credential material is exposed to browser code in order to propagate correlation. *(R37)* |
| **0B-AC-056** | Exactly one normalized safe representation exists; given a backend failure the boundary produces it carrying the §70.5 field set, together with the HTTP status class where it is meaningful to a caller; retryability is populated only from authoritative contract information and never inferred; given a failure bearing raw exception text, a stack trace, sensitive internal configuration, a credential, a secret or a token, none of that material survives into the normalized representation; and this specification introduces no error-rendering or presentation logic, which 0B-SPEC-002 owns. *(R38, R39, R40, R41, R42)* |
| **0B-AC-057** | No streaming implementation exists; no SSE or websocket event schema, run event model, resumption token format or product event name is defined; no feature or component defines a competing direct backend-stream transport; no second streaming boundary has been introduced; and no stream-specific configuration, correlation or error-normalization path exists alongside the approved boundary. R43 and R44 bind the streaming path when it is built, but **0B provides no runtime evidence for a stream that does not yet exist**; their runtime verification is deferred to the capability that introduces streaming. *(R43, R44, R45 — R43 and R44 verified only in what 0B can observe, their runtime behaviour not proven at 0B)* |
| **0B-AC-058** | TanStack Query is the server-state foundation, and transient interaction state remains local React state rather than a second server-state store; one query-client configuration exists at the single provider-composition point, application-level query defaults are defined centrally there, and no feature or query module defines its own query-client default policy; every query key is built through the one cache-key convention, which can express an authoritative scope dimension while none is defined at 0B; no product query and no product cache key exists; backend server state is not duplicated into another client store; and invalidation is performed through the query layer rather than an ad-hoc cross-component mechanism, demonstrated with test-only synthetic state where an executable demonstration is needed. *(R46, R47, R48, R49, R50, R51, R52)* |

**Verification dependency.** 0B-AC-040 through 0B-AC-044, 0B-AC-049, 0B-AC-052,
0B-AC-055 and 0B-AC-057 are satisfied by this specification's own implementation
and are decidable by inspection, 0B-AC-041 being confirmed at milestone review.
0B-AC-045 through 0B-AC-048, 0B-AC-050, 0B-AC-051, 0B-AC-053, 0B-AC-054,
0B-AC-056 and 0B-AC-058 need an executable contract or component test layer.
That layer and its CI wiring are owned by **0B-SPEC-005**; the requirements they
check remain owned here. This is a verification dependency at milestone level,
not a transfer of ownership.

## 16. Deferred

| Deferred from 0B-SPEC-003 | Owner |
|---|---|
| First genuine product-contract pin | Normally M1, through the §30A.2 workflow; may occur during 0B only via the R7 legitimate-backend-artifact exception |
| Backend OpenAPI publication, breaking-change detection before backend release | `knowhub-backend`, M1 |
| Cross-repository compatibility tests and the recorded frontend/backend version pair | Post-0B |
| Every product API: application, user and role, source, Ask, evidence, graph, traceability, impact, Jira and document operations | M1 and later |
| Product queries, product cache keys and their scope dimensions | M1 and later |
| Streaming protocol, run event model and resumable session semantics, and the runtime verification of R43 and R44 | The capability that introduces streaming, M1 and later |
| Cursor pagination and mutation-identifier conventions | M1 and later |
| Configuration model, validation, `NEXT_PUBLIC` policy and secret containment | 0B-SPEC-004 |
| Authentication, session, token, CSRF and security-header mechanics | 0B-SPEC-004 |
| Error rendering, support-identifier display and retry affordances | 0B-SPEC-002 |
| Contract test layer, freshness gate wiring, dependency scanning and CI | 0B-SPEC-005 |
| Browser telemetry SDK, vendor or UX-metrics framework | Deferred; a future ADR when a concrete requirement and technology choice exist |
