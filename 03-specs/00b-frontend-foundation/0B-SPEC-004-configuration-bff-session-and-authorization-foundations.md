# 0B-SPEC-004 — Configuration, BFF session and authorization foundations

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository:** `knowhub-frontend`
- **Status:** Approved
- **§81 0B items owned:** `LOCAL/OIDC BFF session shell`

## 1. Purpose and scope boundary

Establish the frontend security, configuration and session foundation before any
real identity exists: the typed configuration boundary, the authentication-mode
and provider-selection shell, the browser/BFF session boundary, the OIDC
protocol shell, the LOCAL shell, the session state contract, the authorization
foundation, the route and session guard, CSRF and redirect safety, auth and
session safe diagnostics, and the test-only doubles that make all of it
provable.

The milestone constraint this specification is built around: §81 places
**Backend Identity + Application Registry + canonical DB model** at **M1**, after
0B. 0B owns the *shell*. This specification therefore defines a frontend
boundary and its interfaces so M1 can bind real backend identity into it without
redesign, and it invents **no** backend authentication contract, **no** user,
role or permission model, and **no** credential store. Where executable proof is
needed before M1, it comes from clearly test-only doubles that cannot ship as
product authentication (§13).

This is the first 0B specification governed directly by
[ADR-027](../../02-adrs/ADR-027-frontend-bff-session-state-ownership.md), which
postdates 0B-SPEC-001, -002 and -003 and is cited by none of them.

### 1.1 Three boundaries stated once

**Sign-in scope.** This specification owns the **authentication boundary and its
server-side route mechanics only**: sign-in initiation, callback handling,
logout and session-termination mechanics, provider selection, redirect
validation and session establishment and guard flow. It owns **no** rendered
product login screen, **no** provider-chooser UI, **no** identity-provider
branding, **no** credential-entry product UI and **no** user or role
administration. Those remain **M1**, as 0B-SPEC-002 §10 already records. Any
executable proof at 0B uses test-only mechanisms and **must not create a
user-visible product authentication surface** merely to exercise the boundary.

**Session state has two categories, and they are not the same thing.**
**Category A — durable session state** is backend-authoritative and is not
frontend-held in any form. **Category B — short-lived OIDC transaction state**
is the PKCE verifier, `state` and nonce correlation that the §0E.13.3 flow
requires across the authorization-request → callback round trip. §5 and §6 bind
each category separately. Category B is permitted; it must never become
category A.

