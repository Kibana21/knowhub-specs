# 0B-SPEC-005 — Testing, security, production build and CI

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository:** `knowhub-frontend`
- **Status:** Approved
- **§81 0B items owned:** `CI`

## 1. Purpose and milestone-exit boundary

Establish the verification machinery, the production browser-hardening
baseline, the production build and release artifact, and the CI gate set that
together allow milestone 0B to exit.

[§81.1](../../../knowhub_master_blueprint.html#implementation-dependency-graph)
gates a milestone on its acceptance tests passing, telemetry existing, failure
paths being exercised and preceding contracts remaining stable. The four
Approved 0B specifications state behaviour; each of them explicitly defers the
machinery that *demonstrates* that behaviour to this specification. This
specification supplies it.

**Two boundaries govern everything below.**

**Verification does not transfer ownership.** Where this specification requires
a gate over behaviour owned by
[0B-SPEC-001](0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md),
[0B-SPEC-002](0B-SPEC-002-application-shell-design-system-and-accessibility.md),
[0B-SPEC-003](0B-SPEC-003-openapi-contract-and-api-client-foundation.md) or
[0B-SPEC-004](0B-SPEC-004-configuration-bff-session-and-authorization-foundations.md),
the requirement here is that a verifier exists, executes, blocks and produces
evidence. It never redefines the behaviour, and the citing specification remains
its normative owner.

**Building a release artifact is not designing a deployment platform.** This
specification owns the production build, the production frontend container image
and its commit-bound identity. It owns no registry, no deployment definition, no
runtime platform and no environment promotion.

This specification delivers no product capability, no product route, no product
API contract and no real identity.

## 2. Sources

**Blueprint**

- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Testing, Packaging and CI/CD rows
- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**: the frontend CI required-gate list, the frontend published
  outputs, the primary release artifact, and repository independence
- [§0E.9 Explicit non-choices for V1](../../../knowhub_master_blueprint.html#runtime-architecture)
- [§0E.13.10 CSRF, XSS, CORS and browser hardening](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, the CSP, header and XSS rows
- [§0E.16 Security verification / penetration-test readiness](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, the Frontend and Supply chain rows
- [§30A Frontend codebase — knowhub-frontend](../../../knowhub_master_blueprint.html#target-frontend),
  including §30A.2 (contract update workflow)
- [§61A.4 Frontend quality gates](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — type-safety, contract, accessibility, security and E2E rows
- [§65.4 Accessibility, browser and UX constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
  — the supported-browser row
- [§70.5 API behavior rules](../../../knowhub_master_blueprint.html#api-contracts)
  — contract ownership and the cross-repository compatibility gate
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
  and §71.4 forbidden dependencies
- [§75 Acceptance criteria & executable test catalogue](../../../knowhub_master_blueprint.html#acceptance-tests)
  and [§75.1 Test layers](../../../knowhub_master_blueprint.html#acceptance-tests)
- [§76 Environment & deployment matrix](../../../knowhub_master_blueprint.html#environment-matrix)
  — the Identity row: "No insecure bypass committed enabled"
- [§77.1 and §77.2 Configuration model](../../../knowhub_master_blueprint.html#configuration-model)
  — constraining R41 only; the configuration model itself is owned by 0B-SPEC-004
- [§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph)
  and §81.1
- [§84.1, §84.2, §84.3 and §84.4](../../../knowhub_master_blueprint.html#coding-harness-instructions)

**ADRs** — referenced by number and link only; decision text lives in the
canonical files.

- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
- [ADR-021 — Authentication is provider-pluggable](../../02-adrs/ADR-021-provider-pluggable-authentication.md)
  (scopes the test-mode surface in §8 only)
- [ADR-022 — Hybrid RBAC/ABAC authorization with default deny](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
  (scopes the security-negative layer in §8 only)
- [ADR-024 — Premium bright evidence-first UI design language](https://github.com/Kibana21/knowhub-frontend/blob/main/docs/adr/ADR-024-premium-bright-evidence-first-ui.md)
  — canonically owned by `knowhub-frontend` as recorded in the
  [ADR register](../../02-adrs/README.md); constrains R22 only
- [ADR-025 — Canonical user model is independent of auth provider](../../02-adrs/ADR-025-canonical-user-model-independent-of-provider.md)
- [ADR-027 — Frontend BFF session state ownership](../../02-adrs/ADR-027-frontend-bff-session-state-ownership.md)
  (constrains R34 — the multi-instance property)

**Approved specifications** — 0B-SPEC-001 through 0B-SPEC-004 in full, and
[M00-SPEC-003](../00-foundation/M00-SPEC-003-backend-observability-baseline.md)
R8–R10 for the correlation contract referenced in §13.

### 2.1 Inherited verification obligations

Each row names behaviour an Approved specification owns and the machinery this
specification must supply. **No row transfers ownership.**

| Owning specification | Criteria requiring machinery here | Machinery |
|---|---|---|
| 0B-SPEC-001 | 0B-AC-001, 0B-AC-002 | Frozen install and lockfile-currency gates (§11) |
| 0B-SPEC-001 | 0B-AC-003 | Gates executing on the declared Node baseline (§5, §13) |
| 0B-SPEC-001 | 0B-AC-004, 0B-AC-011 | Lint, format and strict-typecheck gates (§5) |
| 0B-SPEC-001 | 0B-AC-006 | An executable boot of the root layout (§6) |
| 0B-SPEC-001 | 0B-AC-007, 0B-AC-009 | Boundary and server-only placement gate (§5) |
| 0B-SPEC-001 | 0B-AC-012 | Secret-scanning gate (§11) |
| 0B-SPEC-001 | 0B-AC-013 | The full validation sequence run with only `knowhub-frontend` checked out (§3, §13) |
| 0B-SPEC-002 | 0B-AC-020, 0B-AC-021, 0B-AC-027, 0B-AC-031 | Component and accessibility layers (§6) |
| 0B-SPEC-002 | 0B-AC-024, 0B-AC-028, 0B-AC-033, 0B-AC-034, 0B-AC-035 | Rendering and browser harness (§6, §8) |
| 0B-SPEC-002 | 0B-AC-025, 0B-AC-036 | Test-only containment (§4); recorded review evidence (§6) |
| 0B-SPEC-003 | 0B-AC-045, 0B-AC-046, 0B-AC-047, 0B-AC-048, 0B-AC-050 | Contract-generation and freshness gates (§7) |
| 0B-SPEC-003 | 0B-AC-051, 0B-AC-053, 0B-AC-054, 0B-AC-056, 0B-AC-058 | Component and contract test layer (§6, §7) |
| 0B-SPEC-003 | 0B-AC-041, 0B-AC-043 | Evidence-honesty reporting (§3, §7); fixture containment (§4) |
| 0B-SPEC-004 | 0B-AC-061, 0B-AC-076, 0B-AC-086 | Build-artifact inspection, secret scan and containment gate (§4, §10, §11) |
| 0B-SPEC-004 | 0B-AC-065, 0B-AC-066, 0B-AC-067, 0B-AC-072, 0B-AC-074, 0B-AC-079, 0B-AC-082, 0B-AC-083, 0B-AC-084, 0B-AC-085 | Browser and security-negative layers (§8) |
| 0B-SPEC-004 | 0B-AC-069, 0B-AC-071 | Multi-instance verification (§8) |
| 0B-SPEC-004 | 0B-AC-060, 0B-AC-062, 0B-AC-063, 0B-AC-064, 0B-AC-068, 0B-AC-070, 0B-AC-073, 0B-AC-075, 0B-AC-077, 0B-AC-078, 0B-AC-080, 0B-AC-081 | Component test layer, where not decided by inspection (§6) |

0B-SPEC-004 §1.1, §14 and §16 additionally **delegate** the application-wide
production HTTP security-header baseline — CSP, HSTS, `X-Content-Type-Options`,
`Referrer-Policy`, `Permissions-Policy` and anti-clickjacking — and its
production and runtime verification to this specification, together with
ownership of [§75](../../../knowhub_master_blueprint.html#acceptance-tests)
AT-035. That is a transfer of ownership, and §9 discharges it.

## 3. Normative requirements — test architecture, determinism and independence

**R1.** The [§75.1](../../../knowhub_master_blueprint.html#acceptance-tests)
test-layer model, as it applies to `knowhub-frontend`, is the **target**
structure for this repository's test tree. 0B materializes only the layers that
contain real 0B tests. No empty test-layer directory is created to mirror the
target model, and a layer is created by the milestone that first supplies real
tests for it.
*(§75.1; §30A; §84.1 — deliver working vertical slices, not a large set of empty
interfaces; §84.3)*

**R2.** The layers materialized at 0B are those for which 0B has real tests:
component and unit; contract; accessibility; and browser/end-to-end including
its security-negative cases. Each exists because 0B behaviour requires it, not
because a gate list names it.
*(§0E.8 frontend CI gate list; §61A.4; §75.1; R1)*

**R3.** Test execution is **deterministic**. A repeated run over unchanged
inputs produces the same result, and no outcome depends on execution order,
ambient wall-clock time or shared mutable external state. Where behaviour under
test is time-sensitive, time is controlled deterministically.
*(§84.2 — success and failure paths are tested; §81.1)*

**R4.** Verification has **no undeclared external application or service
dependency**. Specifically it requires no `knowhub-backend` checkout, no running
KnowHub backend, no real identity provider, no sibling repository, no remote
product service and no workstation-specific state.
*(§71.3 — Independent CI; §0E.8 — "One repository must never be required merely
to compile the other"; [ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md);
0B-SPEC-001 R29; 0B-SPEC-004 R75)*

**R5.** R4 is a **service and repository independence** requirement, not an
offline-execution requirement. It does not restrict loopback or local traffic
between a browser harness and a locally started `knowhub-frontend` runtime,
locally hosted synthetic authorities or providers, deterministic fake clocks,
ordinary dependency acquisition, or a scanner reaching its vulnerability data
source. **This specification imposes no general offline-execution requirement.**
The single offline obligation binding 0B is
[0B-SPEC-003](0B-SPEC-003-openapi-contract-and-api-client-foundation.md) R17,
which governs contract generation and is carried unchanged into R24.
*(§71.3; §0E.8; 0B-SPEC-003 R17)*

**R6.** **Failure paths are exercised, not only success paths.** Every gate this
specification requires is demonstrated to fail on the condition it exists to
detect, and each negative behaviour named in §15 is executed as a failing case
rather than assumed.
*(§81.1 — failure paths are exercised; §84.2)*

**R7.** **Verification evidence states what it demonstrates and no more.** No
evidence artifact, gate result or acceptance record may be presented as showing
compatibility with `knowhub-backend`, the existence of a backend endpoint, real
identity, real authorization, or the configuration of a deployment platform.
*(0B-SPEC-003 R8; §81.1; §84.2; §84.4)*

**R8.** Every verification obligation inherited under §2.1 is executed by a
layer this specification provides. This specification **verifies** those
behaviours and **redefines none of them**; each remains owned by the
specification that states it.
*(0B-SPEC-001 §6; 0B-SPEC-002 §8; 0B-SPEC-003 §14; 0B-SPEC-004 §14)*

## 4. Normative requirements — test-only material and production isolation

**R9.** Test-only material — fixtures, the synthetic contract fixture, the
test-only identity and session double, and test utilities — is **structurally
unreachable from the production bundle and from the production runtime**.
*(§76 Identity row — "No insecure bypass committed enabled"; §84.3; 0B-SPEC-002
R19; 0B-SPEC-003 R11, R12; 0B-SPEC-004 R73)*

**R10.** R9 is satisfied only when all of the following hold, each objectively
decidable:

| # | Property |
|---|---|
| a | No production entry point reaches the material |
| b | No production bundle and no emitted source map contains it |
| c | No production runtime exposes a test-only route, endpoint, authority or profile selector |
| d | Production source cannot accidentally depend on test-only implementation, and such a dependency is detected mechanically |
| e | Explicit local or test enablement is an **additional** protection and never the sole protection |

*(§76 Identity row; 0B-SPEC-004 R72, R73; §84.3)*

**R11.** This specification fixes **no containment mechanism and no filesystem
placement**, and mandates no bundler configuration, path alias, directory
structure or conditional-import technique. Where an Approved specification
already fixes placement — notably 0B-SPEC-003 R11 for the synthetic contract
fixture — that placement continues to bind and this specification adds
verification rather than a second placement rule. For all other test-only
material the mechanism and location are implementation-plan decisions, provided
R9 and R10 hold.
*(0B-SPEC-003 R11; §84.1; 0B-SPEC-001 R11)*

**R12.** A violation of R9 or R10 is detected by an automated gate and **fails**.
*(§0E.8; §61A.4 security row; §84.2)*

## 5. Normative requirements — static quality gates

**R13.** Lint enforcement executes as a gate and **blocks**; the lint tool is
ESLint, as 0B-SPEC-001 R23 requires.
*(§0E.8 frontend CI gate list — ESLint; 0B-SPEC-001 R23)*

**R14.** The mechanical formatting check of 0B-SPEC-001 R24 executes as a gate
and **blocks**.
*(§0E.8; §61A.4; 0B-SPEC-001 R24)*

**R15.** Static type checking executes as a gate and **blocks**; a type error in
KnowHub-owned application source fails it.
*(§0E.8 — strict TypeScript; §61A.4 type-safety row; 0B-SPEC-001 R7, R25)*

**R16.** The module-boundary gate executes and **blocks**, covering both the
§61A.2 layering rule and the server-only placement rule.
*(§61A.4; §84.2; 0B-SPEC-001 R15, R17–R22)*

**R17.** Each gate in R13–R16 is demonstrably **capable of failing**: a
deliberately introduced violation fails it, and removing that violation restores
a passing run.
*(0B-SPEC-001 R22; §81.1; R6)*

**R18.** Gates execute on the Node baseline declared under 0B-SPEC-001 R5 — the
same baseline local tooling uses.
*(0B-SPEC-001 R5; §0E.8)*

## 6. Normative requirements — component and accessibility verification

**R19.** A component and unit verification layer exists, executes as a gate and
**blocks**. §0E.8's frontend CI gate list names **Vitest** for this layer.
*(§0E.8 — "Vitest/component tests"; §0E.1 Testing row; §75.1)*

*Implementation guidance (non-normative): §0E.1 pairs Vitest with React Testing
Library for "focused frontend component tests". §0E.8's gate list names only
Vitest, so the rendering and interaction library is an implementation-plan
choice with React Testing Library as the blueprint's baseline.*

**R20.** That layer executes the inherited obligations §2.1 assigns to a
component or contract test layer, so that each is demonstrated rather than
asserted.
*(§2.1; §61A.4; R8)*

**R21.** Automated accessibility checks run over the surfaces 0B delivers — the
application shell, the primitive layer and the state-class presentations —
execute as a gate and **block**. No accessibility engine is fixed: no source
names one.
*(§0E.8 — "accessibility tests"; §61A.4 accessibility row — "automated
accessibility checks for critical flows"; §75 AT-017; 0B-SPEC-002 R30–R43)*

**R22.** Where a criterion cannot be decided by automation, its evidence is a
**targeted assertion or a recorded review**, identified as such and never
presented as an automated result. **No numeric or automated metric is fabricated
for a criterion automation cannot decide, and no aesthetic or design-quality
judgement is expressed as a test threshold.**
*(§61A.4; 0B-SPEC-002 0B-AC-031 and 0B-AC-036;
[ADR-024](https://github.com/Kibana21/knowhub-frontend/blob/main/docs/adr/ADR-024-premium-bright-evidence-first-ui.md);
R7)*

**R23.** Accessibility evidence covers only the surfaces 0B delivers. The
primary Ask, evidence and source workflows AT-017 names do not exist at 0B, and
no evidence produced here is presented as discharging AT-017.
*(§75 AT-017; 0B-SPEC-002 §9; §81; R7)*

## 7. Normative requirements — contract pipeline and freshness verification

**R24.** The generation properties 0B-SPEC-003 establishes execute as gates:
byte-identical regeneration from the same input and committed configuration;
clear failure on invalid or unparseable input leaving no partial, empty or
stale output; and **generation with no `knowhub-backend` checkout, no running
backend and no network access**.
*(0B-SPEC-003 R15, R16, R17; §0E.8; §61A.4 contract row)*

**R25.** The freshness gate is **wired and blocking**, and detects each drift
case 0B-SPEC-003 R24 enumerates: stale generated output; hand-edited generated
output; generator or configuration drift that changes output; and invalid
contract input.
*(0B-SPEC-003 R23, R24; §0E.8 — "generated-client freshness"; §61A.4)*

**R26.** The gates in R24 and R25 operate on the legitimate pinned
backend-generated artifact where one exists under 0B-SPEC-003 R7, and otherwise
on 0B-SPEC-003's sanctioned synthetic fixture, which R25 of that specification
makes the mandatory freshness demonstration.
*(0B-SPEC-003 R7, R9, R25, R26)*

**R27.** **Contract evidence is reported honestly.** Where a legitimate pinned
backend-generated artifact exists, the **exact backend contract revision used
for the build is recorded and published** among the CI outputs. Where only the
synthetic fixture exists, **no backend contract revision is fabricated or
published**; the absence of a bound product backend contract is recorded
explicitly; the fixture is identified only as test and pipeline evidence; and no
backend compatibility is claimed.
*(§0E.8 published outputs — "the exact backend contract revision used for the
build"; 0B-SPEC-003 R8, R25; §70.5; R7)*

## 8. Normative requirements — browser, multi-instance and security-negative verification

**R28.** A browser and end-to-end verification layer exists, executes as a gate
and **blocks**. §0E.8's frontend CI gate list, §61A.4's E2E row, §75.1 and §84.3
all name **Playwright** for this layer.
*(§0E.8 — "Playwright critical journeys"; §61A.4 E2E row; §75.1; §84.3)*

**R29.** **Chromium is the mandatory blocking browser engine at 0B.** §65.4
requires support for current enterprise-managed versions of Chrome and Edge, and
both are Chromium-based. Safari and WebKit verification remains **conditional on
the organizational policy §65.4 makes it conditional upon**, and becomes required
when that policy applies. **Firefox and mobile-browser verification are not
required**, no source establishing either. Running additional Chrome or Edge
channels or smoke coverage is permitted and is an implementation-plan decision.
*(§65.4 — "Support current enterprise-managed versions of Chrome and Edge;
Safari support where organizational policy requires it"; §0E.17.7; §84.1)*

**R30.** The browser layer drives the authentication and session boundary
**through the test-only identity and session double** 0B-SPEC-004 §13
establishes, and **creates no user-visible product authentication surface** in
order to do so. No product login page, provider chooser, credential-entry UI or
identity-provider branding is introduced by verification.
*(0B-SPEC-004 §1.1, R71–R75; §61A.4 E2E row — "configured OIDC/BFF sign-in test
mode"; §81;
[ADR-021](../../02-adrs/ADR-021-provider-pluggable-authentication.md))*

**R31.** Browser inspection verifies the browser-storage and cookie properties
0B-SPEC-004 R17–R20 require, covering `localStorage`, `sessionStorage`,
IndexedDB, the URL path, query string and fragment, JavaScript-readable cookies,
page props and hydrated state.
*(0B-SPEC-004 R17–R20, 0B-AC-065, 0B-AC-066; §75 AT-027; §61A.4 security row)*

**R32.** The layer verifies the server-side-only credential path: that browser
code constructs no backend `Authorization` header, that the authorization
response is handled server-side with no provider token, authorization code,
client credential or PKCE verifier reaching browser application code, and that a
redirect or callback target outside the approved set is rejected.
*(0B-SPEC-004 R21, R22, R31, R32, R33, R65; 0B-AC-067, 0B-AC-072, 0B-AC-074)*

**R33.** The security-negative layer verifies request integrity: a cross-origin
state-changing browser → BFF request lacking valid CSRF or origin controls is
rejected, a legitimate same-origin one succeeds, and LOCAL and OIDC receive the
same protection.
*(0B-SPEC-004 R60–R64, 0B-AC-084; §0E.13.10; §75 AT-029; §0E.16 Frontend row)*

**R34.** **Multi-instance verification** executes against at least two
concurrently running `knowhub-frontend` instances with no sticky routing: the
same browser session is served by either instance, and an OIDC transaction
initiated on one completes on another. No **frontend-owned durable session state
or session authority** is shared between them. The category-B transaction-state
mechanism remains unfixed by this specification, exactly as 0B-SPEC-004 R30
leaves it.
*(0B-SPEC-004 R25, R30, 0B-AC-069, 0B-AC-071;
[ADR-027](../../02-adrs/ADR-027-frontend-bff-session-state-ownership.md) §2, §6)*

**R35.** The security-negative layer verifies that editing a route URL, client
state, browser storage or JavaScript state grants no capability; that redirect
and deny behaviour is deterministic; that a loading, pending or unknown state
grants no access; and that no product protected route exists.
*(0B-SPEC-004 R51, R52, R54–R59, 0B-AC-082, 0B-AC-083;
[ADR-022](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md);
§0E.16 Authorization row)*

**R36.** Against the **synthetic** session authority, the layer verifies that
the browser session reference rotates on authentication and re-authentication,
that a pre-authentication reference cannot be reused afterwards, and that the
cookie is cleared on sign-out.
*(0B-SPEC-004 R43, 0B-AC-079; §0E.16 Authentication/session row)*

**R37.** The layer verifies non-leakage: no session identifier, raw cookie,
access token, refresh token, external identity-provider token, password, reset
token, OIDC authorization code, MFA secret, claims payload, stack trace,
internal provider response or client secret appears in a log, diagnostic,
telemetry field or browser-visible surface.
*(0B-SPEC-004 R66–R70, 0B-AC-085; 0B-SPEC-002 R27; §0E.13.9; §0E.16 Frontend row)*

**R38.** **Non-claims.** Evidence produced by this layer claims **no** real
backend token rejection, **no** real session invalidation, **no** real
server-side token-cache invalidation, **no** real refresh-token rotation,
**no** real LOCAL credential verification, **no** real federated-token
validation, **no** real OIDC tenant integration and **no** backend
authorization. Each is backend capability delivered at M1 or later.
*(0B-SPEC-004 §15 forward catalogue target and §16; §81; R7)*

## 9. Normative requirements — production browser hardening

0B-SPEC-004 §1.1, §14 and §16 delegate this section's subject matter and AT-035
to this specification. Nothing here duplicates a 0B-SPEC-004 requirement: the
session-coupled controls — cookie security, CSRF and request integrity, Origin
and Referer validation, redirect validation, and auth and session redaction —
remain owned there.

**R39.** The production `knowhub-frontend` runtime emits an **application-wide
HTTP security-header baseline** on the responses it serves.
*(§0E.13.10 Headers row; §0E.16 Frontend row; §75 AT-035; 0B-SPEC-004 §1.1)*

**R40.** A **restrictive production Content Security Policy** is part of that
baseline and establishes: `object-src 'none'`; **no unsafe eval**; **no
arbitrary third-party script sources**; and a **controlled `connect-src`**.
*(§0E.13.10 CSP row)*

**R41.** The `connect-src` value is a **controlled allowlist of approved
browser-visible destinations**. It must **not** create or imply direct browser →
KnowHub-backend access contrary to the boundary 0B-SPEC-003 R26–R27 and
0B-SPEC-004 R21–R22 establish, and must **not** expose a server-only backend
origin merely to populate a directive. Where configuration participates in
determining browser-visible destinations, only values explicitly permitted
through 0B-SPEC-004's browser-exposable configuration boundary may do so. **No
backend origin is added automatically.** The source list itself is not fixed by
this specification.
*(§0E.13.10 — "controlled connect-src"; §0E.13.10 CORS row — prefer same-origin
browser → BFF traffic; 0B-SPEC-003 R26, R27; 0B-SPEC-004 R5–R7, R21, R22; §77.2)*

**R42.** **HSTS** is part of the baseline.
*(§0E.13.10 Headers row; §75 AT-035)*

**R43.** **`X-Content-Type-Options`** is part of the baseline.
*(§0E.13.10 Headers row; §75 AT-035 — nosniff)*

**R44.** A **strict referrer policy** is part of the baseline.
*(§0E.13.10 Headers row; §75 AT-035)*

**R45.** A **`Permissions-Policy`** is part of the baseline.
*(§0E.13.10 Headers row; §75 AT-035)*

**R46.** **Anti-clickjacking protection** is part of the baseline, stated as an
outcome: **unauthorized or unapproved frame embedding is denied**. The mechanism
is not fixed — the sources require the control, not a particular header.
*(§0E.13.10 Headers row — "anti-clickjacking controls"; §0E.16 Frontend row —
clickjacking/header checks; §75 AT-035)*

**R47.** This specification fixes the **presence** of the controls in R40 and
R42–R46 and the CSP properties in R40 and R41, and **fixes no value**: no HSTS
`max-age` or directive set, no referrer-policy value, no `Permissions-Policy`
directive list, no anti-clickjacking mechanism, no `connect-src` source list and
no script nonce or hash strategy. It requires **no** control the sources do not
establish — in particular no COOP, COEP or CORP, no Subresource Integrity, no
CSP reporting endpoint and no CORS policy, the backend API CORS allowlist being
backend-owned.
*(§0E.13.10 — the sources fix no value; §0E.13.10 CORS row; 0B-SPEC-004 §11
guidance; §84.1)*

**R48.** The baseline is **verified against the production artifact and the
runtime it produces**, and a missing, removed or weakened control **fails the
gate**.
*(§0E.16 Frontend row — CSP validation and clickjacking/header checks; §61A.4
security row; §75 AT-035; R6)*

**R49.** **Scope of the claim.** Evidence under R48 is evidence at the
production frontend artifact and runtime boundary. It does **not** demonstrate
deployment-platform configuration: where a control is ultimately applied or
strengthened at a gateway, ingress, CDN or TLS-termination layer,
environment-level confirmation remains deployment work.
*(§76; §76.1; §81; R7)*

## 10. Normative requirements — unsafe-HTML safety and build-artifact inspection

**R50.** **No production rendering path passes unsanitized untrusted content
through a raw-HTML or unsafe DOM boundary.** A gate detects a path by which
untrusted content can reach such a boundary **without the required sanitization
control**, and fails. Because 0B renders no untrusted rich content, its
foundation-level proof may establish that no such path exists. Content that has
passed the required allowlist sanitization is not thereby prohibited from being
rendered by the capability that introduces it.
*(§0E.13.10 XSS row — sanitize untrusted rich HTML/Markdown with an allowlist,
and avoid unsafe DOM APIs; §61A.4 security row; §0E.16 Frontend row — XSS/HTML
sanitization tests; R51)*

**R51.** 0B renders **no untrusted rich content**, so **no sanitization library
is selected here**. §0E.13.10's allowlist-sanitization requirement for untrusted
rich HTML and Markdown, and §61A.4's Markdown and Mermaid sanitization row, bind
the capability that first renders such content. This specification owns only the
foundation-level guard in R50.
*(§0E.13.10 XSS row; §61A.4 security row; §81; 0B-SPEC-002 §8)*

**R52.** **Build-artifact inspection** covers the production bundle and any
emitted source map, and finds no secret, credential, token, resolved provider
credential or server-only configuration value.
*(§0E.16 Frontend row — "no secrets/source maps containing credentials";
§61A.4 security row; 0B-SPEC-004 R5–R7, R9, 0B-AC-061)*

**R53.** The same inspection finds **no test-only fixture, double, authority or
identifier**, discharging R10(b) against the produced artifact.
*(R9, R10; 0B-SPEC-004 R73, 0B-AC-086; §84.3)*

**R54.** This specification **requires no source-map policy and forbids none**.
Whatever the production build emits is subject to R52 and R53.
*(§0E.16 — the requirement is that no source map contain credentials; §84.1)*

## 11. Normative requirements — dependency and supply-chain security

**R55.** Installation in CI and in a clean checkout resolves **strictly from the
committed lockfile** and fails rather than drifting; this executes as a
blocking gate.
*(§0E.16 Supply chain row — pinned lockfiles; 0B-SPEC-001 R2, R3, 0B-AC-001)*

**R56.** Manifest and lockfile currency is checked **mechanically** as a
blocking gate.
*(§0E.16 Supply chain row; 0B-SPEC-001 R4, 0B-AC-002)*

**R57.** A **dependency vulnerability gate** over resolved dependencies is
present and blocking. No scanning product forms part of the architectural
contract; the implementation plan selects it.
*(§0E.8 frontend CI gate list — dependency/container scan; §0E.16 Supply chain
row — SCA; §0E.16 Frontend row — dependency audit)*

**R58.** A **secret-scanning gate** is present and blocking: committed
credentials, tokens, keys and secrets are detected and fail CI. Its capability
is demonstrated against an **ephemeral synthetic credential** placed into a
verification input created for the check and discarded afterwards, so that no
real or persistent credential is committed and none remains once verification
completes. No scanning product forms part of the architectural contract.
*(§0E.16 Supply chain row — secret scanning; §84.3; §65.3; 0B-SPEC-001 R28,
0B-AC-012; 0B-SPEC-004 R38, 0B-AC-076)*

**R59.** A **container vulnerability scan** of the image produced under §12 is
present and blocking. No scanning product forms part of the architectural
contract.
*(§0E.8 frontend CI gate list — dependency/container scan; §0E.16 Supply chain
row — container scan)*

**R60.** A scanning or audit gate reporting **zero findings has passed**. Zero
findings is a legitimate result and is not evidence that a gate is missing or
misconfigured.
*(§0E.8; §84.2)*

**R61.** **SBOM generation, provenance generation and artifact signing are not
0B requirements**, and **no tooling, format or registry integration for them is
selected here**. §0E.16 records them among broader supply-chain controls
"according to platform standard", and §0E.8's frontend CI gate and published-output
lists contain none of them. They are deferred with named ownership in §16 rather
than dropped. **License scanning is likewise not a 0B requirement**, no source
establishing one.
*(§0E.16 Supply chain row; §0E.8 frontend CI row; §84.1 — do not introduce by
default)*

## 12. Normative requirements — production build and release artifact

**R62.** A **production build** of the application is produced, and it executes
as a blocking gate.
*(§0E.8 published outputs; §0E.1 Web application row; §61A.4)*

**R63.** A strict type error and a lint violation each **block the production
build**.
*(§61A.4 type-safety row; §0E.8 — ESLint and strict TypeScript; 0B-SPEC-001
R23, R25)*

**R64.** Once dependencies and build inputs are available, the build requires
**no live KnowHub backend, no real identity provider and no product runtime
service**, and requires **no product OpenAPI contract** — 0B-SPEC-003 R7 makes
the normal 0B path one on which none exists.
*(§71.3 — Independent CI; §0E.8; 0B-SPEC-003 R7, R17;
[ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md);
R5)*

**R65.** **No development-only or test-only code is present in the produced
bundle.**
*(R9, R10, R53; §84.3; 0B-SPEC-004 R73)*

**R66.** A **production `knowhub-frontend` container image** is produced from
the repository, and its build executes as a blocking gate.
*(§0E.1 Packaging row — "knowhub-frontend produces the Next.js image"; §0E.8 —
the frontend primary release artifact and published outputs; §30A — `Dockerfile`)*

**R67.** The release artifact's identity **binds the repository name and the
exact commit**, in the form §0E.8 states as `knowhub-frontend:<git-sha>`. No
registry, no publishing step and no further tag scheme is fixed by this
specification.
*(§0E.8 Primary release artifact column; §84.3)*

**R68.** The image is subject to the container vulnerability scan required by
R59.
*(§0E.8; §0E.16)*

**R69.** **No deployment-platform work is introduced at 0B**: no registry
publishing, no deployment definition, no Kubernetes or ingress configuration, no
cloud runtime design, no Terraform or Bicep, no environment promotion, no
production identity-provider registration, no TLS, CDN or WAF design, and no
`deploy/` content. Building and validating a release artifact is not designing a
deployment platform.
*(§84.3 — `deploy/` is the location for frontend runtime deployment definitions;
0B-SPEC-001 §8 Deferred — `deploy/`, deployment definitions, registry publishing
and environment promotion are post-0B; §0E.9; §84.1; §76.1)*

**R70.** **No backend container implementation detail is imported.** A non-root
user identifier, a container health check, a multi-stage structure, image labels
and an exposed-port choice are **not fixed by this specification**; no frontend
source establishes any of them, and `knowhub-backend`'s image decisions bind
only `knowhub-backend`.
*(§0E.1 Packaging row; §84.1; ADR-016)*

## 13. Normative requirements — CI quality gates and independence

**R71.** The following gates are **required and blocking** for
`knowhub-frontend` at 0B:

| Gate | Requirement | Source |
|---|---|---|
| Lockfile and frozen install | R55, R56 | §0E.16 Supply chain row |
| Lint | R13 | §0E.8 CI gate list — ESLint |
| Format | R14 | §0E.8; §61A.4 |
| Type check | R15 | §0E.8 — strict TypeScript; §61A.4 |
| Module boundary | R16 | §61A.4; 0B-SPEC-001 R21 |
| Component and unit tests | R19, R20 | §0E.8 — Vitest/component tests |
| Contract generation and freshness | R24, R25 | §0E.8 — generated-client freshness; §61A.4 contract row |
| Accessibility | R21 | §0E.8 — accessibility tests; §61A.4 |
| Browser, end-to-end and security-negative | R28–R37 | §0E.8 — Playwright critical journeys; §75.1 |
| Unsafe-HTML guard | R50 | §0E.16 Frontend row; §61A.4 |
| Dependency vulnerability | R57 | §0E.8; §0E.16 |
| Secret scan | R58 | §0E.16; §84.3 |
| Production build | R62, R63 | §0E.8 published outputs |
| Build-artifact and security-header inspection | R48, R52, R53 | §0E.16 Frontend row; §75 AT-035 |
| Container image build | R66 | §0E.1 Packaging; §0E.8 |
| Container vulnerability scan | R59, R68 | §0E.8; §0E.16 |

Each row's source is stated in its third column; the gate set as a whole is
§0E.8's frontend CI required-gate list, extended only where §0E.16, §61A.4 or
§75 AT-035 require a further outcome of this repository.
*(§0E.8 frontend CI row; §0E.16 Frontend and Supply chain rows; §61A.4; §75
AT-035)*

**R72.** **No required gate is advisory**: a gate that executes blocks on
failure. This requires no empty layer and no placeholder job — a test layer
exists because it holds real 0B tests (R1, R2), and a scanning gate that finds
nothing has passed (R60).
*(§0E.8; §81.1; §84.2; R1, R2, R60)*

**R73.** The **complete frontend CI path runs with only `knowhub-frontend`
checked out**. It requires no `knowhub-backend` checkout, source tree or Python
package, no running KnowHub backend, no real identity provider, no sibling
repository and no remote product service. **No job, step or script fetches from
or depends on a `knowhub-backend` checkout or runtime in order to lint, test or
build the frontend.**

**Exception, preserved explicitly.** A legitimate pinned backend-generated
OpenAPI contract artifact governed by 0B-SPEC-003 R2–R4 and R7 is permitted as
contract **input**, and may retain and record its backend contract provenance.
That is an artifact input, not a source-code or runtime dependency on
`knowhub-backend`, and it does not weaken repository independence.
*(§71.3 — Independent CI; §71.4 — `frontend repo ✗→ backend Python packages`;
§0E.8 — cross-repository validation uses a published OpenAPI artifact, package
or image artifact; §30A.2;
[ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md);
[ADR-017](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md);
0B-SPEC-001 R29, 0B-AC-013; 0B-SPEC-003 R2–R4, R7; R4, R5)*

**R74.** CI produces the outputs §0E.8 requires of the frontend pipeline: the
container image, and the test and UX reports. These are **CI evidence
artifacts**.
*(§0E.8 Published outputs column)*

**R75.** **CI evidence artifacts are not runtime telemetry**, and are not
offered as satisfying §81.1's telemetry condition. At 0B that condition rests on
the backend runtime telemetry established by
[M00-SPEC-003](../00-foundation/M00-SPEC-003-backend-observability-baseline.md)
and on the correlation propagation 0B-SPEC-003 R34–R36 establishes into it,
together with the safe auth and session diagnostics 0B-SPEC-004 §12 constrains.
**Frontend browser telemetry remains deferred**: no browser tracing SDK, vendor
integration or metrics framework is introduced.
*(§81.1; §0E.8; M00-SPEC-003 R8–R10; 0B-SPEC-001 R9, R10; 0B-SPEC-003 R37;
0B-SPEC-004 R70)*

**R76.** CI requirements are stated as **outcomes and gates, not as
platform-specific pipeline syntax**, and **this specification fixes no CI
provider**. §0E.1 permits Azure DevOps Pipelines or GitHub Actions according to
enterprise standard. The provider, workflow filenames, pipeline syntax, job and
stage names, runner images, caching and matrix expression are
implementation-planning decisions and may change without amending this
specification.
*(§0E.1 CI/CD row; §84.1)*

**R77.** Gate **ordering is fixed only where it is architecturally meaningful**:
an artifact-inspection gate runs against the artifact the build produced, and no
gate may be satisfied by a step that runs after it. All other ordering,
parallelism and job decomposition are implementation-planning decisions.
*(§0E.8; §84.2)*

## 14. Forward constraints recorded, not implemented at 0B-SPEC-005

- **The cross-repository compatibility gate is not this specification's.** §0E.8
  and §70.5 place breaking-change detection before backend release, and frontend
  compatibility tests against the backend contract selected for release. That
  gate spans both repositories and a published or deployed artifact; §0E.8's
  cross-repo row activates at §81 milestone 9. Nothing produced here satisfies
  it. *(§0E.8; §70.5; §81)*
- **Product journeys named by §61A.4's E2E row.** Account activation and reset,
  session expiry against a real authority, application navigation, Ask and
  evidence, source onboarding, traceability, admin RBAC and the real
  access-denied journey all presuppose capability delivered at M1 and later. The
  0B browser layer covers only the boundary behaviour 0B-SPEC-004 delivers.
  *(§61A.4 E2E row; §81)*
- **Untrusted content sanitization.** §0E.13.10's allowlist sanitization and
  §61A.4's Markdown and Mermaid row bind the capability that first renders
  untrusted rich content; R50 is the foundation guard only. *(§0E.13.10;
  §61A.4; §81)*
- **Deployment-platform confirmation of the security baseline.** R49 records the
  limit of the 0B claim; environment-level confirmation is deployment work.
  *(§76; §76.1)*
- **Performance and bundle quality.** No source fixes a bundle-size budget, a
  Core Web Vitals target, a Lighthouse threshold or a startup-time target for
  `knowhub-frontend`, and §61A.4's performance row presupposes volume 0B does not
  have. None is introduced, and none is invented. *(§61A.4; §65.1; §84.1;
  0B-SPEC-002 §8)*
- **Coverage thresholds and visual-regression testing.** No source establishes a
  coverage percentage or a screenshot-comparison gate; neither is introduced.
  *(§0E.8; §61A.4; §84.1)*
- **A §84.4 stop condition recorded in advance.** A verification or CI mechanism
  that introduced frontend-owned durable or shared **session or authorization**
  persistence or authority, a product session or authorization store, or a
  requirement for sticky routing would contradict ADR-027 §2 and §6 and is new
  infrastructure under §84.1 — stop and surface; a superseding ADR is required.
  A mechanism is **not** implicated merely because short-lived category-B
  transaction correlation works across frontend instances, which 0B-SPEC-004 R30
  permits. *(§84.1; §84.4; ADR-027; 0B-SPEC-004 R30, §14)*

## 15. Acceptance criteria

| ID | Criterion |
|---|---|
| **0B-AC-090** | The §75.1 layer model is recorded as the target for the test tree; only layers holding real 0B tests are materialized — component and unit, contract, accessibility, and browser/end-to-end including its security-negative cases — and no empty test-layer directory exists for a layer with no tests yet. *(R1, R2)* |
| **0B-AC-091** | **Test execution is deterministic over harness-controlled inputs**: an unchanged controlled input produces the same test result for the application, component, contract, browser and security-negative behaviour under test, with no dependence on execution order or on ambient wall-clock time, and with time controlled deterministically where the behaviour under test is time-sensitive. Verification has no undeclared external application or service dependency: no `knowhub-backend` checkout, no running KnowHub backend, no real identity provider, no sibling repository, no remote product service and no workstation-specific state. Loopback or local traffic to a locally started frontend runtime, locally hosted synthetic authorities, deterministic fake clocks, ordinary dependency acquisition and scanner data access are unaffected, no general offline-execution requirement being imposed. **Determinism is not claimed for mutable external security intelligence**: a dependency vulnerability database, container vulnerability intelligence or another permitted external advisory feed may change between runs and may legitimately change a scanning gate's result; no requirement to pin or snapshot such data is created, and those gates remain blocking. Failure paths are exercised rather than assumed. *(R3, R4, R5, R6)* |
| **0B-AC-092** | Test-only material — fixtures, the synthetic contract fixture, the identity and session double, and test utilities — is structurally unreachable from the production bundle and the production runtime: no production entry point reaches it, no production bundle or emitted source map contains it, no production runtime exposes a test-only route, endpoint, authority or profile selector, and an accidental production dependency on test-only implementation is detected mechanically. Explicit local or test enablement is present only as an additional protection and is not accepted as the guarantee. The placement 0B-SPEC-003 R11 already fixes for the synthetic contract fixture continues to bind; no other containment mechanism, filesystem placement or bundler technique is mandated. A violation fails an automated gate. *(R9, R10, R11, R12)* |
| **0B-AC-093** | The complete validation sequence — install, lint, format, type check, boundary, component and unit, contract generation and freshness, accessibility, browser and security-negative, production build, artifact inspection, container image and container scan — completes with only `knowhub-frontend` checked out, requiring no `knowhub-backend` checkout, no running KnowHub backend, no real identity provider, no sibling repository and no remote product service. *(R4, R73)* |
| **0B-AC-094** | The lint, mechanical format and strict type-check gates each execute and block; a deliberately introduced violation of each fails CI, and removing it restores a passing run. *(R13, R14, R15, R17)* |
| **0B-AC-095** | The module-boundary gate executes and blocks over both the §61A.2 layering rule and the server-only placement rule; a deliberately introduced violation of each fails it, and removing it restores a passing run. *(R16, R17)* |
| **0B-AC-096** | The component and unit layer executes as a blocking gate and demonstrates the inherited obligations §2.1 assigns to a component or contract test layer — including 0B-AC-006, 0B-AC-020, 0B-AC-021, 0B-AC-051, 0B-AC-053, 0B-AC-054, 0B-AC-056 and 0B-AC-058, and the 0B-SPEC-004 component-decidable set 0B-AC-060, 0B-AC-062, 0B-AC-063, 0B-AC-064, 0B-AC-068, 0B-AC-070, 0B-AC-073, 0B-AC-075, 0B-AC-077, 0B-AC-078, 0B-AC-080 and 0B-AC-081 — without redefining any of the behaviours those criteria govern. *(R8, R19, R20)* |
| **0B-AC-097** | Automated accessibility checks run over the application shell, the primitive layer and the state-class presentations as a blocking gate, evidencing 0B-AC-027 and 0B-AC-031; criteria automation cannot decide — including the 0B-AC-036 review checklist — are evidenced by targeted assertion or recorded review, identified as such; no numeric or automated metric is fabricated for a criterion automation cannot decide, and no aesthetic judgement is expressed as a test threshold; and the evidence is scoped to the surfaces 0B delivers, AT-017 not being presented as discharged. *(R21, R22, R23)* |
| **0B-AC-098** | The browser and end-to-end layer executes as a blocking gate on Chromium, evidencing 0B-AC-024, 0B-AC-028, 0B-AC-033, 0B-AC-034 and 0B-AC-035; Safari and WebKit verification remains conditional on the organizational policy §65.4 makes it conditional upon, and no Firefox or mobile-browser verification is required. *(R28, R29)* |
| **0B-AC-099** | Contract-generation gates execute and block: regeneration from the same input and committed configuration is byte-identical; a malformed or unparseable document fails clearly and leaves no partial, empty or stale output; and generation completes with no `knowhub-backend` checkout, no running backend and no network access. *(R24)* |
| **0B-AC-100** | The freshness gate is wired and blocking and detects each 0B-SPEC-003 R24 drift case — stale generated output, hand-edited generated output, generator or configuration drift that alters output, and invalid contract input — with regeneration restoring the expected state; the gate operates on the legitimate pinned backend-generated artifact where one exists under 0B-SPEC-003 R7 and otherwise on the sanctioned synthetic fixture. *(R25, R26)* |
| **0B-AC-101** | Browser inspection confirms that no KnowHub access token, KnowHub refresh or session token, and no external identity-provider token is present in `localStorage`, `sessionStorage`, IndexedDB, a URL path, query string or fragment, a JavaScript-readable cookie, page props or hydrated state, and confirms the hardened properties of the opaque session cookie. *(R31)* |
| **0B-AC-102** | The layer confirms that browser code constructs no backend `Authorization` header; that the authorization response is handled server-side with no provider token, authorization code, client credential or PKCE code verifier reaching browser application code; and that a redirect or callback target outside the approved set is rejected. The boundary is driven through the test-only identity and session double, and no user-visible product authentication surface — login page, provider chooser, credential-entry UI or provider branding — has been created in order to test it. *(R30, R32)* |
| **0B-AC-103** | A cross-origin state-changing browser → BFF request lacking valid CSRF or origin integrity is rejected, a legitimate same-origin state-changing request carrying valid controls succeeds, and LOCAL and OIDC receive the same protection. *(R33)* |
| **0B-AC-104** | With at least two concurrently running frontend instances and no sticky routing, either instance serves the same browser session and an OIDC transaction initiated on one completes on another, with no shared frontend-owned durable session state or session authority; the category-B transaction-state mechanism remains unfixed by this specification. *(R34)* |
| **0B-AC-105** | Editing a route URL, client state, browser storage or JavaScript state grants no capability; redirect and deny behaviour is deterministic; a loading, pending or unknown state grants no access; and no product protected route exists. *(R35)* |
| **0B-AC-106** | Against the synthetic session authority, the browser session reference rotates on authentication and re-authentication, a pre-authentication reference cannot be reused afterwards, and the cookie is cleared on sign-out; and no session identifier, raw cookie, access token, refresh token, external identity-provider token, password, reset token, OIDC authorization code, MFA secret, claims payload, stack trace, internal provider response or client secret appears in a log, diagnostic, telemetry field or browser-visible surface. **No real backend token rejection, session invalidation, server-side token-cache invalidation, refresh-token rotation, LOCAL credential verification, federated-token validation, OIDC tenant integration or backend authorization is claimed.** *(R36, R37, R38)* |
| **0B-AC-107** | Responses served by the production artifact carry a restrictive Content Security Policy establishing `object-src 'none'`, no unsafe eval, no arbitrary third-party script source, and a controlled `connect-src` allowlist limited to approved browser-visible destinations — introducing no direct browser → KnowHub-backend destination, exposing no server-only backend origin, drawing only on browser-exposable configuration where configuration participates, and adding no backend origin automatically. Weakening or removing any of these fails the gate. *(R40, R41, R48)* |
| **0B-AC-108** | Responses served by the production artifact carry the application-wide baseline: HSTS, `X-Content-Type-Options`, a strict referrer policy, a `Permissions-Policy`, and an anti-clickjacking control under which unauthorized or unapproved frame embedding is denied. Removing or weakening any control fails the gate. No value and no mechanism is fixed by this specification, and no control the sources do not establish — COOP, COEP, CORP, Subresource Integrity, CSP reporting or a CORS policy — has been introduced. The evidence is at the production artifact and runtime boundary and does not demonstrate deployment-platform configuration, environment-level confirmation remaining deployment work where a control is applied or strengthened at a gateway, ingress, CDN or TLS-termination layer. *(R39, R42, R43, R44, R45, R46, R47, R48, R49)* |
| **0B-AC-109** | No production rendering path passes unsanitized untrusted content through a raw-HTML or unsafe DOM boundary: no path exists by which untrusted content can reach such a boundary without the required sanitization control, and a deliberately introduced one fails the gate. Content that has passed the required allowlist sanitization is not prohibited from being rendered by the capability that introduces it. No sanitization library has been selected, 0B rendering no untrusted rich content, and the allowlist-sanitization obligation is recorded as binding the capability that first renders such content. *(R50, R51)* |
| **0B-AC-110** | Inspection of the production bundle and any emitted source map finds no secret, credential, token, resolved provider credential or server-only configuration value, and no test-only fixture, double, authority or identifier. No source-map policy is required or forbidden; whatever the build emits is inspected. *(R52, R53, R54)* |
| **0B-AC-111** | Installation resolves strictly from the committed lockfile and fails rather than drifting; manifest and lockfile currency is checked mechanically; and a dependency vulnerability gate is present and blocking — each as a blocking gate, with a zero-finding scan result accepted as a pass. No SBOM, provenance or signing tooling, format or registry integration has been selected or introduced, and no license-scanning requirement has been created. *(R55, R56, R57, R60, R61)* |
| **0B-AC-112** | The secret-scanning gate is present and blocking, and is demonstrated against an ephemeral synthetic credential placed into a verification input created for the check and discarded afterwards: the scan fails on that input and passes once it is removed. No real or persistent credential is committed, and no detectable test credential remains once verification completes. *(R58)* |
| **0B-AC-113** | The production build executes as a blocking gate; a strict type error and a lint violation each block it; with dependencies and build inputs available it requires no live KnowHub backend, no real identity provider, no product runtime service and no product OpenAPI contract; and no development-only or test-only code is present in the produced bundle. *(R62, R63, R64, R65)* |
| **0B-AC-114** | The production frontend container image builds from the repository as a blocking gate; the release artifact's identity binds the repository name and the exact commit in the form §0E.8 states, with no registry, publishing step or further tag scheme fixed; a blocking container vulnerability scan covers the image. No registry publishing, deployment definition, Kubernetes or ingress configuration, cloud runtime design, Terraform or Bicep, environment promotion, production identity-provider registration, TLS, CDN or WAF design, or `deploy/` content has been introduced; and no backend container implementation detail — non-root user identifier, health check, multi-stage structure, image labels or exposed-port choice — has been imported or fixed. *(R59, R66, R67, R68, R69, R70)* |
| **0B-AC-115** | The CI contract holds. Every gate in R71 is present and blocking, none is advisory, and no empty test layer or placeholder job has been created in order to have a gate; gates execute on the Node baseline declared under 0B-SPEC-001 R5, evidencing 0B-AC-003; the path runs with only `knowhub-frontend` checked out, with no `knowhub-backend` checkout, source tree, Python package or runtime required in order to lint, test or build, while a legitimate pinned backend-generated OpenAPI artifact governed by 0B-SPEC-003 remains permitted as contract input together with its recorded backend contract provenance; the required container image and the required test and UX reports are produced and are identified as CI evidence artifacts rather than runtime telemetry, no browser telemetry SDK, vendor integration or metrics framework having been introduced; contract-input identity is recorded honestly — where a legitimate pinned backend-generated artifact exists the exact backend contract revision used for the build is recorded and published, and where only the synthetic fixture exists no backend contract revision is fabricated or published, the absence of a bound product backend contract is recorded explicitly, the fixture is identified only as test and pipeline evidence, and no backend compatibility is claimed, evidencing 0B-AC-041; CI outcomes are stated provider-independently with no CI provider fixed; and gate ordering is constrained only where architecturally meaningful. *(R7, R18, R27, R71, R72, R73, R74, R75, R76, R77)* |

**Verification dependency.** This specification's criteria are satisfied by its
own implementation: it is the verification layer, and depends on no successor
specification. 0B-AC-090, 0B-AC-092, 0B-AC-111 and 0B-AC-114 are partly
decidable by inspection; every other criterion is executed by a layer required
here.

**Forward catalogue target.**
[§75](../../../knowhub_master_blueprint.html#acceptance-tests) entries are
affected as follows, and no conclusion recorded by an earlier 0B specification
is changed:

- **AT-035 is owned by this specification and is fully evidenced at 0B** by
  0B-AC-107 and 0B-AC-108, at the production frontend artifact and runtime
  boundary, subject to the scope of R49.
- **AT-027 is fully discharged** — 0B-SPEC-004 0B-AC-066 states the behaviour;
  0B-AC-101 executes it.
- **AT-029 is fully discharged** — 0B-SPEC-004 0B-AC-084 states the behaviour;
  0B-AC-103 executes it.
- **AT-028 remains partial**: 0B-AC-106 executes only the browser half against a
  synthetic authority. Real server session and token-cache invalidation is M1.
- **AT-026 remains partial**: 0B-AC-106 executes only the non-leakage half.
  Central token rejection is M1.
- **AT-037 remains partial**: 0B-AC-112 executes only the negative half — no
  committed credential and no hard-coded privileged credential. The bootstrap
  flow is backend and M1.
- **AT-017 remains partial**: 0B-AC-097 and 0B-AC-098 evidence it only for the
  shell, the primitives and the state-class presentations, as 0B-SPEC-002 §9
  records. It is satisfied by the milestone delivering the workflows it names.
- **AT-025, AT-030, AT-031 and AT-042** gain no evidence here; 0B-AC-105
  establishes a frontend precondition only.
- All remaining §75 entries are backend or product capability at M1 and later.

**Identifier form.** Acceptance criteria carry their full milestone-global
identifiers, `0B-AC-090` through `0B-AC-115`. `0B-AC-116` through `0B-AC-119`
remain reserved and unused; under the 0B allocation rule an unused number inside
a block stays unused and is never reassigned.

## 16. Deferred

| Deferred from 0B-SPEC-005 | Owner |
|---|---|
| SBOM generation, provenance generation and artifact signing, and any tooling, format or registry integration for them | Deployment, release and platform-standard work |
| License scanning | Not a KnowHub requirement at 0B; a future platform standard if one is established |
| Cross-repository compatibility tests, the breaking-change gate before backend release, and the recorded frontend/backend version pair | §81 milestone 9; `knowhub-backend` for the backend half |
| Safari and WebKit browser verification | The point at which the organizational policy §65.4 makes it conditional upon applies |
| Firefox and mobile-browser verification | Not required; a future source if one is established |
| Product end-to-end journeys — account activation and reset, session expiry against a real authority, application navigation, Ask and evidence, source onboarding, traceability, admin RBAC, real access-denied | M1 and later |
| Allowlist sanitization of untrusted rich HTML, Markdown and Mermaid, and its test layer | The capability that first renders untrusted rich content, M1 and later |
| Real backend session invalidation, token-cache invalidation, refresh-token rotation, central token rejection, LOCAL credential verification, federated-token validation and backend authorization, and their verification | M1, `knowhub-backend` |
| Real OIDC tenant and client registration, and deployment, gateway or ingress identity configuration | Deployment and environment configuration, M1 and later |
| Deployment-platform confirmation of the production security-header baseline | Deployment work |
| Registry publishing, `deploy/` content, deployment definitions, Kubernetes or ingress, cloud runtime topology, Terraform or Bicep, environment promotion and §76.1 promotion rules | Post-0B |
| Bundle-size budgets, Core Web Vitals, Lighthouse thresholds, startup-time targets, coverage thresholds and visual-regression testing | Deferred; measurement-driven, and only where an authoritative source establishes one |
| Load and resilience testing | Post-0B |
| Browser telemetry SDK, vendor or UX-metrics framework, and `src/lib/telemetry/` | Deferred; a future ADR when a concrete requirement and technology choice exist |
