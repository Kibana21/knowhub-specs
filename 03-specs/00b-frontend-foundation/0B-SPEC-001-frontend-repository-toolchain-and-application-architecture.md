# 0B-SPEC-001 — Frontend repository, toolchain and application architecture

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository:** `knowhub-frontend`
- **Status:** Approved
- **§81 0B items owned:** `Next.js/TypeScript`

## 1. Purpose

Define the shape of the `knowhub-frontend` repository at 0B: how dependencies
are managed, what the language and runtime baseline is, which directories
exist and what each layer may depend on, how the application is structured
under the App Router, which quality tooling gates the code, and what
independence from `knowhub-backend` means in practice.

This specification establishes structure. It delivers no product capability
and no visual, contract, session or pipeline behaviour.

## 2. Sources

**Blueprint**

- [§30A Frontend codebase — knowhub-frontend](../../../knowhub_master_blueprint.html#target-frontend),
  including §30A.1 (frontend responsibility boundary)
- [§61A Frontend file-level target specification](../../../knowhub_master_blueprint.html#frontend-file-spec),
  including §61A.2 (frontend layering) and §61A.4 (frontend quality gates)
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Web application, Design system, Client data/state and Testing rows
- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**
- [§0E.15 Frontend authentication, authorization & administration implementation](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, for the server-only placement rule in §4.3 only
- [§0E.16 Security verification / penetration-test readiness](../../../knowhub_master_blueprint.html#runtime-architecture)
  — supply-chain row
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
- [§84.1 Mandatory working style](../../../knowhub_master_blueprint.html#coding-harness-instructions)
  — vertical slices over empty interfaces; typed contracts
- [§84.3 Repository hygiene expected from the harness](../../../knowhub_master_blueprint.html#coding-harness-instructions)
- [§65.3 Security, privacy and compliance constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
- [§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph)
  and §81.1

**ADRs** — referenced by number and link only; their decision text lives in
the canonical files.

- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
  (integration boundary; mechanics owned by 0B-SPEC-003)
- [ADR-001 — KnowHub is Python-first and API-first](../../02-adrs/ADR-001-python-first-api-first.md)
- [ADR-022 — Hybrid RBAC/ABAC authorization with default deny](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
  (constrains R20 only; authorization mechanism is owned by 0B-SPEC-004)
- [ADR-026 — ADR ownership, location, numbering and register](../../02-adrs/ADR-026-adr-ownership-and-location.md)
  (governance context for R26)

## 3. Normative requirements — dependency management and language baseline

**R1.** KnowHub frontend application and runtime **logic** is implemented in
TypeScript with React and Next.js. Stylesheets, static assets, configuration
files, JSON artifacts and documentation are not application logic and are
unaffected by this requirement. No Python and no backend application
implementation is introduced into `knowhub-frontend`, and no copy or
transliteration of a backend model, domain rule or service implementation
exists there.
*(§30A; §0E.8; §84.1; §84.3; ADR-016; ADR-001)*

**R2.** Dependencies are declared in `package.json` and resolved with
**pnpm**. A committed pnpm lockfile is the single reproducible resolution for
the repository. No second dependency path — npm, yarn, a vendored tree or a
globally installed package — may be required to obtain a working environment.
*(§30A; §84.3; §0E.8 local development)*

**R3.** Installation in CI and in a clean checkout resolves strictly from the
committed lockfile. An install that would change the resolution must fail
rather than silently drift.
*(§0E.16 supply-chain row — pinned lockfiles)*

**R4.** The committed lockfile is always current for the committed manifest,
and nothing required to install, check, test or build the repository exists
outside that declared set. How currency is assured — commit discipline, a
frozen-install check, or both — is an implementation-plan decision.
*(§0E.16 supply-chain row — pinned lockfiles; §84.3)*

**R5.** The supported Node baseline is declared in the repository, and the
same baseline is used by local tooling and by CI. A contributor and CI must
not be able to run on different major runtimes without that being visible.
*(§0E.1 Web application row — Next.js/React fixes the runtime family; §0E.16
— reproducible, pinned builds; §0E.8 — frontend CI builds the repository on
its own. No blueprint section states a Node version; the requirement is that a
baseline be declared and shared, not which one.)*

**R6.** The Next.js and React baselines are declared and pinned by the
lockfile.
*(§0E.1 Web application row)*

**R7.** TypeScript runs in strict mode. KnowHub-owned application source
contains no blanket `any` escape hatch. Suppressions are narrow, local and
justified; blanket suppression of a file, module or rule class is not an
acceptable way to pass the gate.
*(§61A.4 type-safety row — "no blanket `any`"; §84.1 — use typed contracts)*

*Implementation guidance (non-normative): exact Node, Next.js, React and
TypeScript versions are not fixed by this specification. The implementation
plan selects them against current releases at the time it is written, and may
change them without amending this specification so long as R5–R7 hold. No
blueprint section or ADR requires a specific version.*

## 4. Normative requirements — source layout, application architecture and module boundaries

### 4.1 Target layout versus 0B materialization

**R8.** The directory layout in
[§30A](../../../knowhub_master_blueprint.html#target-frontend) is the **target
architecture** for `knowhub-frontend`. It is not the 0B deliverable.

**R9.** 0B materializes only directories that carry real 0B content. An empty
directory must not be created to mirror the future tree. In particular 0B
creates no route or component directory for enquiry, evidence, architecture,
graph, traceability, impact, sources, governance or administration; no product
route; and **no `src/lib/telemetry/`**, because browser telemetry
implementation is deferred and no 0B content belongs in it.
*(§84.1 — deliver working vertical slices, not a large set of empty
interfaces)*

### 4.2 0B directory inventory

**R10.** The directories materialized at 0B are exactly those below. This
specification fixes *which* directories exist and what each is for; the
specification named in the third column fixes *what goes in them*.

| Directory | 0B responsibility | Contents specified by |
|---|---|---|
| `src/app/` | App Router routes and layouts. The root layout and the provider-composition point exist at 0B. | This specification (§4.3); auth shell routes by 0B-SPEC-004 |
| `src/components/` | React components. | 0B-SPEC-002 (primitives, shell); 0B-SPEC-004 (access components) |
| `src/lib/api/` | Contract-generated types and the single API access boundary. | 0B-SPEC-003 |
| `src/lib/auth/` | Server-only session, route-guard and permission boundary. | 0B-SPEC-004 |
| `src/lib/query/` | Server-state client configuration and cache-key conventions. | 0B-SPEC-003 |
| `src/styles/` | Design tokens and global styles. | 0B-SPEC-002 |
| `contracts/` | The governed location for a pinned backend contract artifact. Its 0B content is the governance note recording the rule that binds future contents; no contract artifact is materialized at 0B. | 0B-SPEC-003 |
| `scripts/` | The contract generation entry point. | 0B-SPEC-003 |
| `tests/` | Test layers that have real 0B behaviour to test. | 0B-SPEC-005 |
| `docs/adr/` | Frontend-scoped ADRs. | §84.3; ADR-026 |

**R11.** A directory is added to `knowhub-frontend` only when a milestone
delivers content that belongs in it. Adding one is a structural change
justified by that milestone's specification, not by the §30A target diagram.

*Implementation guidance (non-normative): `docs/ux/`, `public/`,
`src/hooks/`, `src/view-models/` and `src/lib/format/` appear in §30A and
§84.3 and are expected later. Each is created by the change that introduces
its first real content, under R9 and R11, without amending this
specification.*

### 4.3 Application architecture

**R12.** The application uses the Next.js **App Router** and has exactly one
root layout.
*(§30A — `src/app/layout.tsx`; §0E.2 — routes around stable application IDs)*

**R13.** Providers are composed at exactly one composition point reachable
from the root layout. Providers are not assembled ad hoc per route.
*(§61A.2 — a single layered path from routes downward)*

**R14.** The repository states one explicit server/client component policy,
including which of the two is the default, and applies it consistently. A
component that crosses the boundary does so as a deliberate, visible choice
rather than by accident. This specification requires that the policy exist, be
stated and be applied; it does not choose the default.
*(§30A.1 — Next.js route handlers/server actions implement the minimal
boundary; §61A.2 — dependencies flow downward through one layered path)*

*Implementation guidance (non-normative): server components as the default,
with client components introduced at the boundary where interactivity requires
them, is the posture that follows most directly from R15 and from §61A.4's
performance row. The blueprint does not mandate it, so the choice and its
enforcement belong to the implementation plan.*

**R15.** Authentication, session, token and identity-provider protocol logic
is **server-only** by placement: no module carrying it may be reachable from a
client component bundle. This requirement fixes *where* such code may live;
what it does is owned by 0B-SPEC-004.
*(§0E.15 — "No raw token APIs in client components"; §0E.13.7)*

**R16.** Route organization must accommodate future product modules without
restructuring the application. 0B introduces no product route.
*(§61A.1 — the route map is delivered by M1 and later; §81)*

### 4.4 Frontend layering and module boundaries

**R17.** The layering of
[§61A.2](../../../knowhub_master_blueprint.html#frontend-file-spec) is
normative for `knowhub-frontend` from 0B onward, and is recorded here so that
the rule exists before most of the layers it governs do:

```
app/ routes + page composition
  → feature components
    → server-state hooks / UI view-models
      → lib/api generated contract + server client
        → BFF/session boundary
          → HTTPS / authenticated streaming
            → FastAPI backend
```

Dependencies flow downward only. A layer does not reach past the layer beneath
it, and no layer depends upward.

**R18.** Page and feature components do not implement backend domain semantics
directly. They compose the layers beneath them.
*(§61A.2; §30A.1)*

**R19.** View-models are presentation types. They may combine several backend
representations for display, but they must remain presentation types and must
not become a second domain model.
*(§61A.2 — "UI view models … must remain presentation types")*

**R20.** The frontend must not reimplement authorization logic, freshness
scoring, impact semantics or evidence-resolution semantics in TypeScript. The
backend is authoritative for domain and security truth; the frontend uses
backend-provided identifiers, authority and freshness exactly as given.
*(§61A.2; §30A.1; §71.3 — "No frontend privilege";
[ADR-022](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md))*

**R21.** A violation of R15 or R17–R20 must **fail CI automatically** once
enforcement is wired. Automated enforcement is the normative requirement; no
specific enforcement product is part of the architectural contract. The gate
itself is specified by 0B-SPEC-005.
*(§61A.4; §84.2)*

**R22.** Because 0B materializes only part of the target tree, boundary
enforcement at 0B is necessarily partial in coverage. The mechanism must
nonetheless be wired, gating and demonstrably able to fail on a violation, so
that later milestones inherit a working gate rather than build one.
*(§81.1 — preceding contracts remain stable)*

*Implementation guidance (non-normative): lint-based import restrictions,
a module-boundary plugin, TypeScript project references or a dedicated
boundary tool can each satisfy R21. The tool, its configuration and its
command wiring belong to the implementation plan and may change without
amending this specification, so long as R21 and R22 hold.*

## 5. Normative requirements — quality tooling and repository hygiene

**R23.** Lint enforcement uses **ESLint**, and violations fail CI.
*(§0E.8 frontend CI gate list — ESLint)*

**R24.** Source formatting is mechanically determined and mechanically
checkable: a formatting deviation is detectable without human judgement, and
formatting is never a matter of review opinion. Whether this is enforced by
the lint tool or by a dedicated formatter is an implementation-plan decision.
*(§0E.8 names a lint gate and no formatter; §61A.4 and §84.2 require gates to
be mechanical. The normative content here is the outcome — deterministic,
checkable formatting — not any tool.)*

**R25.** Static type checking runs the TypeScript compiler in strict mode, and
a type error in KnowHub-owned application source fails CI. Strictness must not
be configured so as to force semantically empty annotations on third-party
framework surfaces where the framework supplies no usable types.
*(§61A.4 type-safety row; §0E.8 — strict TypeScript; §84.1)*

**R26.** The repository contains `docs/adr/` for frontend-scoped ADRs.
`docs/ux/` is created by the change that records the first real UX decision,
under R9 and R11.
*(§84.3;
[ADR-026](../../02-adrs/ADR-026-adr-ownership-and-location.md))*

**R27.** The repository contains a `README.md` that describes what the
repository is and how to obtain a working local environment through the
documented workflow.
*(§84.3; §0E.8 local development)*

**R28.** No generated secret, local token, source-system credential,
production data snapshot or model API key is committed. This is enforced as a
CI gate specified by 0B-SPEC-005.
*(§84.3; §65.3)*

**R29.** `knowhub-frontend` must install, lint, format-check, type-check, test
and build without cloning `knowhub-backend`, without importing backend source,
and without access to a workstation-specific service.
*(§71.3 — Independent CI; §0E.8;
[ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md))*

**R30.** No source-level sharing mechanism between the application
repositories may be introduced: no git submodule, no shared TypeScript or
Python model package, no unpublished shared DTO source dependency and no
import from a backend checkout. The versioned OpenAPI artifact is the only
integration boundary; its mechanics are owned by 0B-SPEC-003.
*(§71.3 — "No source-level shared package"; §0E.8;
[ADR-017](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md); §84.4)*

## 6. Forward constraints recorded, not implemented at 0B-SPEC-001

- **Contract authority and the `contracts/` directory.** The backend owns the
  HTTP contract; the frontend generates its client from a pinned artifact and
  never hand-edits one. The directory exists under R10; its governance, the
  generation pipeline and the freshness gate are owned by **0B-SPEC-003**.
  *(§0E.8; ADR-017)*
- **Configuration and secret containment in the browser bundle.** R28 covers
  committed artifacts only. The configuration model — server-only versus
  browser-exposed values, validation, and the rule that no secret reaches a
  browser bundle or source map — is owned by **0B-SPEC-004**.
  *(§77.1; §77.2; §0E.16)*
- **Design language and the application shell.** R10 fixes that
  `src/components/` and `src/styles/` exist. Tokens, primitives, the shell,
  UI state classes and accessibility are owned by **0B-SPEC-002**.
  *(§0E.17; ADR-024)*
- **Session, route protection and authorization-aware UI.** R15 fixes
  server-only placement. The session boundary, provider-neutral abstraction,
  route guards and permission mechanism are owned by **0B-SPEC-004**.
  *(§0E.13; §0E.15; ADR-021)*
- **CI wiring, test layers, supply-chain scanning, production build and
  container.** R21, R23, R24, R25 and R28 state required outcomes. The gates
  that enforce them, the test layers and the production artifact are owned by
  **0B-SPEC-005**. Verification by 0B-SPEC-005 does not transfer normative
  ownership of any requirement in this specification.
  *(§0E.8; §61A.4)*

## 7. Acceptance criteria

| ID | Criterion |
|---|---|
| **0B-AC-001** | A clean checkout reaches a working environment through the documented pnpm workflow alone, using the committed lockfile. No alternative dependency path is required, and an install that would change the resolution fails. *(R2, R3)* |
| **0B-AC-002** | The committed manifest and the committed lockfile are consistent; an inconsistent or stale lockfile is detected mechanically rather than by review; and nothing outside the declared set is required to install, check, test or build. *(R4)* |
| **0B-AC-003** | The Node, Next.js and React baselines are declared in the repository, and local tooling and CI run on the same declared Node baseline. *(R5, R6)* |
| **0B-AC-004** | TypeScript runs in strict mode and KnowHub-owned application source contains no blanket `any`; any suppression present is narrow, local and carries a justification. *(R7, R25)* |
| **0B-AC-005** | The materialized directory set matches R10. No directory exists for a capability owned by a later milestone, no empty directory mirrors the target tree, and `src/lib/telemetry/` does not exist. *(R8, R9, R10, R11)* |
| **0B-AC-006** | The application uses the App Router, has exactly one root layout and one provider-composition point, boots, and serves that root layout. *(R12, R13)* |
| **0B-AC-007** | The server/client component policy is stated in the repository and applied; and no module carrying authentication, session, token or identity-provider protocol logic is reachable from a client component bundle. *(R14, R15)* |
| **0B-AC-008** | No product route exists, and a new product module can be added as a sibling route group without altering the root layout or the provider-composition point. *(R13, R16)* |
| **0B-AC-009** | A deliberately introduced violation of the §61A.2 layering, or of the server-only placement rule, causes the boundary gate to fail; removing it restores a green run. *(R17, R21, R22)* |
| **0B-AC-010** | No frontend source implements authorization, freshness scoring, impact semantics or evidence resolution, and no view-model has become a domain model. *(R18, R19, R20)* |
| **0B-AC-011** | A lint violation fails CI, and a formatting deviation is detected mechanically rather than by review. *(R23, R24)* |
| **0B-AC-012** | `docs/adr/` exists, `README.md` describes the repository and its documented local workflow, and the repository contains no committed secret, credential, token, data snapshot or model API key. *(R26, R27, R28)* |
| **0B-AC-013** | The full local validation sequence — install, lint, format-check, type-check, test, build — completes with only `knowhub-frontend` cloned, with no backend import, no submodule, no shared DTO source package and no workstation-specific service; and the repository's application and runtime logic is implemented in TypeScript with React and Next.js, carries no Python application or runtime implementation, contains no backend application implementation, and reproduces no backend model, domain rule or service implementation by copy or by transliteration. *(R1, R29, R30)* |

## 8. Deferred

| Deferred from 0B-SPEC-001 | Owner |
|---|---|
| Design tokens, UI primitives, application shell, UI state classes, accessibility implementation | 0B-SPEC-002 |
| Generated API client, contract governance, generation pipeline, freshness gate, correlation and error normalization, server-state conventions | 0B-SPEC-003 |
| Frontend configuration model, secret containment in the browser bundle, session boundary, provider-neutral authentication abstraction, route protection, authorization-aware UI, CSRF, security headers, safe client logging | 0B-SPEC-004 |
| Test layers, CI gate wiring, supply-chain scanning, production build, container image and runtime | 0B-SPEC-005 |
| `docs/ux/`, `public/`, `src/hooks/`, `src/view-models/`, `src/lib/format/` | The change introducing each one's first real content, under R9 and R11 |
| `src/lib/telemetry/` and any browser telemetry SDK, vendor or UX-metrics framework | Deferred; a future ADR when a concrete requirement and technology choice exist |
| A real `contracts/knowhub-openapi.json` and the first pinned backend contract | M1, through the §30A.2 contract-update workflow |
| All product routes, screens and navigation entries in §61A.1 | M1 and later |
| Real authentication: LOCAL credential verification, OIDC exchange, token and session lifecycle, user/role/application administration | M1 |
| `deploy/`, deployment definitions, registry publishing and environment promotion | Post-0B |