**Security-header ownership is split.** This specification owns only the
controls whose semantics are coupled to the authentication and session
boundary — cookie security, CSRF and request integrity, Origin and Referer
validation, redirect validation, and auth and session safe logging and
redaction. The **application-wide production HTTP security-header baseline** —
CSP, HSTS, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`
and anti-clickjacking — together with its production and runtime verification,
is owned by **0B-SPEC-005**. That successor ownership is recorded here so
0B-SPEC-001's broader deferral is not silently dropped, and no normative
requirement is duplicated across the two specifications.

### 1.2 Directory placement

0B-SPEC-001 R10 fixes the directories materialized at 0B and assigns
`src/lib/auth/`, auth shell routes in `src/app/` and access components in
`src/components/` to this specification. It lists no configuration directory.
This specification requires that **one** configuration boundary exist (§3.1) and
fixes **no path for it**; wherever it lands, any directory added for it is a
structural change justified under **0B-SPEC-001 R11**, which provides exactly
that mechanism. No amendment to an approved specification is required.

## 2. Sources

**Blueprint**

- [§0E.13 Enterprise identity, authentication, token & session architecture](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, in its entirety: §0E.13.1 authentication modes, §0E.13.2
  identity-provider abstraction, §0E.13.3 unified browser/BFF flow, §0E.13.4
  first-run bootstrap, §0E.13.5 local credential security, §0E.13.6 OIDC
  provider contract, §0E.13.7 token ownership and persistence rules, §0E.13.9
  session lifecycle, §0E.13.10 CSRF, XSS, CORS and browser hardening
- [§0E.14 Authorization architecture, RBAC/ABAC, user & role management](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, for the authoritative-server-side rule only; §0E.14.9's
  identity and access API paths are recorded as a forward constraint in §14 and
  are not requirements here
- [§0E.15 Frontend authentication, authorization & administration implementation](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**. Its "Recommended frontend placement" tree is **recommended**,
  not a normative URL or file contract
- [§0E.16 Security verification / penetration-test readiness](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, authentication/session, OIDC and frontend control-area rows
- [§0E.17.8 UI anti-patterns — explicitly prohibited](../../../knowhub_master_blueprint.html#runtime-architecture)
  — rendering unauthorized modules then disabling their buttons; exposing bearer
  tokens, debug claims or secrets in developer UI
- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**, the authentication and streaming boundary
- [§0E.9 Explicit non-choices for V1](../../../knowhub_master_blueprint.html#runtime-architecture)
  — the standing preference against introducing infrastructure without measured
  need
- [§30A Frontend codebase — knowhub-frontend](../../../knowhub_master_blueprint.html#target-frontend),
  including §30A.1 — route handlers and server actions may implement the minimal
  boundary but must not become an alternative business backend
- [§61A.2 Frontend layering](../../../knowhub_master_blueprint.html#frontend-file-spec),
  [§61A.3 UI state classes](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — the Auth-state and server-state rows,
  [§61A.4 Frontend quality gates](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — the security and E2E rows, the latter naming "configured OIDC/BFF sign-in
  test mode"
- [§61A.1 Route map](../../../knowhub_master_blueprint.html#frontend-file-spec)
  — the normative route map, which contains no `/auth/*` route
- [§65.3 Security, privacy and compliance constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
  and the §65 normative rule
- [§70.5 API behavior rules](../../../knowhub_master_blueprint.html#api-contracts)
  — the safe error structure this specification maps into rather than redefines
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
  — "No frontend privilege"; and §71.4 forbidden dependencies
- [§75 Acceptance criteria & executable test catalogue](../../../knowhub_master_blueprint.html#acceptance-tests)
  — AT-025 through AT-044; the forward catalogue targets are recorded in §15
- [§76 Environment & deployment matrix](../../../knowhub_master_blueprint.html#environment-matrix)
  — the Identity row: synthetic users, optional mocked OIDC, and "No insecure
  bypass committed enabled"
- [§77 Configuration model & feature flags](../../../knowhub_master_blueprint.html#configuration-model)
  — §77.1 configuration domains and §77.2 precedence and the secret-reference
  rule
- [§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph),
  including the parallelization rule and §81.1
- [§84.1, §84.3 and §84.4](../../../knowhub_master_blueprint.html#coding-harness-instructions)

**ADRs** — referenced by number and link only; decision text lives in the
canonical files.

- [ADR-027 — Frontend BFF session state ownership](../../02-adrs/ADR-027-frontend-bff-session-state-ownership.md)
  (primary; this is the first 0B specification governed directly by it)
- [ADR-021 — Authentication is provider-pluggable](../../02-adrs/ADR-021-provider-pluggable-authentication.md)
- [ADR-022 — Hybrid RBAC/ABAC authorization with default deny](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md)
- [ADR-025 — Canonical user model is independent of auth provider](../../02-adrs/ADR-025-canonical-user-model-independent-of-provider.md)
  (constrains §4, §8, §9 and §10 — the canonical session and authorization model
  is provider-independent; the authentication boundary may still branch on
  provider metadata for protocol mechanics, and no frontend user, principal,
  role or permission type is defined here)
- [ADR-023 — Platform Super Admin is not a content bypass](../../02-adrs/ADR-023-super-admin-is-not-a-content-bypass.md)
  (constrains §9 — no frontend administrative bypass)
- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
  (constrains §9 and §14 — identity, session, user, role and permission types
  arrive generated from the pinned contract in M1 and are never hand-authored
  here)
- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
  (constrains §13 — 0B verification requires no backend checkout and no running
  backend)

**Approved specifications**

- [0B-SPEC-001](0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md)
  — R10 (directory inventory, including `src/lib/auth/`), R11 (the mechanism for
  adding a directory), R13 (single provider-composition point), R14 (the
  server/client component policy), R15 (auth, session, token and
  identity-provider protocol logic is server-only **by placement**), R16, R17
  (layering), R20 (no reimplemented authorization logic in TypeScript), R21–R22
  (automated boundary enforcement)
- [0B-SPEC-002](0B-SPEC-002-application-shell-design-system-and-accessibility.md)
  — R17 (an unauthorized navigation item is **absent**), R21 and R26 (the
  forbidden and access-denied **presentation**), R18–R19 (the shell renders with
  no authenticated session; demonstrations use test-only doubles unreachable
  from production), R27 (safe error content)
- [0B-SPEC-003](0B-SPEC-003-openapi-contract-and-api-client-foundation.md)
  — R27 (server-side and browser-safe paths), R32–R33 (the API boundary consumes
  a validated backend origin and reads no environment variable), R37 (no browser
  telemetry system), R38–R42 (the normalized safe error representation), R44
  (streaming bearer credentials remain server-side), R46–R49 (the query client
  and the scope-carrying cache-key convention)

**Interface obligations discharged.** This specification discharges obligations
that approved specifications placed on it by name: **0B-SPEC-003 R32 and R33**
(§3.1 and §3.2 produce the validated backend origin the API boundary consumes,
and own the environment-read, exposure-split and secret rules it disclaims);
**0B-SPEC-003 R27 and R44** (§5 defines what the server-side path injects);
**0B-SPEC-001 R15** (§5, §6, §10 and §12 define what the server-only modules
*do*, R15 having fixed only *where* they may live); **0B-SPEC-001 §8 Deferred**
(configuration model, secret containment, session boundary, provider-neutral
abstraction, route protection, authorization-aware UI, CSRF and safe client
logging are discharged here; the application-wide security-header baseline is
re-deferred to 0B-SPEC-005 in §16); **0B-SPEC-002 R17 and §10 Deferred** (§9
defines the permission semantics and their source, whose rendering R17 owns);
**0B-SPEC-002 R26** (§8 defines what places the application in the forbidden
state that R26 renders).

## 3. Normative requirements — the frontend configuration boundary

### 3.1 The configuration layer

**R1.** `knowhub-frontend` has **one typed configuration boundary**. Behaviour
that varies by environment is expressed as configuration there rather than as
constants scattered through application code.
*(§77 — typed configuration/policy, not scattered constants; §77.1)*

**R2.** That boundary is the **single access point** for configuration. It is
the only place configuration is read from the environment, parsed, validated,
defaulted or type-narrowed.
*(§77; 0B-SPEC-003 R33 — the API boundary performs none of these)*

**R3.** No production module outside the configuration boundary reads an
environment variable directly. Consumers receive already-validated values.
*(§77; 0B-SPEC-003 R32, R33)*

**R4.** This specification **names no environment variable**. Variable names,
casing and grouping are implementation-plan decisions; the blueprint fixes none.
*(§77 — no frontend variable name is fixed anywhere in the blueprint)*

*Implementation guidance (non-normative): where the configuration boundary lives
in the source tree is an implementation-plan decision. 0B-SPEC-001 R10 lists no
configuration directory; adding one is the structural change R11 contemplates,
justified by this specification. Neither the directory nor the module filename is
part of the architectural contract.*

### 3.2 Server-only and browser-exposed configuration

**R5.** Every configuration value is **explicitly declared** either server-only
or browser-exposed. The declaration is visible in the configuration boundary, not
implied by where a value happens to be consumed.
*(§0E.16 — no secrets or source maps containing credentials; §61A.4 security row)*

**R6.** Browser exposure follows an **explicit allowlist**. A value is exposed to
browser code only because it was declared exposable; nothing becomes
browser-visible by accident or by default.
*(§0E.16; §61A.4 security row)*

**R7.** **No server-only configuration value reaches a browser bundle or a source
map.** This includes provider configuration, client credentials and any value
whose declaration is server-only.
*(§0E.16 frontend row — no secrets/source maps containing credentials; §61A.4;
§0E.17.8 — do not expose bearer tokens, debug claims or secrets in developer UI)*

**R8.** The configuration boundary **produces the validated backend origin** that
the API boundary consumes under 0B-SPEC-003 R32. It is supplied as an already
validated value; the API boundary performs no parsing, validation or defaulting
of its own.
*(0B-SPEC-003 R32, R33; §0E.8; §77.1 — Platform domain, API URLs)*

### 3.3 Secrets, references and fail-closed validation

**R9.** Production secret-bearing frontend configuration is represented **only
through a secret or credential reference or identifier**. **Raw production
secret values are not ordinary frontend configuration values**, and resolution
remains **server-side through the environment's approved secret
mechanism/boundary**. This specification **selects and introduces no particular
secret-manager product, SDK, secret name, environment-variable name or
credential**, and **no secret-manager infrastructure is introduced merely for
0B**.
*(§77.2 — secrets are never ordinary settings values; configuration references a
secret/credential identifier resolved through the environment's approved secret
manager; §65.3; §84.1 — do not introduce new infrastructure by default; §84.3)*

**R10.** Missing or invalid **security-sensitive** configuration **fails
closed**, at the earliest safe point, with a message identifying what is missing
or invalid. It does not silently default to a less secure mode, and a
security-critical production default fails closed rather than open.
*(§77.2 — lower scopes must not silently weaken mandatory security controls; §65
normative rule; ADR-022 — default deny)*

**R11.** A narrower configuration scope may make behaviour **stricter**. It must
not silently weaken a mandatory security control.
*(§77.2)*

## 4. Normative requirements — authentication-mode and provider-selection shell

**R12.** The provider model represents **LOCAL, OIDC and HYBRID**. HYBRID means
local authentication and **one or more** OIDC providers enabled simultaneously,
so the model expresses an **enabled provider set**, not a two-valued mode.
*(§0E.13.1 — the three modes and their simultaneity; §77.1 — Identity policy
names LOCAL/OIDC/HYBRID and "enabled providers")*

**R13.** Provider selection is **configuration-driven through one stable
interface**. It is environment and organization policy, not business logic, and
adding a provider later does not replace the session or authorization model.
*(§0E.13.1; ADR-021)*

**R14.** **No provider identity is hard-coded** in a route, component or module.
No page component names Entra or any other specific provider.
*(§0E.15 — "Do not hard-code Entra into page components"; §0E.13.6 — Entra is a
supported configuration, not a hard dependency)*

**R15.** **Canonical session and authorization semantics are provider-neutral.**
There is one canonical session shape regardless of provider — never a
`LocalSession` versus `OidcSession` split — and no downstream canonical
consumer, route guard, permission gate or entitlement decision branches on
provider type. Provider identity is available as a **displayable attribute** of
the session.
*(ADR-025; ADR-027 §5; §0E.15 — show authentication provider/session details;
§61A.3 Auth-state row)*

**R16.** The **authentication boundary may branch on provider metadata where
required for protocol mechanics** — OIDC authorization-request initiation,
callback handling, provider-appropriate logout and provider-specific
discovery. Such protocol branching **never changes entitlement, permission or
canonical identity semantics**, and provider identity is never an authorization
discriminator.
*(§0E.13.9 — invoke upstream IdP logout only when appropriate to the provider;
§0E.13.1 — the LOCAL and OIDC protocol legs differ; §0E.13.3; ADR-025)*

*Implementation guidance (non-normative): §0E.13.2's `IdentityProvider` Protocol
is a **backend** abstraction. This specification defines frontend provider
selection and initiation only, and must not mirror that Protocol into
TypeScript; under ADR-017 the real types arrive generated in M1.*

## 5. Normative requirements — the browser and BFF session boundary

**R17.** The browser receives **only an opaque session reference**: high-entropy
with at least 128 bits of entropy, `HttpOnly`, `Secure`, and governed by an
explicit SameSite policy.
*(§0E.13.7 — Browser session identifier row; §0E.8; ADR-027 §3)*

**R18.** That reference **encodes no permissions, no identity attributes and no
token material**.
*(§0E.13.7; ADR-027 §3)*

**R19.** **No KnowHub access token, KnowHub refresh or session token, or external
identity-provider token is present in browser-reachable storage or state**:
`localStorage`, `sessionStorage`, IndexedDB, a URL path, query string or
fragment, a JavaScript-readable cookie, page props or hydrated state.
*(§0E.13.7; §61A.4 security row; §0E.13.10 XSS row; ADR-021; ADR-027 §2)*

**R20.** No such material appears in a browser-visible log, diagnostic or
telemetry field.
*(§0E.13.9 — no secrets in telemetry; §0E.17.8)*

**R21.** **Browser code never constructs an `Authorization` header for the
KnowHub backend.** The bearer credential exists only on the server side of the
boundary.
*(§0E.13.3 — the bearer flows BFF → FastAPI; §0E.15 — no raw token APIs in
client components; §61A.2; 0B-SPEC-003 R27, R44)*

**R22.** Authenticated backend access occurs **only through the approved
server-side path** 0B-SPEC-003 R27 establishes.
*(0B-SPEC-003 R27; §0E.8; §61A.2)*

**R23.** **Category A — durable session state.** `knowhub-frontend` acquires **no
durable session datastore**: no database, no Redis, no session store and no
frontend-owned durable persistence under any name, including "cache", "session
cache", "token cache" or "warm store". The backend is authoritative for durable
session validity, revocation, invalidation, the internal access-token lifecycle,
the refresh and session lifecycle, and effective authorization.
*(ADR-027 §1, §2; §71.4 — the frontend depends on no PostgreSQL, Redis or Blob;
§0E.9; §84.1 — do not introduce new infrastructure by default)*

**R24.** **Token custody at the boundary.** The KnowHub internal access token is
**request-scoped and server-side only**, obtained by presenting the opaque
browser reference and discarded when the request completes; it is not cached
frontend-side across requests. An external identity-provider token exists
**only transiently during the server-side callback and exchange**, and is never
persisted as frontend session authority. The KnowHub refresh and session token
is **backend authority and is not frontend-held in any form**.
*(§0E.13.7 — the token ownership table; ADR-027 §1, §2)*

**R25.** The boundary requires **no sticky frontend routing**. Any
`knowhub-frontend` instance serves the same browser session, and frontend
instances remain horizontally scalable with no shared frontend-owned session
state.
*(ADR-027 §6; ADR-016)*

## 6. Normative requirements — the OIDC protocol shell

**R26.** Browser sign-in through an OIDC provider uses the **Authorization Code
flow with PKCE**.
*(§0E.13.6; §0E.13.1 OIDC row; §0E.13 preamble; ADR-021)*

**R27.** **The OAuth implicit flow is prohibited.** No implicit-flow code path
exists.
*(§0E.13.6 — "Never use the OAuth implicit flow")*

**R28.** A **`state` value is generated per authorization request and validated
at the BFF on callback**. A mismatched, missing or unrecognised `state` fails
closed. This is request binding at the BFF, not identity validation.
*(§0E.13.6; §0E.13.3 — the callback returns to the BFF; §0E.16 OIDC row)*

**R29.** A **nonce is generated per authorization request and bound to the
pending transaction** at the BFF.
*(§0E.13.6; §0E.16 OIDC row)*

**R30.** **Category B — short-lived OIDC transaction state.** The PKCE code
verifier, `state` and nonce correlation **may span the authorization-request →
callback round trip**, and must satisfy every one of the following. It exists
only for the lifetime of **one authentication transaction**. It **must not become
durable session authority**. It **must not require sticky frontend routing**, and
must work across horizontally scalable frontend instances. It is
**integrity-protected**. Its **confidential components — notably the PKCE code
verifier — remain inaccessible to browser application JavaScript**. It contains
**no durable user, role or permission authority**. It is **cleared or invalidated
when the transaction completes, expires or fails**. It **never contains or
becomes a product session or authorization store**.

This specification **fixes no persistence or transport mechanism for category B
and names no mechanism class**. The implementation plan selects one satisfying
every property above. A mechanism that becomes durable or cross-transaction,
becomes a product session or authorization store, requires sticky routing, or
introduces frontend-owned durable or shared **session or authorization**
persistence is category A, contradicts ADR-027 §2 and §6, and is a §84.4 stop
condition requiring a superseding ADR.
*(§0E.13.3 — the flow spans browser → BFF → IdP → callback → BFF; §0E.13.7 —
"transient/token cache during OIDC flow"; ADR-027 §2, §6; §84.1; §84.4)*

**R31.** The **redirect and callback target is validated against approved
values** before the browser is redirected and on return.
*(§0E.13.6 — provider-specific tenant/domain policy; §0E.13.10 — validate
approved Origin/Referer; R65)*

**R32.** The **authorization response is handled server-side**. No provider
token, authorization code, client credential or PKCE code verifier reaches
browser application code.
*(§0E.13.3 — callback to BFF; §0E.13.7; §0E.15 — no raw token APIs in client
components; §30A — the callback is a server-only route/handler)*

**R33.** Provider configuration and client credentials are **server-only**, and
an authentication failure **fails closed**.
*(§0E.13.7; §3.2 R5–R7; ADR-022 — default deny)*

**R34.** The frontend BFF performs **no issuer, audience or client-ID,
signature or JWKS, expiry or not-before, tenant-domain-policy, or ID-token
nonce-claim validation**, and **declares no federated-exchange contract**. That
validation occurs **at the federated boundary**, which is backend and M1. How a
nonce reaches that boundary is an M1 contract question this specification does
not answer.
*(§0E.16 OIDC row — rejected "at the federated boundary"; §0E.13.3 — FastAPI
identity exchange; §0E.14.9 — the exchange endpoint validates external OIDC
identity; ADR-017; §81)*

*Implementation guidance (non-normative): the OIDC library is an
implementation-plan choice. §0E.13.6 requires a generic standards-based adapter
and names no package. A library that relocated session authority, required
browser-side token custody, introduced a frontend-owned durable session store or
moved entitlement decisions into the frontend would breach ADR-021, ADR-022 or
ADR-027 and would require a superseding ADR under §84.1, triggering §84.4.*

## 7. Normative requirements — the LOCAL BFF shell

**R35.** LOCAL authentication traverses the **same BFF and session path shape**
as OIDC and terminates in the same canonical session abstraction.
*(§0E.13.1; §0E.13.3; ADR-021; ADR-027 §5; ADR-025)*

**R36.** **LOCAL is not a client-side authentication bypass** and creates no
second authorization path. No configuration, build mode or profile turns LOCAL
into a way of skipping the session boundary.
*(§0E.13.1 — LOCAL is a first-class provider; §76 Identity row — "No insecure
bypass committed enabled"; ADR-025; §84.1 — never weaken authorization for
convenience)*

**R37.** The frontend holds **no product credential store** and trusts **no
browser-supplied user, role or permission data**. Credential verification,
credential storage, password policy, MFA, reset and account lifecycle are
backend and M1.
*(§0E.13.1 — the backend verifies local credentials; §0E.13.5; §0E.15 — the
Python API remains authoritative; §81)*

**R38.** **No hard-coded privileged credential exists in any repository
artifact** — no default administrator, no universal username and password, no
development password, and no committed enabled insecure bypass.
*(§0E.13 preamble — "No hard-coded 'default admin' credential"; §0E.13.4 — never
ship a universal username/password; §76 Identity row; §84.3 — no generated
secrets or local tokens may be committed)*

*Implementation guidance (non-normative): §0E.13.4's first-run bootstrap —
`knowhub admin bootstrap` or a one-time bootstrap token — is a backend
capability. It is the blueprint's answer to day-one LOCAL deployment and requires
nothing of the frontend at 0B.*

## 8. Normative requirements — session state and lifecycle reactions

**R39.** The session abstraction represents and distinguishes six states:
**unauthenticated, authenticated, expired, revoked, forbidden and
authentication-error**.
*(§0E.15 — 401 versus 403; §61A.3 Auth-state row; §0E.13.9)*

**R40.** **Unauthenticated initiates the sign-in flow** at the server boundary.
*(§0E.15 — unauthenticated flows redirect to provider-aware sign-in)*

**R41.** **Forbidden reaches the access-denied experience** whose presentation
0B-SPEC-002 R26 owns. This specification defines what places the application in
that state; it defines no presentation.
*(§0E.15 — authenticated-but-unauthorized users see a polished Access Denied
experience; 0B-SPEC-002 R21, R26)*

**R42.** **Security-sensitive uncertainty fails closed.** An indeterminate,
unknown or unresolvable session state does not yield access.
*(ADR-022 — default deny; §65 normative rule; §84.1)*

**R43.** The **browser session reference rotates on authentication and on
re-authentication**, a pre-authentication reference cannot be reused afterwards,
and the cookie is cleared on sign-out.
*(§0E.13.7 — rotation after login/elevation; §0E.13.9 — rotation and logout;
§0E.16 — session fixation tests)*

**R44.** A **session transition invalidates cached server state** through the
query layer 0B-SPEC-003 R47–R49 establish, so no cached result survives across
an authorization scope boundary.
*(§61A.3 server-state row; 0B-SPEC-003 R47, R48, R49; ADR-022)*

**R45.** The **backend remains authoritative** for durable session validity,
revocation and the token lifecycle. The frontend boundary **reacts to
backend-determined outcomes without deciding them**, and **no frontend component,
guard, port or cached value holds session authority**.
*(ADR-027 §1, §4, §7; §0E.15 — the Python API remains authoritative for sessions;
§71.3 — no frontend privilege)*

*Implementation guidance (non-normative): this specification fixes no idle
timeout, absolute lifetime, rotation interval or revocation-convergence latency.
§0E.13.9 states its values as configurable defaults and examples, and ADR-027 §7
defers revocation convergence and its latency to M1.*

## 9. Normative requirements — the authorization foundation

**R46.** **The backend is authoritative for authorization, and authorization is
default-deny.**
*(ADR-022; §0E.14 — every protected backend operation is authorized server-side;
§71.3)*

**R47.** **Frontend authorization controls UX availability only and never
constitutes enforcement.** Hidden or absent UI is never an authorization control,
and no trust decision derives from a hidden UI control.
*(§0E.15 — PermissionGate is not security; §71.3 — hiding a button is UX, never
authorization; §61A.4 security row; 0B-SPEC-001 R20)*

**R48.** **No independently-authored frontend permission taxonomy or
authorization policy model exists**, and **no TypeScript RBAC, ABAC or policy
evaluator independently derives entitlement** from roles, provider claims or
client state.
*(ADR-022; §61A.2 — never copy authorization rules into TypeScript; §0E.15;
0B-SPEC-001 R20)*

**R49.** Effective authorization is consumed through **one permissions port**.
Once M1 supplies the authoritative contract, **generated permission and
effective-authorization identifiers and types may flow through that port**, and
the frontend **may project authoritative effective authorization into UX
visibility and availability**. That projection is a **UX decision only**: the
frontend **does not independently determine or assert entitlement**. **0B
introduces no permission name, role name, DTO or product permission taxonomy**,
and the port is bound to nothing real at 0B.
*(§61A.3 Auth-state row — the browser sees effective-permission state, and the
backend remains authoritative for entitlement; §0E.15 — permissions consume
`/me/permissions` only; ADR-017 — real types arrive generated; §81)*

**R50.** **Unknown, absent or indeterminate effective authorization fails
closed** for protected capability. A capability is not presented as usable
without a positive determination.
*(ADR-022 — default deny; §65 normative rule)*

**R51.** **Editing client state, a route URL, browser storage or JavaScript state
grants no capability.** Frontend state is never the thing that makes an operation
permitted.
*(§0E.15 — direct URL/API access must still be denied by the backend; §71.3;
§0E.16 authorization row)*

**R52.** **Browser-supplied and identity-provider-supplied role or permission
claims are not treated as authority**, and permissions are not inferred from
display role labels absent an authoritative contract that defines the mapping.
*(§0E.13.6 — external claims never bypass source ACL/classification policy;
§0E.13.8 — never trust role, user or permission values supplied by the browser;
ADR-023)*

**R53.** Authorization-sensitive cached state carries the scope dimensions
required so that a cached result is never served across an authorization scope
boundary, consistent with **0B-SPEC-003 R49**.
*(0B-SPEC-003 R48, R49; §61A.3; ADR-022)*

## 10. Normative requirements — the route and session guard

**R54.** **Exactly one canonical route and session guard abstraction exists**,
and protected areas consume it.
*(§0E.15 — route guards; 0B-SPEC-001 R15, R17)*

**R55.** The authentication and session check happens at the **server or BFF
boundary**, not after the browser has received protected content.
*(§30A.1 — route handlers and server actions implement the minimal boundary;
0B-SPEC-001 R14, R15)*

**R56.** Protected content is **not rendered and then hidden after client
hydration** where doing so would disclose protected information.
*(§0E.17.8 — do not render unauthorized modules then merely disable their
buttons; §61A.4 — no trust decisions from hidden UI controls; 0B-SPEC-002 R17)*

**R57.** Redirect and deny behaviour is **deterministic**: the same state yields
the same outcome.
*(§0E.15 — 401 versus 403; §65 normative rule)*

**R58.** **No feature implements its own authentication check, and no feature
reads a cookie or token directly.** Features consume the canonical abstraction.
*(0B-SPEC-001 R15 — server-only by placement; §0E.15 — no raw token APIs in
client components; §61A.2)*

**R59.** A **loading, pending or unknown state never grants access**, and **0B
introduces no product protected route**.
*(ADR-022 — default deny; §61A.1 — product routes belong to M1 and later; §81;
0B-SPEC-001 R16)*

## 11. Normative requirements — CSRF, request integrity and redirect safety

**R60.** **State-changing browser → BFF requests carry request-integrity
protection.**
*(§0E.13.10 — SameSite cookies plus framework CSRF protection/synchronizer token
for state-changing BFF actions)*

**R61.** A **cross-origin** state-changing browser → BFF request lacking valid
required CSRF or origin integrity is **rejected**, while a legitimate
**same-origin** state-changing request carrying valid controls **succeeds**.
*(§0E.13.10; §75 AT-029)*

**R62.** **Approved `Origin` and `Referer` validation applies** to state-changing
requests.
*(§0E.13.10 — validate approved Origin/Referer)*

**R63.** Cookie-based session use **creates no silent CSRF exposure**: the cookie
policy and the request-integrity control are chosen together so that neither
alone is relied upon.
*(§0E.13.10; §0E.13.7 — appropriate SameSite policy)*

**R64.** **LOCAL and OIDC receive the same protection.** No provider, mode or
profile weakens request integrity.
*(§0E.13.1 — authorization is identical regardless of how the user
authenticated; §84.1 — never weaken authorization for convenience)*

**R65.** **Redirect destinations are validated against approved origins and
paths, and open redirects are prohibited** — including post-sign-in return
targets, post-logout targets and OIDC callback targets.
*(§0E.13.10; §0E.13.6; R31)*

*Implementation guidance (non-normative): the exact framework, library, token and
header mechanics are implementation-plan choices that must satisfy §0E.13.10's
required outcome and controls — SameSite cookies, framework CSRF protection or
synchronizer-token protection, and approved Origin and Referer validation. This
specification chooses between no CSRF patterns the sources do not mention, and
the exact SameSite value is likewise an implementation-plan choice: §0E.13.7 says
"appropriate SameSite policy" and fixes no value. This specification does not
extend into CORS policy.*

## 12. Normative requirements — auth and session safe diagnostics

**R66.** Provider failures, token failures, session failures and authorization
failures are **normalized into safe application outcomes**.
*(§0E.13.9; §70.5 — the safe error structure; §0E.14.9 — authentication endpoints
return generic failure messages)*

**R67.** **The §0E.13.9 redaction list binds frontend diagnostics.** No session
identifier, raw cookie, access token, refresh token, external identity-provider
token, password, reset token, OIDC authorization code or MFA secret appears in a
log, diagnostic, telemetry field or browser-visible error.
*(§0E.13.9 — no secrets in telemetry; §0E.16 frontend row)*

**R68.** **Callback and provider error material is normalized, not passed
through.** Raw provider error responses do not reach a rendering surface.
*(§0E.13.9; §0E.14.9 — generic failure messages; §0E.16 — safe security telemetry
without leaking token content)*

**R69.** No **claims payload, stack trace, internal provider response, client
secret or other secret** reaches a browser surface.
*(§0E.17.8 — do not expose bearer tokens, debug claims or secrets in developer
UI; §61A.4; 0B-SPEC-002 R27)*

**R70.** Auth and session failures **map into the normalized safe representation
0B-SPEC-003 R38–R42 owns**. This specification defines **no second error shape
and no error presentation**, and **introduces no browser telemetry system** — no
tracing SDK, no vendor integration and no metrics framework.
*(0B-SPEC-003 R37, R38–R42; 0B-SPEC-002 R27; 0B-SPEC-001 R10 — no
`src/lib/telemetry/`; §70.5)*

## 13. Normative requirements — test-only identity and session doubles

**R71.** A test-only identity and session double **exercises the canonical
production interfaces and state machine** defined by §4 through §12 rather than
bypassing them. Exercising the double proves the boundary, not a parallel path.
*(§61A.4 E2E row — "configured OIDC/BFF sign-in test mode"; §81.1 — failure paths
are exercised; §81 parallelization rule)*

**R72.** The double is enabled **only in an explicit local or test profile**, and
**cannot be selected accidentally by production configuration**.
*(§84.1 — mock authorization only in an explicit local test profile; §76 Identity
row; §77.2 — a lower scope must not silently weaken a mandatory security control)*

**R73.** The double is **unreachable from the production bundle and the
production runtime**, and **no insecure bypass is committed enabled**.
*(§76 Identity row — "No insecure bypass committed enabled"; §84.3; 0B-SPEC-002
R19; 0B-SPEC-001 R15)*

**R74.** The double **defines no product authentication API contract, no product
credential store and no user, role or permission model**, and **possesses no
authorization authority** — it cannot grant real product capability. M1 and the
backend remain authoritative.
*(§81 — Backend Identity is M1; ADR-017 — no hand-authored contract; ADR-027 §1,
§8; §0E.15)*

**R75.** The double supports the **six synthetic states** R39 enumerates and
requires **no `knowhub-backend` checkout and no running backend** to exercise.
*(R39; ADR-016; §81 parallelization rule; §71.3 — independent CI)*

*Implementation guidance (non-normative): where the double and its fixtures live,
how they are wired into a test run, and the harness that executes them are owned
by 0B-SPEC-005 and the implementation plan. This specification owns the
behavioural and containment properties above.*

## 14. Forward constraints recorded, not implemented at 0B-SPEC-004

- **Backend identity and access API paths.** §0E.14.9 fixes literal paths —
  `/api/v1/auth/providers`, `/auth/local/login`, `/auth/federated/exchange`,
  `/auth/refresh`, `/auth/logout`, the password and MFA endpoints, `/api/v1/me`
  and `/me/permissions`, and the `/api/v1/admin/*` families. They are **backend
  contract, owned by M1**, and reach the frontend only as generated types from
  the pinned OpenAPI artifact. **This specification declares none of them as a
  frontend-owned contract** and defines no request or response shape.
  *(§0E.14.9; ADR-017; 0B-SPEC-003; §81)*
- **The recommended frontend placement tree.** §0E.15's
  `src/app/auth/…`, `src/lib/auth/…` and `src/components/access/…` tree is headed
  "Recommended frontend placement", and §30A's is a target diagram. §61A.1, the
  normative route map, contains no `/auth/*` route. They are **guidance, not a
  normative URL or file contract**; this specification fixes no path.
  *(§0E.15; §30A; §61A.1)*
- **Session lifecycle values and revocation convergence.** Idle timeout,
  absolute lifetime, rotation interval, and the mechanism and latency by which
  the frontend boundary converges on a revocation are **M1**. ADR-027 §7 defers
  them explicitly.
  *(§0E.13.9; ADR-027 §7, §8)*
- **Identity policy that is backend policy.** §77.1's Identity policy domain
  includes session and refresh lifetimes, bootstrap disabled state, account lock
  and rate policy, password and reset policy, and provider-group mapping
  behaviour. Those are **backend** policy. The frontend configuration boundary
  owns only what the frontend process needs to operate its BFF, and does not
  become a second home for them.
  *(§77.1; ADR-027 §4)*
- **Effective authorization data.** The permissions port is bound to nothing real
  at 0B. Its payload shape and the generated identifiers and types that flow
  through it come from the **M1 backend contract**.
  *(§0E.15; ADR-017; §81)*
- **Application-wide production HTTP security headers.** CSP, HSTS,
  `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy` and
  anti-clickjacking, and their production and runtime verification, are owned by
  **0B-SPEC-005**. §75 AT-035 tests production frontend responses. This
  specification owns only the session-coupled controls in §11 and §12, and
  duplicates no requirement with 0B-SPEC-005.
  *(§0E.13.10; §0E.16 frontend row; §75 AT-035)*
- **Harnesses, fixtures and CI.** The browser, build-artifact, multi-instance and
  security test layers, the location of test fixtures and every gate's CI wiring
  are owned by **0B-SPEC-005**. Verification by 0B-SPEC-005 does not transfer
  normative ownership of any requirement here.
  *(§0E.16; §61A.4; §75.1)*
- **Two §84.4 stop conditions this specification records in advance.** First, a
  mechanism for category-B transaction state that becomes durable or
  cross-transaction, becomes a product session or authorization store, requires
  sticky routing, or introduces frontend-owned durable or shared session or
  authorization persistence, contradicts ADR-027 §2 and §6 and is new
  infrastructure under §84.1 — stop and surface; a superseding ADR is required. Second, an
  authentication library that relocates session authority, requires browser-side
  token custody or moves entitlement decisions into the frontend breaches
  ADR-021, ADR-022 or ADR-027 — the same rule applies.
  *(§84.1; §84.4; ADR-027)*

## 15. Acceptance criteria

| ID | Criterion |
|---|---|
| **0B-AC-060** | One typed configuration layer exists and is the only place configuration is read from the environment, parsed, validated or defaulted; no production module outside it reads an environment variable directly; behaviour that varies by environment is expressed there rather than as scattered constants; and no environment-variable name is fixed by this specification. *(R1, R2, R3, R4)* |
| **0B-AC-061** | Every configuration value is declared server-only or browser-exposed and browser exposure is by allowlist; secret-bearing configuration is expressed as a secret or credential reference or identifier rather than a raw secret settings value, and no raw secret value is exposed as an ordinary frontend configuration value, resolution remaining server-side; the validated backend origin is produced for the API boundary; and no server-only value, resolved secret, provider credential or token appears in a browser bundle or source map. *(R5, R6, R7, R8, R9)* |
| **0B-AC-062** | Missing or invalid required security-sensitive configuration fails closed at the earliest safe point with a message identifying what is wrong; it does not silently default to a less secure mode; a security-critical production default fails closed; and a narrower scope cannot silently weaken a mandatory security control. *(R10, R11)* |
| **0B-AC-063** | Provider selection is configuration-driven through one stable interface and expresses an enabled provider set, so LOCAL, OIDC and HYBRID — including more than one simultaneously enabled OIDC provider — are all representable; and no provider identity is hard-coded in any route, component or module. *(R12, R13, R14)* |
| **0B-AC-064** | Canonical session and authorization semantics are provider-neutral: one canonical session shape exists with no `LocalSession`/`OidcSession` split, and no downstream canonical consumer, route guard, permission gate or entitlement decision consults provider type. Provider-conditional behaviour appears only inside the authentication boundary for protocol mechanics — initiation, callback, provider-appropriate logout, discovery — and changes no entitlement, permission or canonical identity semantics. Provider identity remains available as a displayable attribute. *(R15, R16)* |
| **0B-AC-065** | The browser receives only an opaque session reference of at least 128 bits of entropy, `HttpOnly`, `Secure` and governed by an explicit SameSite policy, encoding no permissions, no identity attributes and no token material. *(R17, R18)* |
| **0B-AC-066** | No KnowHub access token, KnowHub refresh or session token, or external identity-provider token is present in `localStorage`, `sessionStorage`, IndexedDB, a URL path, query string or fragment, a JavaScript-readable cookie, page props, hydrated state, a browser-visible log, diagnostic or telemetry field. *(R19, R20)* |
| **0B-AC-067** | No browser code constructs an `Authorization` header for the KnowHub backend, and authenticated backend access occurs only through the approved server-side path. *(R21, R22)* |
| **0B-AC-068** | No durable frontend session datastore exists under any name — database, Redis, session store, or frontend-owned durable persistence called a cache, session cache, token cache or warm store. The KnowHub internal access token is request-scoped and server-side only and is not cached across requests; an external identity-provider token exists only transiently during the server-side callback and exchange; no KnowHub refresh or session token is held frontend-side. *(R23, R24)* |
| **0B-AC-069** | No sticky frontend routing is required: a second frontend instance serves the same browser session identically, **and completes an OIDC transaction initiated on the first instance**, with no shared frontend-owned session state. *(R25, R30)* |
| **0B-AC-070** | The OIDC shell uses the Authorization Code flow with PKCE; no implicit-flow code path exists; a `state` value is generated per authorization request and validated at the BFF on callback, with a mismatched, missing or unrecognised value failing closed; and a nonce is generated and bound to the pending transaction per authorization request. *(R26, R27, R28, R29)* |
| **0B-AC-071** | OIDC transaction state survives the authorization-request → callback round trip while remaining transaction-scoped: it is integrity-protected; its confidential components, notably the PKCE code verifier, are inaccessible to browser application JavaScript; it carries no durable user, role or permission authority; it is cleared or invalidated on completion, on expiry and on failure; and it is not reusable as a product session or authorization store. No mechanism class is mandated by this specification. *(R30)* |
| **0B-AC-072** | The authorization response is handled server-side; no provider token, authorization code, client credential or PKCE code verifier reaches browser application code; provider configuration and client credentials are server-only; and an authentication failure fails closed. *(R32, R33)* |
| **0B-AC-073** | The frontend BFF performs no issuer, audience or client-ID, signature or JWKS, expiry or not-before, tenant-policy or ID-token nonce-claim validation, and declares no federated-exchange contract; no backend authentication endpoint request or response shape is declared anywhere in this specification's implementation. *(R34)* |
| **0B-AC-074** | A redirect or callback target outside the approved set is rejected — including post-sign-in return targets, post-logout targets and OIDC callback targets — and no open redirect exists. *(R31, R65)* |
| **0B-AC-075** | LOCAL traverses the same BFF and session path shape as OIDC and terminates in the same canonical session abstraction; it creates no second authorization path and no configuration, build mode or profile turns it into a way of skipping the session boundary; the frontend holds no product credential store; and no browser-supplied user, role or permission data is trusted. *(R35, R36, R37)* |
| **0B-AC-076** | No hard-coded privileged credential exists in any repository artifact: no default administrator, no universal username and password, no development password, and no committed enabled insecure bypass. *(R38)* |
| **0B-AC-077** | Unauthenticated, authenticated, expired, revoked, forbidden and authentication-error are each represented and distinguishable; the unauthenticated state initiates the sign-in flow at the server boundary; and the forbidden state reaches the access-denied experience 0B-SPEC-002 renders, this specification defining no presentation. *(R39, R40, R41)* |
| **0B-AC-078** | Security-sensitive uncertainty fails closed, and no frontend component, guard, port or cached value holds session authority: given a test-only authority reporting a session invalid, expired or revoked, the boundary reflects that outcome and grants nothing, and no frontend path re-establishes access without a fresh determination from the authority. *(R42, R45)* |
| **0B-AC-079** | The browser session reference rotates on authentication and on re-authentication; a pre-authentication reference cannot be reused afterwards; the cookie is cleared on sign-out; and a stale reference is rejected by the synthetic session authority. **Real backend session invalidation, real server-side token-cache invalidation and real refresh-token rotation are not claimed at 0B.** *(R43)* |
| **0B-AC-080** | A session transition invalidates cached server state through the query layer, and no cached result is served across an authorization scope boundary. *(R44, R53)* |
| **0B-AC-081** | Frontend authorization controls UX availability only and never constitutes enforcement, with the backend authoritative and default-deny: exactly one permissions port exists; unknown, absent or indeterminate effective authorization fails closed for protected capability; no independently-authored frontend permission taxonomy or authorization policy model exists, and no TypeScript RBAC, ABAC or policy evaluator independently derives entitlement from roles, provider claims or client state; and a capability suppressed in the UI is suppressed without independently deriving or asserting entitlement — UX availability is a projection of authoritative effective authorization received through the permissions port. At 0B the port is bound to nothing real and no permission name, role name, DTO or product permission taxonomy is introduced; once M1 supplies the authoritative contract, generated permission and effective-authorization identifiers and types flow through the port unchanged. *(R46, R47, R48, R49, R50)* |
| **0B-AC-082** | Editing client state, a route URL, browser storage or JavaScript state grants no capability; and browser-supplied and identity-provider-supplied role and permission claims are not treated as authority, with no permission inferred from a display role label. *(R51, R52)* |
| **0B-AC-083** | Exactly one route and session guard abstraction exists and protected areas consume it; the check happens at the server or BFF boundary; no feature implements its own authentication check and no feature reads a cookie or token directly; protected content is not disclosed by rendering then hiding after client hydration; redirect and deny behaviour is deterministic; a loading, pending or unknown state grants no access; and no product protected route exists. *(R54, R55, R56, R57, R58, R59)* |
| **0B-AC-084** | A cross-origin state-changing browser → BFF request lacking valid required CSRF or origin integrity is rejected, while a legitimate same-origin state-changing request carrying valid controls succeeds; approved `Origin` and `Referer` validation applies; cookie-based session use creates no silent CSRF exposure; and LOCAL and OIDC receive the same protection. *(R60, R61, R62, R63, R64)* |
| **0B-AC-085** | No session identifier, raw cookie, access token, refresh token, external identity-provider token, password, reset token, OIDC authorization code, MFA secret, claims payload, stack trace, internal provider response or client secret reaches a log, diagnostic, telemetry field or browser surface; provider, token, session and authorization failures surface only as safe outcomes, with callback and provider error material normalized rather than passed through; those outcomes map into the normalized representation 0B-SPEC-003 owns, no second error shape and no error presentation being defined here; and no browser telemetry system is introduced. *(R66, R67, R68, R69, R70)* |
| **0B-AC-086** | The test-only identity and session double satisfies its containment contract: it exercises the canonical production interfaces and state machine rather than bypassing them; it is enabled only in an explicit local or test profile and cannot be selected accidentally by production configuration; it is unreachable from the production bundle and the production runtime, with no insecure bypass committed enabled; it defines no product authentication API contract, no product credential store and no user, role or permission model; it possesses no authorization authority and cannot grant real product capability; it supports the six synthetic states; and it requires no `knowhub-backend` checkout and no running backend to exercise. *(R71, R72, R73, R74, R75)* |

**Verification dependency.** 0B-AC-060, 0B-AC-062, 0B-AC-063, 0B-AC-064,
0B-AC-068, 0B-AC-070, 0B-AC-073, 0B-AC-075, 0B-AC-077, 0B-AC-078, 0B-AC-080 and
0B-AC-081 are decidable by inspection or by a component test layer. 0B-AC-061,
0B-AC-065, 0B-AC-066, 0B-AC-067, 0B-AC-069, 0B-AC-071, 0B-AC-072, 0B-AC-074,
0B-AC-076, 0B-AC-079, 0B-AC-082, 0B-AC-083, 0B-AC-084, 0B-AC-085 and 0B-AC-086
additionally need a browser, build-artifact, multi-instance or security test
layer to execute. That layer, the location of its fixtures and its CI wiring are
owned by **0B-SPEC-005**; the requirements they check remain owned here. This is
a verification dependency at milestone level, not a transfer of ownership.

**Forward catalogue target.** Criteria here contribute to §75 entries without
discharging more than 0B can honestly prove. **0B-AC-066 fully discharges
AT-027** and **0B-AC-084 fully discharges AT-029**. **0B-AC-079 discharges only
the browser half of AT-028** — reference rotation, fixation resistance and cookie
clearing against a synthetic authority; real server session and token-cache
invalidation is M1, and AT-028 is **not** satisfied at 0B. **0B-AC-085 discharges
only the non-leakage half of AT-026**; central token rejection is M1.
**0B-AC-076 discharges only the negative half of AT-037**; the bootstrap flow
itself is backend. 0B-AC-064, 0B-AC-082 and 0B-AC-073 establish frontend
preconditions for AT-042, AT-030, AT-031 and AT-025 without discharging them.
**AT-035 is owned by 0B-SPEC-005.** AT-033, AT-034, AT-036, AT-038, AT-039,
AT-040, AT-041, AT-043 and AT-044 are M1 and backend.

**Identifier form.** Acceptance criteria carry their full milestone-global
identifiers, `0B-AC-060` through `0B-AC-086`. `0B-AC-087` through `0B-AC-089`
remain reserved and unused; under the 0B allocation rule an unused number inside
a block stays unused and is never reassigned.

## 16. Deferred

| Deferred from 0B-SPEC-004 | Owner |
|---|---|
| Canonical user, principal, role and permission model; identity persistence | M1 |
| Backend authentication, session, token, refresh and logout endpoints and their contracts | M1, `knowhub-backend` |
| Backend identity-provider adapters and the `IdentityProvider` Protocol | M1, `knowhub-backend` |
| Backend federated identity validation and exchange: issuer, audience, signature and JWKS, expiry and not-before, tenant policy, ID-token nonce claim | M1, `knowhub-backend` |
| LOCAL credential store, Argon2id hashing, pepper, password policy, login rate limiting and abuse resistance | M1, `knowhub-backend` |
| First-run bootstrap, one-time bootstrap token and the bootstrap-disabled state | M1, `knowhub-backend` |
| MFA implementation, password reset, account activation, account recovery and invitation or onboarding workflows | M1 and later |
| Application registry | M1 |
| Backend authorization engine, RBAC and ABAC evaluation, source ACL and classification enforcement | M1 and later |
| Durable session authority, session revocation, administrative revoke, token lifecycle, refresh rotation, and revocation-convergence mechanism and latency | M1 (ADR-027 §7, §8) |
| Session idle timeout, absolute lifetime and rotation interval values | M1 |
| Enterprise directory and group mapping, role assignment workflow, permission administration | M1 and later |
| Real OIDC tenant and client registration, and deployment, gateway or ingress identity configuration | Deployment and environment configuration, M1 and later |
| Product login page, provider-chooser UI, identity-provider branding, credential-entry UI, and user and role administration screens | M1 (0B-SPEC-002 §10) |
| Product protected routes and modules, product authorization policies, real authenticated product journeys | M1 and later; end-to-end integration at §81 milestone 9 |
| Effective-authorization payload shape and generated permission identifiers and types | M1, through the pinned OpenAPI contract (ADR-017) |
| Application-wide production HTTP security headers — CSP, HSTS, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, anti-clickjacking — and their production and runtime verification | 0B-SPEC-005 |
| Browser, build-artifact, multi-instance and security test harnesses, test-fixture location and CI wiring | 0B-SPEC-005 |
| Browser telemetry SDK, vendor or UX-metrics framework, and `src/lib/telemetry/` | Deferred; a future ADR when a concrete requirement and technology choice exist |
