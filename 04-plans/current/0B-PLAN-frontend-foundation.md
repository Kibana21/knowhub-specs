# 0B implementation plan — Frontend foundation (`knowhub-frontend`)

- **Milestone:** 0B (§81 milestone 0B — Frontend foundation)
- **Repository implemented:** `knowhub-frontend`
- **Status:** Current
- **Specifications implemented:** 0B-SPEC-001 – 0B-SPEC-005, all Approved
- **Tasks:** [`0B-tasks/`](0B-tasks/README.md) — `0B-T01` – `0B-T32`

---

## 1. Purpose and scope

This plan states **how** milestone 0B is built: in what order, with what tools,
in which files, and with what evidence. It introduces **no requirement**. Every
requirement reference points at an Approved 0B specification; every acceptance
criterion is one of the 102 allocated `0B-AC-nnn` identifiers.

Precedence, without exception:

> **Approved specification → Accepted ADR → this plan → a task file.**

Where a task and this plan disagree, the plan wins. Where this plan and a
specification disagree, **the specification wins and implementation stops** until
the conflict is surfaced (§84.4).

0B delivers the frontend **foundation**: a repository that installs, lints,
formats, type-checks, tests, builds and containerises on its own; a bright design
system and data-agnostic shell; a reproducible OpenAPI generation pipeline proved
without a backend contract; a LOCAL/OIDC BFF session boundary proved without real
identity; and a blocking CI gate set. It delivers **no product capability**.

## 2. Authoritative inputs

**Approved specifications** — the contract for this plan:

| Spec | Owns | ACs |
|---|---|---|
| [0B-SPEC-001](../../03-specs/00b-frontend-foundation/0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md) | repository, toolchain, layout, App Router architecture, module boundaries, quality-tooling outcomes, repository independence | 0B-AC-001–013 |
| [0B-SPEC-002](../../03-specs/00b-frontend-foundation/0B-SPEC-002-application-shell-design-system-and-accessibility.md) | tokens, primitives, shell, UI state classes, safe error presentation, accessibility, responsive behaviour | 0B-AC-020–036 |
| [0B-SPEC-003](../../03-specs/00b-frontend-foundation/0B-SPEC-003-openapi-contract-and-api-client-foundation.md) | contract governance, generation pipeline, freshness, the single API boundary, correlation, error normalization, streaming boundary, server state | 0B-AC-040–058 |
| [0B-SPEC-004](../../03-specs/00b-frontend-foundation/0B-SPEC-004-configuration-bff-session-and-authorization-foundations.md) | configuration boundary, provider selection, browser/BFF session boundary, OIDC shell, session lifecycle, authorization foundation, guard, CSRF, redirects, safe diagnostics, test doubles | 0B-AC-060–086 |
| [0B-SPEC-005](../../03-specs/00b-frontend-foundation/0B-SPEC-005-testing-security-production-build-and-ci.md) | test architecture, test-only isolation, static gates, component/a11y/contract/browser/multi-instance/security layers, production browser hardening, supply chain, production build and container, CI gate set | 0B-AC-090–115 |

Throughout this plan and every task file, **`S1`–`S5`** abbreviate
0B-SPEC-001 – 0B-SPEC-005, and `S4 R30` means requirement R30 of 0B-SPEC-004.

**Accepted ADRs** applicable to 0B, read from the
[live register](../../02-adrs/README.md):

| ADR | Bearing on 0B |
|---|---|
| [ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md) | two independently versioned repositories; independent CI |
| [ADR-017](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md) | OpenAPI is the only integration boundary; generate, never hand-copy |
| [ADR-021](../../02-adrs/ADR-021-provider-pluggable-authentication.md) | provider-pluggable authentication behind a minimal BFF |
| [ADR-022](../../02-adrs/ADR-022-hybrid-rbac-abac-authorization-default-deny.md) | backend-authoritative authorization, default deny |
| [ADR-023](../../02-adrs/ADR-023-super-admin-is-not-a-content-bypass.md) | no frontend administrative bypass |
| [ADR-025](../../02-adrs/ADR-025-canonical-user-model-independent-of-provider.md) | one canonical, provider-independent session model |
| [ADR-026](../../02-adrs/ADR-026-adr-ownership-and-location.md) | ADR ownership and the register |
| [ADR-027](../../02-adrs/ADR-027-frontend-bff-session-state-ownership.md) | **primary** — backend owns durable session; the BFF is a boundary, not an authority |
| ADR-024 | premium bright evidence-first UI; canonical file in `knowhub-frontend/docs/adr/` |

**Blueprint** — §0E (all subsections), §30A/.1/.2, §61A/.1–.4, §65, §69, §70.5,
§71.3/.4, §75/§75.1, §76/§76.1, §77.1/.2, §81/§81.1, §84.1–§84.4.

**Structural precedent only:** the M00 plan, task set and verification report are
used for *document shape*. No backend implementation decision is inherited.

**Not consulted:** `terrain-main`, `deepwiki-rs-main`. No Approved 0B requirement
needs behavioural evidence from them.

## 3. Repository baseline

Verified at authoring time; all three working trees clean.

| Repository | HEAD | Branch | Tree |
|---|---|---|---|
| `knowhub-specs` | `b0ed3a0c` | `main` | clean |
| `knowhub-frontend` | `b2787a57` | `main` | clean |
| `knowhub-backend` | `bdb411b0` | `main` | clean |

`knowhub-frontend` contains exactly `README.md` (empty) and
`docs/adr/ADR-024-premium-bright-evidence-first-ui.md`. There is no manifest, no
source, no test and no CI. 0B is a greenfield build on one committed ADR.

**One backend fact was checked**, because S3 R7 requires knowing whether an
authoritative published contract exists: `knowhub-backend/docs/api/` contains no
contract artifact and its README records that none is published. Therefore the
**S3 R7 normal path holds** — `contracts/knowhub-openapi.json` does not exist and
none may be manufactured. Backend implementation details are **not** normative
frontend contracts and nothing else in `knowhub-backend` informs this plan.

A second consequence follows. No authoritative backend-published contract or
documented transport binding exists for correlation, so under **S3 R36 the
product correlation binding is unbound at 0B**. A correlation header name
appearing in backend implementation source is expressly not a source this plan
may adopt.

## 4. Explicit milestone boundaries

**In scope:** everything the five Approved specifications require, and nothing
else.

**Out of scope at 0B** — deferred with the owner each specification names:
application registry; Portfolio or any product functionality; real users, roles
or permissions; user- or role-management screens; LOCAL credential storage,
hashing, password policy, MFA, reset, activation, onboarding or first-admin
bootstrap; real enterprise OIDC tenant or client integration; backend
authentication, session or token endpoints; Ask, Sources, Evidence, Architecture,
Graph, Traceability, Impact; real application navigation; the first real product
OpenAPI contract; streaming; browser telemetry and `src/lib/telemetry/`;
deployment-platform work and `deploy/`; registry publishing; SBOM, provenance,
signing, licence scanning; Safari/WebKit, Firefox and mobile verification;
coverage thresholds, bundle budgets, Core Web Vitals, Lighthouse, visual
regression.

**The 0B root screen is a foundation demonstration surface, not a delivered
Portfolio implementation.** It renders the shell frame with no product navigation
entry and an explicit statement that no application capability exists at this
milestone. The file says so in its own header comment so it cannot later be
mistaken for §61A.1's Portfolio route.

## 5. Non-negotiable specification and ADR constraints

Binding on every task. Each is a restatement of an Approved source, not a new
rule.

1. Application and runtime logic is TypeScript/React/Next.js only; no Python, no
   backend implementation, no transliterated backend model. *(S1 R1)*
2. pnpm plus a committed lockfile is the single resolution path; an install that
   would change resolution fails. *(S1 R2–R4)*
3. Only directories carrying real 0B content are materialized. No
   `src/lib/telemetry/`. No product route or product component directory.
   *(S1 R8–R11)*
4. Exactly one root layout and one provider-composition point. *(S1 R12–R13)*
5. Authentication, session, token and identity-provider protocol logic is
   **server-only by placement** and unreachable from any client bundle.
   *(S1 R15)*
6. §61A.2 layering is normative and a violation fails CI automatically.
   *(S1 R17–R22)*
7. No source-level sharing with `knowhub-backend`; OpenAPI is the only
   integration boundary. *(S1 R29–R30; ADR-016; ADR-017)*
8. Bright and light only. No dark theme, no theme switcher, no contained dark
   code surface at 0B. *(S2 R6–R7; ADR-024)*
9. An unauthorized navigation item is **absent**, never rendered-and-disabled;
   hiding is UX, never authorization. *(S2 R17; §71.3)*
10. The 0B shell is data-agnostic, and no fabricated application, user,
    permission or navigation data exists in production source. *(S2 R18–R19)*
11. WCAG 2.2 AA binds the shell, primitives and state classes. *(S2 R30–R43)*
12. No product contract may be manufactured; the synthetic fixture is the
    mandatory pipeline demonstration and can never be promoted into `contracts/`.
    *(S3 R4–R12, R25)*
13. **Pipeline verification is not compatibility verification.** No artifact,
    gate result or document may imply backend compatibility, endpoint existence
    or a version pair. *(S3 R8; S5 R7, R27)*
14. Exactly one API boundary, with distinct server-side and browser-safe paths
    and no product operation at 0B. *(S3 R26–R31)*
15. One typed configuration boundary is the only place environment values are
    read, parsed, validated or defaulted; browser exposure is by explicit
    allowlist; secrets are references; security-sensitive configuration fails
    closed. *(S4 R1–R11)*
16. The browser holds only an opaque ≥128-bit `HttpOnly`/`Secure`/SameSite-governed
    reference. No KnowHub access, refresh or session token and no
    identity-provider token exists in browser-reachable storage. No frontend
    durable session datastore under any name. No sticky routing. The backend is
    the authority. *(S4 R17–R25, R45; ADR-027)*
17. **Category A ≠ Category B.** Short-lived OIDC transaction state is permitted
    and must satisfy every one of S4 R30's eight properties. Anything that
    becomes durable or cross-transaction, becomes a product session or
    authorization store, or requires sticky routing is a §84.4 stop condition
    requiring a superseding ADR. *(S4 R30, §14)*
18. The BFF declares **no** federated-exchange contract and **no** backend
    authentication endpoint request or response shape. *(S4 R34; 0B-AC-073)*
19. One permissions port, bound to nothing real; no permission name, role name,
    DTO or taxonomy at 0B; unknown or indeterminate fails closed.
    *(S4 R48–R50; ADR-022)*
20. One canonical route and session guard, checked at the server/BFF boundary;
    no product protected route exists. *(S4 R54–R59)*
21. Test-only material is structurally unreachable from the production bundle and
    runtime; explicit test enablement is an **additional**, never the sole,
    protection. *(S5 R9–R12)*
22. Every required gate blocks and is demonstrably capable of failing.
    *(S5 R6, R17, R72)*
23. Chromium is the mandatory blocking browser engine; Firefox and mobile are not
    required; Safari/WebKit stays conditional on the §65.4 policy. *(S5 R29)*
24. The security-header baseline is verified against the production artifact.
    No COOP, COEP, CORP, Subresource Integrity, CSP reporting or CORS policy.
    *(S5 R39–R49)*
25. No backend container implementation detail is imported and no
    deployment-platform work is introduced. *(S5 R69–R70)*
26. No SBOM, provenance, signing or licence scanning. *(S5 R61)*

## 6. Reversible implementation choices

Everything in §7 and §§10–24 that this plan **selects** is a reversible
implementation choice unless the table marks it source-fixed. Changing one
requires no specification amendment and no ADR.

**Reversible here:** package versions and version ranges; file and directory
names below the level S1 R10 fixes; environment-variable names; scanner, linter
and formatter products; test file layout; CI provider, workflow filename, job and
step names; Docker stage structure and base image; header *values* the sources
leave open; cookie names; the fixture filename; the correlation binding used for
tests.

**Source-fixed, and therefore not this plan's to change:** the directory
*inventory* of S1 R10; ESLint as the lint tool (S1 R23); Vitest as the component
layer (S5 R19); Playwright as the browser layer (S5 R28); Chromium as the
blocking engine (S5 R29); TanStack Query as the server-state foundation
(S3 R46); the `knowhub-frontend:<git-sha>` artifact identity form (S5 R67);
`object-src 'none'`, absence of unsafe eval, absence of arbitrary third-party
script sources and a controlled `connect-src` (S5 R40).

**A package version, a filename, a test mechanism or a CI topology is never an
architectural requirement.** No task file may present one as such.

## 7. Dependency decisions and pinned toolchain

Selected by one rule: **the newest stable version inside the range that the whole
set mutually supports** — never the highest number available. Every peer range
and engine constraint below was verified mechanically against the registry; the
set resolves with **no unmet peer**.

### 7.1 Two compatibility-driven pins, recorded so a later upgrade is routine

**TypeScript 5.9.3, not 7.0.2.** TypeScript 7.0.2 is the current `latest`, but
`typescript-eslint@8.70.0` declares `typescript >=4.8.4 <6.1.0` and
`openapi-typescript@7.13.0` declares `typescript ^5.x`. Type-aware linting
enforces S1 R21/R25 and S5 R15/R16; generation is the subject of S3 R13–R18.
5.9.3 is the newest version inside the mutually supported range.
**Upgrade trigger:** both packages declaring support — a dependency bump, not an
architectural decision.

**Vitest 4.1.11, not 5.0.1.** Vitest 5.0.0 shipped 2026-09-03 and 5.0.1 on
2026-09-15. 4.1.11 is the settled line and accepts Vite ^6/^7/^8. A foundation
repository whose purpose is a stable blocking gate set does not adopt a
two-week-old major. **Upgrade trigger:** a 5.x line with a month of quiet.

### 7.2 Pinned set

| Package | Version | Role |
|---|---|---|
| node | **24.21.0** | runtime baseline — Active LTS; declared in `.nvmrc`, `engines`, CI and Dockerfile |
| pnpm | **12.4.2** | package manager, pinned through Corepack `packageManager` |
| `next` | 16.3.5 | App Router, production build, standalone output |
| `react` / `react-dom` | 19.3.0 | UI runtime |
| `typescript` | 5.9.3 | strict compiler — see §7.1 |
| `@types/node` | 24.13.5 | Node 24 types |
| `@types/react` / `@types/react-dom` | 19.3.0 | React types |
| `eslint` | 10.10.0 | lint gate |
| `typescript-eslint` | 8.70.0 | type-aware rules |
| `eslint-config-next` | 16.3.5 | framework rules |
| `eslint-plugin-react-hooks` | 7.1.1 | hook correctness |
| `eslint-plugin-boundaries` | 7.2.0 | §61A.2 layering and server-only placement |
| `globals` | 17.12.0 | flat-config environments |
| `prettier` | 3.9.7 | mechanical formatting |
| `prettier-plugin-tailwindcss` | 0.8.1 | deterministic class ordering |
| `tailwindcss` / `@tailwindcss/postcss` | 4.3.3 | token system via `@theme` |
| `postcss` | 8.5.28 | Tailwind pipeline |
| `@tanstack/react-query` | 5.103.1 | server state |
| `zod` | 4.6.5 | configuration schema |
| `jose` | 6.2.12 | AEAD sealing of category-B transaction state |
| `openid-client` | 6.8.8 | generic standards-based OIDC, server-only |
| `server-only` | 0.0.1 | client-bundle poison pill |
| `lucide-react` | 1.47.0 | the one icon family (§0E.17.1) |
| `geist` | 1.7.2 | self-hosted Geist via the package's own `geist/font/sans` export — 45 bundled `woff2` files, no Google-font build fetch |
| `clsx` / `tailwind-merge` / `class-variance-authority` | 2.1.1 / 3.7.0 / 0.7.1 | primitive variant plumbing |
| `@radix-ui/react-dialog` | 1.1.23 | accessible primitive behaviour |
| `@radix-ui/react-select` | 2.3.7 | accessible primitive behaviour |
| `@radix-ui/react-checkbox` | 1.3.11 | accessible primitive behaviour |
| `@radix-ui/react-radio-group` | 1.4.7 | accessible primitive behaviour |
| `@radix-ui/react-slot` | 1.3.3 | accessible primitive behaviour |
| `@radix-ui/react-label` | 2.1.15 | accessible primitive behaviour |
| `openapi-typescript` / `openapi-fetch` | 7.13.0 / 0.17.0 | the §0E.8-named generator pair |
| `vitest` | 4.1.11 | component, unit and contract runner |
| `vite` / `@vitejs/plugin-react` | 8.3.0 / 6.1.1 | Vitest transform |
| `jsdom` | 30.1.0 | component DOM — **requires Node `^24.15.0`**, which 24.21.0 satisfies and 24.13.x does not |
| `@testing-library/react` / `/dom` / `/jest-dom` / `/user-event` | 16.3.3 / 10.4.2 / 7.0.1 / 14.6.7 | rendering and interaction |
| `axe-core` | 4.13.0 | the single accessibility engine |
| `@axe-core/playwright` | 4.13.0 | layout-aware accessibility pass |
| `@playwright/test` | 1.63.0 | browser, E2E and security-negative layer |

**Deliberately excluded**, to keep the footprint minimal (§84.1): coverage and UI
packages for Vitest (S5 §14 defers coverage thresholds); React Query devtools (a
browser debug surface, at odds with §0E.17.8); `iron-session` (a library framed
as a *session* store would blur the ADR-027 category A/B line — see §14); form
libraries (0B submits no product form); any sanitization library (S5 R51 forbids
selecting one); any SBOM, provenance, signing or licence tool (S5 R61); any
browser telemetry SDK (S1 R9; S3 R37; S4 R70; S5 R75).

### 7.3 `package.json` scripts

```
dev · build · start
lint · lint:fix · format · format:check · typecheck
test · test:watch · test:e2e
generate:api        node scripts/generate-api-client.mjs     (the §0E.8 / §30A.2 name)
contract:verify     node scripts/verify-contract-freshness.mjs
inspect:artifact    node scripts/inspect-build-artifact.mjs
verify:headers      node scripts/verify-security-headers.mjs
evidence:contract   node scripts/write-contract-evidence.mjs
verify              lint && format:check && typecheck && test && contract:verify && build
```

`pnpm verify` is the single local sequence `README.md` documents for 0B-AC-013.

## 8. Planned final repository structure

Only what is expected to exist after 0B. No empty directory mirrors a future
milestone.

```
knowhub-frontend/
├── .dockerignore  .env.example  .gitignore  .npmrc  .nvmrc  .prettierignore
├── Dockerfile                                 production image            S5 R66
├── README.md                                  repo + `pnpm verify`        S1 R27
├── contract-generation.config.json            explicit generation I/O     S3 R14
├── eslint.config.mjs                          lint, layering, server-only, unsafe-DOM
├── next.config.ts  package.json  pnpm-lock.yaml  pnpm-workspace.yaml
├── playwright.config.ts  postcss.config.mjs  prettier.config.mjs
├── tsconfig.json  vitest.config.ts
│
├── contracts/README.md          governance note only — NO knowhub-openapi.json   S3 R1, R7
│
├── scripts/                     generate-api-client · verify-contract-freshness
│                                inspect-build-artifact · verify-security-headers
│                                write-contract-evidence
│
├── docs/adr/ADR-024-….md        existing                                  S1 R26
├── docs/ux/0B-accessibility-review.md   recorded review evidence          S5 R22
│
├── src/
│   ├── proxy.ts                 CSP nonce + security headers (Next 16)    S5 R39–R46
│   ├── app/
│   │   ├── layout.tsx           the one root layout                       S1 R12
│   │   ├── providers.tsx        the one composition point                 S1 R13; S3 R47
│   │   ├── page.tsx             foundation demonstration, NOT Portfolio   S2 R18; S1 R16
│   │   ├── error.tsx · global-error.tsx · not-found.tsx                   S2 R26
│   │   └── auth/                sign-in · callback · sign-out · session · access-denied
│   ├── components/ui/ · states/ · shell/ · access/                        S2; S4 R47
│   ├── lib/
│   │   ├── config/  schema · server · public · index                      S4 R1–R11
│   │   ├── api/     client · server-client · correlation · errors · types S3 R26–R42
│   │   ├── auth/    session · session-authority · authority-binding ·
│   │   │            providers · oidc · transaction-state · csrf ·
│   │   │            redirects · guard · permissions · diagnostics         S4
│   │   ├── query/   client · keys                                         S3 R46–R52
│   │   └── utils.ts (`cn` only)
│   └── styles/globals.css       Tailwind `@theme` — the single token source  S2 R1
│
└── tests/                       unit · component · contract · accessibility ·
                                 e2e/{shell,auth,security,multi-instance,
                                      accessibility,headers} ·
                                 fixtures/{contract,correlation,shell,secret-scan} ·
                                 harness/{synthetic-authority,attacker-origin,
                                          authority-binding.ts}
```

**Deliberately not created at 0B:** `src/lib/telemetry/` (S1 R9 names it),
`src/hooks/`, `src/view-models/`, `src/lib/format/`, `public/`, `deploy/`, every
product route and component directory, `contracts/knowhub-openapi.json`, and
`src/lib/api/generated/` — no product contract exists, so the directory would be
empty and S1 R9 forbids it. The product output path is recorded in
`contract-generation.config.json` and the directory is created by the change that
pins a real artifact.

## 9. Frontend architecture and layering model

**Server/client policy (S1 R14).** *Server components are the default.* A
component becomes a client component only where interactivity requires it, and
the boundary is visible: every `'use client'` file lives under
`src/components/**` and never imports `src/lib/auth/**` or
`src/lib/config/server*`. The policy is stated in `README.md` and enforced
mechanically.

**Composition (S1 R12–R13).** `src/app/layout.tsx` is the only root layout and
renders exactly one `<Providers>` (`src/app/providers.tsx`), which composes the
query client and nothing else at 0B. A future product route group is a sibling
under `src/app/` and changes neither file — the property 0B-AC-008 checks.

**Layering (S1 R17).** `eslint-plugin-boundaries` element types with exactly this
permitted edge set:

```
app          → components, lib-auth, lib-config, lib-query, lib-api
components   → components, lib-query, lib-api, lib-utils
               (never lib-auth, never lib-config/server)
lib-query    → lib-api
lib-api      → lib-config, generated
lib-auth     → lib-config, lib-api, lib-query
lib-config   → (nothing)
tests        → everything
src/**       → tests/**                              FORBIDDEN
```

Dependencies flow downward only; nothing depends upward; `lib-auth` is
additionally marked server-only.

**Server-only protection (S1 R15) uses two independent mechanisms**, because one
is not enough for a placement rule: the `server-only` package makes a client
bundle that reaches such a module **fail to build**, and a boundaries rule
forbids the import in the first place and fails lint.

**Design-system layering (S2 R8).** Primitives are built before pages; the shell
composes primitives; `page.tsx` composes the shell. No primitive is extracted
retroactively from a page.

**No layer implements backend domain semantics.** No authorization logic,
freshness scoring, impact semantics or evidence resolution exists in TypeScript,
and no view-model becomes a second domain model. *(S1 R18–R20; ADR-022)*

## 10. Configuration boundary

One boundary at `src/lib/config/`, added under S1 R11 exactly as S4 §1.2
contemplates. It is the **only** place an environment value is read, parsed,
validated, defaulted or narrowed. *(S4 R1–R3)*

- `schema.ts` — a Zod schema in which **every value declares
  `exposure: 'server' | 'browser'`** (S4 R5). Browser exposure is an explicit
  allowlist, never a default (S4 R6).
- `server.ts` — marked `server-only`; holds server-only values.
- `public.ts` — browser-exposed values only.
- `index.ts` — typed accessors, including the **validated backend origin** the
  API boundary consumes (S4 R8; S3 R32–R33).

**Naming (this plan's choice; S4 R4 fixes none).** Server-only:
`KNOWHUB_<DOMAIN>__<KEY>`. Browser-exposed: `NEXT_PUBLIC_KNOWHUB_<KEY>` — the
framework prefix is required for inlining and acts as a second signal alongside
the declared allowlist.

**Secrets (S4 R9).** Secret-bearing configuration is expressed only as a secret
or credential **reference** (for example `…_CLIENT_SECRET_REF`), resolved
server-side through the environment's approved mechanism. No secret-manager
product, SDK, secret name or credential is selected, and no secret-manager
infrastructure is introduced for 0B.

**Fail-closed (S4 R10–R11).** Validation runs once at module initialisation.
Missing or invalid security-sensitive configuration throws with a message naming
what is missing — it never silently defaults to a weaker mode. A narrower scope
may only make behaviour stricter.

A lint rule forbids `process.env` anywhere in `src/**` outside this boundary.

## 11. Design-system and shell implementation strategy

**Tokens (S2 R1–R3).** A single `@theme` block in `src/styles/globals.css` is the
sole authoritative definition. Tailwind v4 emits each token as a CSS custom
property *and* as a utility, so a primitive using `bg-surface` and a rule reading
`var(--color-surface)` share one definition — which is precisely what 0B-AC-020
tests by changing a token and asserting every consumer moves.

Resolved values within the §0E.17.1 direction (approximations resolved by this
plan, as R3 permits): canvas `#F7F9FC`; surface `#FFFFFF`; text primary
`#0F172A`; text secondary `#475569`; accent `#2563EB`; restrained teal secondary
`#0D9488`; border `#E5EAF1`; radius 8/10/12px; type scale 12/14/16/20/24/32px
with tabular numerals for metrics; elevation only on drawers and modal surfaces.
Status tokens express **generic presentation semantics only** — neutral, info,
success, warning, error — at accessible contrast. No additional brand or accent
palette. No dark theme, no switcher, no contained dark code surface (S2 R6–R7).

**Typography.** Geist through the package's own `geist/font/sans` export
(`GeistSans`), which wraps `next/font/local` over the 45 `woff2` files the
package bundles — self-hosted, no build-time font fetch, and `font-src 'self'`
needs no exception. `next/font/google` is not used.

**Primitives (S2 R9–R11).** Exactly the R9 set and no more: button; input,
select, textarea; checkbox, radio; table; status badge; entity pill; drawer or
sheet; modal confirmation; banner or alert; skeleton, empty, error, forbidden,
not-found, progress; page/workspace shell. Radix supplies focus, dismissal and
ARIA behaviour; CVA supplies variants. **No product-specific surface** — evidence
citation, graph control, admin permission matrix — is built; the shell exposes
composition slots instead of placeholder components.

**Shell (S2 R12–R20).** Stable left navigation; application-switcher slot;
compact top bar carrying the global command affordance and environment, help and
identity slots; full-width workspace; page header. Every persistent-context slot
renders correctly while empty. The ⌘/Ctrl-K affordance is present, focusable and
**not disabled**, and on activation opens a surface stating the capability is not
yet available while performing no search and executing no command. No product
navigation entry and no product route target. No fabricated application, user,
permission or navigation data in production source — demonstrations use
`tests/fixtures/shell/`.

**State classes and errors (S2 R21–R29).** Every §61A.3 state has a reusable
presentation; skeletons rather than spinners for known layouts; empty states
carry one obvious next action; an important error renders near the failed object
with any supplied support or correlation identifier displayed; toasts carry
transient confirmation only. Error presentation consumes the normalized error
§12 produces and renders no raw exception text, stack trace, credential, secret
or internal value. The async-progress presentation renders externally supplied
stage, progress, last event and error and derives none of them.

**Accessibility (S2 R30–R43).** Semantic landmarks; one token-driven visible
focus ring; focus order following reading order; accessible names on every
control; labels with programmatic error association; a non-colour signal on every
status treatment; 120–180 ms transitions with a `prefers-reduced-motion` rule
that **removes** motion rather than shortening it; modal surfaces receive,
contain and restore focus while non-modal drawers do not trap; a tablet collapse
strategy turning side panes into sheets with every control still reachable.

## 12. OpenAPI and client-generation strategy

**What exists at 0B.** `contracts/` holds only `README.md`, the governance note
recording S3 R2–R4 and the `/api/v1` rule.
**`contracts/knowhub-openapi.json` does not exist and none is manufactured**
(S3 R7; §3).

**Pipeline (S3 R13–R18).** `contract-generation.config.json` names input and
output explicitly; the input has **no default**, so generation cannot discover a
document implicitly, and pointing the pipeline at a different document is a
configuration change rather than a code change (S3 R14).

- Product output path, recorded and unused at 0B: `src/lib/api/generated/`.
- Fixture output, exercised at 0B: `tests/fixtures/contract/generated/`,
  **committed** so drift is detectable.

Generation is deterministic — same input, same committed configuration,
lockfile-pinned generator ⇒ byte-identical output; fails loudly on malformed
input leaving nothing partial behind; and requires no backend checkout, no
running backend, no network and no workstation state.

**The synthetic fixture (S3 R9–R12).**
`tests/fixtures/contract/synthetic-pipeline-fixture.openapi.json` — outside
`contracts/`, outside `src/`, unreachable from production code, describing an
obviously synthetic surface with no KnowHub vocabulary, no M1 endpoint name, no
`/api/v1` or product-like path, no authentication schema and no business schema.
It is never promoted, copied or renamed into `contracts/`, and nothing generated
from it ships as a product client.

**Both R7 paths stay valid.** Only the config file's `input` changes when a
legitimate backend-generated artifact arrives through §30A.2. Pipeline, freshness
gate, boundary and evidence script are unchanged; the scripts treat the presence
of `contracts/knowhub-openapi.json` as a branch, not a rewrite.

**The single API boundary (S3 R26–R33).** `src/lib/api/` is handwritten and
generic over the generated `paths` type, so no product operation is defined and a
real contract binds it without edits. Distinct server-side and browser-safe
paths; the browser-safe path carries no bearer credential; exactly one place
converts a transport outcome into an application-level result; no component
inspects an HTTP status or parses a backend error body. The boundary reads no
environment variable and obtains its origin from §10.

**Correlation (S3 R34–R36).** A `CorrelationBinding { headerName, generate,
isConforming }` is a **required** boundary parameter with **no default**, so the
product binding stays unbound. 0B supplies only a clearly synthetic test binding
under `tests/fixtures/correlation/`. A conforming inbound identifier is
preserved, an absent one originated, a non-conforming one replaced rather than
propagated. No browser telemetry system is introduced.

**Normalized error (S3 R38–R42).** One `ApiError` produced only in
`src/lib/api/errors.ts`, carrying the §70.5 field set — code, message,
correlation identifier, retryable flag, optional remediation hint — plus the HTTP
status class where meaningful. `retryable` stays unset unless the contract states
it; it is never inferred. Nothing unsafe survives normalization. Presentation
belongs to §11 and is defined nowhere here.

**Evidence honesty (S5 R27).** `scripts/write-contract-evidence.mjs` publishes a
record naming the input, its hash and what it demonstrates, stating explicitly
that **no product backend contract is bound**, fabricating no backend revision,
and claiming **no compatibility with `knowhub-backend`**. The plan makes it
impossible to confuse pipeline verification with compatibility verification: the
two are separately named in the evidence artifact, in `contracts/README.md` and
in the verification report.

**Streaming (S3 R43–R45).** No streaming implementation, protocol, event schema,
run event model, resumption token format or product event name is built. R43 and
R44 bind the streaming path when it is built; 0B provides no runtime evidence for
a stream that does not exist.

## 13. BFF, authentication and session architecture

Governed by ADR-027 and S4.

**Canonical session (S4 R15; ADR-025).** One provider-neutral shape with six
states — unauthenticated, authenticated, expired, revoked, forbidden,
authentication-error. There is no `LocalSession`/`OidcSession` split. Provider
identity is a **displayable attribute only**: no guard, gate, cache key or
entitlement path reads it. Provider-conditional behaviour exists solely inside
the authentication boundary for protocol mechanics — initiation, callback,
provider-appropriate logout, discovery — and changes no entitlement, permission
or identity semantics (S4 R16).

**Provider selection (S4 R12–R14).** The configuration layer produces one
`AuthProviderRegistry` expressing an **enabled provider set** — local enabled or
not, plus zero or more OIDC providers keyed by an opaque `providerId` — so LOCAL,
OIDC and HYBRID, including several simultaneous OIDC providers, are all
representable. No provider identity is hard-coded in any route, component or
module.

**The session authority port stays unbound (S4 R34, R45; 0B-AC-073).**
`session-authority.ts` declares an interface with **no transport, no path and no
request or response shape**, because declaring a backend authentication endpoint
shape is prohibited at 0B. `authority-binding.ts` is the single seam; its
production content resolves to an unbound implementation whose every call yields
`authentication-error`, so the production runtime **fails closed** with no
authority present. M1 replaces exactly this module with a binding generated from
the pinned contract.

**Two build profiles, one seam.**

| | Production profile | Verification profile |
|---|---|---|
| Source | identical | identical |
| `authority-binding` resolves to | `src/lib/auth/authority-binding.ts` (unbound) | `tests/harness/authority-binding.ts` |
| Mechanism | — | **compile-time module alias**; no environment flag, no dynamic import, no hidden route, no runtime profile selector |
| Used for | bundle and source-map inspection, CSP/header gate, Dockerfile and image, release evidence | Playwright: auth, security-negative, session, multi-instance |
| Published | yes | **never** |

Exactly one application module differs. **No application behaviour and no product
capability forks.** Both multi-instance instances are built from the verification
profile, so cross-instance OIDC still exercises the approved category-B
mechanism. The production artifact-inspection gate proves the synthetic authority
module, its identifiers and its source-map content are absent (S5 R53).

**Token custody (S4 R19–R24).** No KnowHub access, refresh or session token and
no identity-provider token exists in `localStorage`, `sessionStorage`,
IndexedDB, a URL path, query string or fragment, a JavaScript-readable cookie,
page props or hydrated state, nor in any browser-visible log or diagnostic.
Browser code never constructs an `Authorization` header for the backend. An
internal access token, when M1 introduces one, is request-scoped and server-side
only. **No durable frontend session datastore exists under any name** — no
database, no Redis, no session store, no "cache", "session cache", "token cache"
or "warm store" (S4 R23; ADR-027 §2).

**Cookies.**

| Cookie | Attributes | Contents |
|---|---|---|
| `__Host-kh.session` | `HttpOnly; Secure; SameSite=Lax; Path=/` | a 256-bit random opaque reference and nothing else (S4 R17–R18) |
| `__Host-kh.oidc-txn` | `HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=300` | the sealed category-B payload (§14) |
| `__Host-kh.csrf` | `Secure; SameSite=Lax; Path=/`, deliberately **not** `HttpOnly` | a random CSRF token — not session, token or identity material, so S4 R19 is untouched |

`SameSite=Lax` rather than `Strict`: the OIDC callback is a cross-site top-level
GET navigation that `Strict` would strip. S4 R63 requires the cookie policy and
the integrity control to be chosen **together** so neither alone is relied upon —
hence `Lax` **plus** the double-submit token **plus** `Origin`/`Referer`
validation (§15).

**`Secure`, `HttpOnly`, the SameSite policy and the `__Host-` prefix are never
relaxed, in any profile.** That is fixed, and no fallback changes it — S4 R36 and
R38 and §76's "No insecure bypass committed enabled" leave no room for one.

**How the browser actually behaves is verified, not assumed.** The browser layer
is intended to run against `http://localhost`, but whether the **pinned Chromium
build** accepts `Secure` and `__Host-` cookies over HTTP on localhost is
established by executing the check against that build (T22) rather than asserted
here. **If those cookies are rejected, the browser harness uses local TLS.**
Removing `Secure`, dropping the `__Host-` prefix or creating an insecure profile
is never an acceptable workaround.

**Session lifecycle (S4 R39–R45).** The reference rotates on authentication and
re-authentication; a pre-authentication reference cannot be reused; the cookie is
cleared on sign-out. A session transition invalidates cached server state through
the query layer, so no cached result is served across an authorization scope
boundary. Security-sensitive uncertainty fails closed, and no frontend component,
guard, port or cached value holds session authority.

**LOCAL (S4 R35–R38).** LOCAL traverses the same BFF and session path shape as
OIDC and terminates in the same canonical abstraction. It is not a client-side
bypass and creates no second authorization path. The frontend holds no product
credential store and trusts no browser-supplied user, role or permission data.
No hard-coded privileged credential exists in any repository artifact.

**OIDC protocol mechanics, and the APIs 0B may use (S4 R34).** The frontend BFF
performs the protocol mechanics it owns and **no federated validation whatever**.
Concretely, with `openid-client` 6.8.8 the permitted surface is:

| Permitted at 0B | Purpose |
|---|---|
| `randomPKCECodeVerifier()`, `calculatePKCECodeChallenge()` | PKCE |
| `randomState()`, `randomNonce()` | per-request `state` and nonce generation |
| `buildAuthorizationUrl()` | authorization-request construction |
| `buildEndSessionUrl()` | provider-appropriate logout (§0E.13.9) |
| `discovery()` | authorization and end-session endpoint discovery — permitted protocol branching under S4 R16 |

**Prohibited at 0B**, because each performs validation S4 R34 reserves to the
federated boundary: `authorizationCodeGrant()` — it exchanges the code **and**
validates the ID token's issuer, audience, signature/JWKS, `exp`/`nbf` and, via
`expectedNonce`, the **nonce claim** — together with `implicitAuthentication()`,
`refreshTokenGrant()`, `fetchUserInfo()`, `tokenIntrospection()` and
`tokenRevocation()`.

**The 0B BFF therefore performs no token exchange**, and **declares no
federated-exchange contract** (S4 R34). What 0B owns at the callback is exactly:
the authorization response stays server-side; `state` is validated against the
sealed category-B transaction; callback and redirect targets are validated; and
the production federated-authority binding remains **unbound and fail-closed**.

**The concrete handoff is an M1 contract question 0B does not answer.** How the
authorization code and PKCE verifier reach the federated boundary — the request
and response shape of that exchange — is deferred to M1 exactly as the nonce's
route to that boundary is, and for the same reason: §0E.13.3 places the exchange
at the FastAPI identity boundary, S4 R34 forbids the frontend declaring its
contract, and ADR-027 §8 defers OIDC exchange details to M1. Writing that shape
now — in production source, or indirectly as a payload type on the authority
port — would be declaring the contract in all but name.

**What 0B proves, and what M1 adds.** These are different things and the plan
states the line once, here:

> **0B proves:** the OIDC protocol shell, category-B transaction integrity, and
> horizontal callback behaviour.
>
> **M1 adds:** federated token exchange, provider validation, and canonical
> authenticated-session establishment through the backend contract.

The synthetic OIDC provider is therefore an **authorization-redirect test
authority**, not evidence that a production federated exchange exists. Because no
production interface carries an authorization response to a federated authority,
**no verification-profile implementation may invent one** — doing so would
perform an exchange the production interface does not expose while claiming to
exercise that unchanged interface, which is not evidence of anything.

The browser layer may therefore prove: authorization-request construction;
Authorization Code + PKCE mechanics; per-request `state` and nonce generation;
`state` validation at the BFF; category-B integrity, confidentiality and expiry;
server-side authorization-response handling; callback and redirect-target
validation; transaction-cookie clearing; replay rejection; cross-instance
transaction completion using the shared sealing configuration; that no provider
token, authorization code or PKCE verifier reaches browser application code; and
that production **fails closed because the federated exchange boundary is
intentionally unbound**.

It must **not** claim: successful federated token exchange; successful ID-token
validation; successful OIDC-authenticated session creation; end-to-end federated
login; an authorization-code / PKCE-verifier exchange contract; or any M1 backend
authentication API behaviour. Session-boundary properties — the opaque reference,
the six canonical states, rotation, revocation, cookie clearing and the storage
and redaction sweeps — are exercised through the `SessionAuthorityPort`, an
interface that does exist, and are never described as established by a federated
token exchange.

**Nonce, stated precisely.** A `nonce` is **generated per authorization request
and bound to the category-B transaction state** (S4 R29). The BFF **generates and
validates `state`** (S4 R28). The BFF performs **no ID-token nonce-claim
validation** — that, like issuer, audience, signature, expiry and tenant policy,
occurs at the federated boundary, which is backend and M1 (S4 R34). How the nonce
reaches that boundary is an M1 contract question 0B does not answer and does not
claim.

**The synthetic authority (S4 R71–R75) is test-only and out of process.** A small
Node service under `tests/harness/synthetic-authority/` plays the role M1's
backend will play **for the session boundary**: it issues, resolves, rotates,
revokes and reports all six states through the `SessionAuthorityPort`. It also
hosts a minimal local OIDC provider, which is an **authorization-redirect test
authority** used to drive the transaction round trip — it performs no federated
exchange and evidences none. Being out of process is what lets two frontend
instances resolve the same session **without any frontend-owned shared state**. It exercises the canonical production interfaces
rather than bypassing them, possesses no authorization authority, defines no
product authentication API contract, credential store or user/role/permission
model, and ships in nothing.

## 14. Category-B OIDC transaction-state mechanism

**Mechanism.** A JWE (`dir` + `A256GCM`, via `jose`) sealed with a server-only
symmetric key supplied as a secret **reference** through the configuration
boundary (S4 R9), carried as `__Host-kh.oidc-txn`, `Max-Age=300`. Payload:
`state`, `nonce`, PKCE `codeVerifier`, `providerId`, validated `returnTo`, `exp`.
Deleted on completion, on failure and on expiry.

**Verified against every S4 R30 property:**

| R30 property | How it is satisfied |
|---|---|
| Exists only for one authentication transaction | 300 s `Max-Age` plus an `exp` claim; deleted by the callback on every terminal outcome |
| Must not become durable session authority | carries no subject, no permission, no session reference; exchanged and discarded |
| No sticky routing; works across instances | any instance holding the same **configured key** can open it — configuration, not frontend-owned state |
| Integrity-protected | AEAD; any tamper fails decryption and fails closed |
| Confidential components inaccessible to browser JS | `HttpOnly` **and** encrypted |
| No durable user, role or permission authority | no such field exists |
| Cleared on completion, expiry and failure | explicit deletion on all three paths |
| Never a product session or authorization store | distinct name, distinct key, distinct lifetime; the guard never reads it |

**Why this is not ADR-027 Alternative A.** ADR-027 rejected *sealed
browser-carried **session** material* because a sealed session survives
revocation. This seals only category-B transaction correlation, which ADR-027 §8
leaves undecided, §0E.13.7 calls a "transient/token cache during OIDC flow",
S4 R30 expressly permits, and S5 §14 expressly states is **not** implicated
merely because it works across frontend instances. It introduces no new
persistent or shared runtime infrastructure: the key is configuration every
horizontally scaled deployment already carries.

**Stop condition carried into every affected task.** If implementation requires
shared frontend *runtime state*, sticky routing, a durable frontend session
store, browser token custody, or frontend entitlement authority, implementation
**stops** and a superseding-ADR decision is surfaced (§84.4; S4 §14; S5 §14).

**Library choice recorded.** `iron-session` was rejected precisely because
adopting a library framed as a *session* store would blur the category A/B line
this plan depends on. `jose` seals a transaction, names nothing a session, and is
the same standards library the OIDC leg needs.

## 15. CSRF and redirect security

Two request-binding regimes exist, because two different kinds of request cross
the BFF and only one of them is a browser-originated application action.

**Regime 1 — ordinary state-changing browser → BFF requests** (sign-in
*initiation*, sign-out, and every future application action). A double-submit
token in `__Host-kh.csrf`, echoed by the client in `X-KnowHub-CSRF`, **plus
mandatory `Origin` (falling back to `Referer`) allowlist validation**, applied by
one shared server-side helper. Neither control is relied on alone (S4 R63).
**LOCAL and OIDC initiation pass through the identical helper**, so no provider,
mode or profile weakens request integrity (S4 R64).

**Regime 2 — the OIDC authorization callback.** This request is a top-level
redirect issued by the authorization server, not a browser-originated
application action: the authorization server cannot set a custom header and does
not present our origin. Requiring regime 1's mechanism here would demand an HTTP
control the redirect cannot supply. Its request binding is instead established
by the mechanism §0E.13.6 prescribes and S4 R26–R31 require — **Authorization
Code + PKCE, a per-request `state` validated at the BFF against the sealed
category-B transaction state, and strict callback-target validation** — with a
mismatched, missing or unrecognised `state` failing closed.

**S4 R64 is satisfied, not narrowed.** It requires that LOCAL and OIDC receive
the same protection and that no provider weakens request integrity. Both
providers' *initiation* legs pass through the identical regime-1 helper; the
OIDC callback additionally carries regime-2 binding that the LOCAL leg has no
equivalent of. No provider is protected less; the two legs are protected by the
control appropriate to the request each actually is. R64 is not read as requiring
a mechanism an authorization-server redirect cannot carry.

This plan extends into no CORS policy (S4 §11; S5 R47).

**Redirects (S4 R31, R65).** One `assertApprovedRedirect()` accepts only a
same-origin absolute path — rejecting `//host`, `\host`, any scheme and
percent-encoded variants of each — or an exact match in a configured
approved-origin list for post-logout targets. The OIDC callback target is
validated against the configured registered redirect URI before the browser is
redirected and again on return. It is used by sign-in return, sign-out and
callback; **no open redirect exists**.

## 16. Permissions port and fail-closed behaviour

**One port (S4 R46–R53).** `src/lib/auth/permissions.ts` exposes a single
`PermissionsPort` returning an opaque effective-authorization result —
`{ known: false }` or `{ known: true, allows(capabilityId) }`. At 0B the bound
implementation is the unbound one, returning `{ known: false }`, so **every**
capability query fails closed.

Consequences the tasks must hold:

- **0B introduces no permission name, role name, DTO or permission taxonomy**,
  and no TypeScript RBAC, ABAC or policy evaluator derives entitlement from
  roles, provider claims or client state.
- Frontend authorization controls **UX availability only** and never constitutes
  enforcement. Hidden or absent UI is not an authorization control and no trust
  decision derives from one.
- A navigation item the shell is told is unauthorized is **absent**, never
  rendered and disabled (S2 R17).
- Unknown, absent or indeterminate authorization fails closed for protected
  capability (S4 R50).
- Editing client state, a route URL, browser storage or JavaScript state grants
  no capability, and browser- or provider-supplied role and permission claims are
  never treated as authority (S4 R51–R52; ADR-023).
- Once M1 supplies the authoritative contract, generated permission and
  effective-authorization identifiers flow through the port unchanged.

**One guard (S4 R54–R59).** `resolveSessionState()` and `requireSession()` are
server-only and are called at the server boundary of any protected segment.
The Next 16 Proxy is **not** the guard — it carries the header baseline only.
Protected content is never rendered and then hidden after hydration. Redirect and
deny behaviour is deterministic. A loading, pending or unknown state grants no
access. **No product protected route exists**, so the guard is exercised through
the auth-shell routes and the shell's session-dependent regions — never through a
test-only route, which S5 R10(c) forbids.

## 17. Query and cache foundation

**One client (S3 R46–R47).** TanStack Query is the server-state foundation, with
local React state for transient interaction state and no second server-state
store. A single `makeQueryClient()` in `src/lib/query/client.ts` defines all
application-level defaults, composed at the one provider-composition point. No
feature or query module defines its own query-client default policy. The client
is created **per request on the server and once per browser**, so no query cache
is ever shared between users.

**One key convention (S3 R48–R49).** `createQueryKey({ scope, resource, params })`
in `src/lib/query/keys.ts` is the only way a key is built; keys are never
assembled ad hoc at call sites. `QueryScope` **can** carry authoritative scope
dimensions so a cached result is never served across a scope boundary, and
**declares none at 0B**, because the capabilities defining scope do not exist.

**Invalidation (S3 R51; S4 R44, R53).** Invalidation flows through the query
layer, never through ad-hoc cross-component signalling. A session transition
clears the client outright, which is the deterministic way to guarantee no cached
result survives an authorization scope change.

**No product query and no product cache key exists** (S3 R52). Executable
demonstrations use test-only synthetic state.

## 18. Test architecture

| Layer | Runner / engine | Location |
|---|---|---|
| Unit | Vitest, node project | `tests/unit/` |
| Component | Vitest, jsdom + React Testing Library | `tests/component/` |
| Contract | Vitest, node project | `tests/contract/` |
| Accessibility — structural | Vitest, jsdom + axe-core | `tests/accessibility/` |
| Accessibility — authoritative | Playwright + `@axe-core/playwright` | `tests/e2e/accessibility/` |
| Browser / E2E | Playwright, Chromium | `tests/e2e/{shell,auth}/` |
| Security-negative | Playwright, `security` project | `tests/e2e/security/` |
| Multi-instance | Playwright | `tests/e2e/multi-instance/` |
| Header / artifact | Node scripts | `scripts/` |

Only layers holding real 0B tests are materialized; no empty test-layer directory
mirrors §75.1 (S5 R1–R2).

**Determinism (S5 R3).** A repeated run over unchanged inputs produces the same
result. No outcome depends on execution order or ambient wall-clock time; time is
controlled with fake timers where behaviour is time-sensitive. Playwright runs
with `retries: 0`, because retries mask the non-determinism R3 forbids.
Determinism is **not** claimed for mutable external security intelligence — a
vulnerability database may legitimately change a scan result between runs, and
those gates stay blocking (0B-AC-091).

**Independence (S5 R4–R5).** Verification needs no `knowhub-backend` checkout, no
running backend, no real identity provider, no sibling repository, no remote
product service and no workstation state. This is *service and repository*
independence, not offline execution: loopback traffic to locally started
instances, locally hosted synthetic authorities, fake clocks, dependency
acquisition and scanner data access are all unaffected. **The single offline
obligation is contract generation** (S3 R17, carried into S5 R24).

**Test-only containment (S5 R9–R12) uses four independent mechanisms**, so that
explicit test enablement is never the sole protection:

1. `tests/` lives outside `src/` — no production entry point reaches it.
2. `tsconfig` build `include` excludes it.
3. A boundaries rule forbids `src/**` importing `tests/**`, failing lint.
4. Artifact inspection fails on any test sentinel in the production bundle or a
   source map.

**Failure-path demonstration (S5 R6, R17).** Every blocking gate gets a recorded
"deliberate violation fails it; removing it restores green" exercise: a layering
violation, a client import of `lib/auth`, a blanket `any`, a formatting
deviation, a hand-edited generated file, a stale generated file, a malformed
contract, a `dangerouslySetInnerHTML`, an ephemeral synthetic credential, a
weakened CSP directive, and a test sentinel in production source.

**Two harness design decisions that carry weight:**

- **The attacker origin is a port, not a host.** `127.0.0.1` and `localhost` are
  different *hosts*, so a page there would carry no cookies and the CSRF test
  would pass for the wrong reason. `http://localhost:3900` is the same host —
  cookies **are** sent — while `Origin` differs, because origins include the
  port, and a cross-origin page cannot set `X-KnowHub-CSRF`. That isolates the
  Origin-validation and token controls precisely.
- **Multi-instance needs no proxy.** Cookies are not port-scoped, so
  `localhost:3100` and `localhost:3101` share one cookie jar: sign in on
  instance A and complete the OIDC callback on instance B, with no sticky routing
  and no shared frontend state.

## 19. Accessibility verification strategy

**One engine, two placements.** axe-core is the single accessibility engine
(S5 R21 fixes none, so this plan selects it). It runs in two places for a
technical reason, not for redundancy:

- **Structural rules in the component layer** (jsdom): accessible names, labels,
  programmatic error association, landmark structure, ARIA correctness.
- **The authoritative pass in Chromium** via `@axe-core/playwright`, with tags
  `wcag2a, wcag2aa, wcag21a, wcag21aa, wcag22aa` enabled.

**Why the split is necessary.** `color-contrast` and WCAG 2.2 AA `target-size`
(2.5.8) — both required by 0B-AC-031 — cannot be evaluated in jsdom, which has no
layout engine. Running them only in the component layer would produce a green
result that proves nothing. Real layout is what makes those assertions honest.

**Where automation cannot decide (S5 R22).** Evidence is a targeted assertion or
a **recorded review**, identified as such and never presented as an automated
result. `docs/ux/0B-accessibility-review.md` carries dated, attributed rows —
criterion, method (`inspection` or `targeted assertion`), surface, outcome,
reviewer. **No numeric or automated metric is fabricated for a criterion
automation cannot decide, and no aesthetic or design-quality judgement is
expressed as a test threshold.** The R44 prohibition checklist (0B-AC-036) is
recorded-review evidence by construction.

**Scope (S5 R23).** Evidence covers only the surfaces 0B delivers — shell,
primitives, state classes. The primary Ask, evidence and source workflows AT-017
names do not exist, and **no evidence produced here is presented as discharging
AT-017**.

## 20. Production CSP and security-header strategy

Delegated to S5 by S4 §1.1, §14 and §16, together with ownership of AT-035.

### 20.1 Content Security Policy

Emitted per request from `src/proxy.ts` with a fresh nonce:

```
default-src 'self'; base-uri 'self'; object-src 'none';
frame-ancestors 'none'; frame-src 'none'; form-action 'self';
script-src 'self' 'nonce-{N}' 'strict-dynamic';
style-src 'self' 'nonce-{N}'; style-src-attr 'unsafe-inline';
img-src 'self' data: blob:; font-src 'self'; connect-src 'self';
worker-src 'self' blob:; manifest-src 'self'; upgrade-insecure-requests
```

| Directive | Why this value |
|---|---|
| `object-src 'none'` | S5 R40, verbatim |
| no `'unsafe-eval'` | S5 R40. The strict policy binds the **production** runtime; `next dev` needs eval and R39 scopes the baseline to production |
| `script-src 'self' 'nonce' 'strict-dynamic'` | S5 R40's "no arbitrary third-party script sources". Next propagates a nonce found in the CSP header to its own scripts, so no `'unsafe-inline'` is needed. See the dynamic-rendering consequence below |
| `connect-src 'self'` | S5 R41 exactly: the browser talks only to the BFF. **No backend origin is added, automatically or otherwise**, and no server-only backend origin is exposed to populate a directive |
| `style-src-attr 'unsafe-inline'` | Necessary and permitted. Radix positions floating surfaces with inline `style` **attributes**, which `style-src 'self'` alone blocks. S5 R40 restricts eval and third-party script, not style attributes; scoping to `style-src-attr` keeps inline `<style>` elements blocked |
| `font-src 'self'` | Geist is self-hosted; no third-party font origin |
| `frame-ancestors 'none'` | the S5 R46 anti-clickjacking outcome |
| `upgrade-insecure-requests` | no mixed content in production |

Deliberately **absent** per S5 R47: COOP, COEP, CORP, Subresource Integrity, any
`report-uri`/`report-to`, and any CORS policy.

### 20.2 Header baseline

| Header | Value | Reason |
|---|---|---|
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains` | S5 R42. Two years, subdomains included. **No `preload`** — submission is a deployment decision and R49 keeps the 0B claim at the artifact boundary |
| `X-Content-Type-Options` | `nosniff` | S5 R43 |
| `Referrer-Policy` | `no-referrer` | S5 R44 asks for a strict policy. The application is same-origin and routes will later carry application identifiers, so leaking any part of a URL has no upside. Documented alternative if a provider ever requires it: `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `accelerometer=(), autoplay=(), camera=(), display-capture=(), encrypted-media=(), fullscreen=(self), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), midi=(), payment=(), publickey-credentials-get=(self), screen-wake-lock=(), usb=(), xr-spatial-tracking=()` | S5 R45. Everything denied except `fullscreen` (a future diagram surface) and `publickey-credentials-get` (§0E.13.5's WebAuthn path at M1), both restricted to `self` |
| `X-Frame-Options` | `DENY` | S5 R46, defence-in-depth beside `frame-ancestors` |

**Placement.** The Next 16 **Proxy** (`src/proxy.ts`, the renamed `middleware`
convention) carries the whole baseline for document and route responses under
matcher `/((?!_next/static|_next/image|favicon.ico).*)`; `next.config.ts
headers()` carries only HSTS and `nosniff` for `/_next/static/:path*`. No path
receives a header from both places, so no duplicate or conflicting value can
occur. The Proxy remains a request and header boundary only — **it is never the
authentication or authorization guard**, which stays at the server boundary
(§16).

**Dynamic-rendering consequence of the nonce strategy.** A per-request nonce can
only reach framework and page scripts if the document response is **dynamically
rendered** — a statically rendered document is produced at build time, before any
nonce exists, and would silently fall outside the nonce model while still
appearing to pass a header-only check. This is an implementation consequence of
the CSP strategy selected above, not a new product requirement. The production
runtime verification in §20.2 therefore additionally asserts that a document
protected by the nonce CSP is dynamically rendered, that framework and page
scripts carry the expected nonce, that no static-rendered document bypasses the
model, and that the policy still contains no `'unsafe-eval'`.

**Verification (S5 R48).** `scripts/verify-security-headers.mjs` starts the
**production** build, fetches a document response and a static asset, and asserts
every header, every CSP directive and the negatives — `'unsafe-eval'` absent, no
third-party script source, `connect-src` exactly `'self'`. It additionally
asserts the dynamic-rendering properties above: the nonce differs between two
requests for the same document, every framework and page `<script>` carries that
request's nonce, and no document served under the nonce CSP was statically
rendered. A weakened or removed control fails the gate, demonstrated by
deliberately weakening one.

**Scope of claim (S5 R49).** Evidence is at the production frontend artifact and
runtime boundary. It does **not** demonstrate deployment-platform configuration;
where a control is later applied or strengthened at a gateway, ingress, CDN or
TLS terminator, environment-level confirmation remains deployment work.

## 21. Artifact, source-map and test-double containment checks

**Unsafe-HTML guard (S5 R50–R51).** Local ESLint `no-restricted-syntax` and
`no-restricted-properties` rules ban `dangerouslySetInnerHTML`, `innerHTML`,
`outerHTML`, `insertAdjacentHTML`, `document.write`, `eval` and `new Function`
in `src/**` — no extra plugin. 0B renders no untrusted rich content, so **no
sanitization library is selected**, and §0E.13.10's allowlist-sanitization
obligation is recorded as binding the capability that first renders such content.

**Artifact inspection (S5 R52–R54).** `scripts/inspect-build-artifact.mjs` scans
`.next/static/**`, `.next/standalone/**` and every emitted `*.map` for:

1. the **values** of server-only configuration present in the build environment;
2. forbidden patterns — `client_secret`, private-key headers, session-key
   material;
3. test-only sentinels — harness module names, fixture operation identifiers, and
   a `KNOWHUB_TEST_ONLY_SENTINEL` constant embedded in every harness module.

Any hit fails the gate. **No source-map policy is required or forbidden**
(S5 R54); the Next default (`productionBrowserSourceMaps: false`) is this plan's
reversible choice, and whatever the build emits is inspected regardless.

## 22. Container and release-artifact strategy

Every item below is an **implementation choice**, justified on its own merits.
None is claimed as a specification requirement, and **no `knowhub-backend` image
decision is imported** (S5 R70).

```
stage deps     node:24-bookworm-slim@sha256:<digest>
               corepack enable; pnpm install --frozen-lockfile
stage build    same base; pnpm build      (next.config: output: 'standalone')
stage runtime  same base
               copy .next/standalone and .next/static
               USER 10001:10001 ; EXPOSE 3000 ; CMD ["node","server.js"]
```

| Choice | Justification |
|---|---|
| `node:24-bookworm-slim`, digest-pinned | matches the declared Node baseline exactly; digest pinning makes the image reproducible |
| three stages | build tooling and dev dependencies never reach the runtime layer, which is also what keeps S5 R65 easy to hold |
| `output: 'standalone'` | Next's own minimal server; avoids shipping `node_modules` wholesale |
| non-root `10001` | least privilege, at no cost |
| `EXPOSE 3000` | Next's default; no port policy is asserted |
| no `HEALTHCHECK`, no labels | no source requires either, and S5 R70 forbids importing the backend's choices. Add them when a deployment target asks |
| `.dockerignore` | keeps `tests/`, `.next`, `.git` and `node_modules` out of the build context — a containment aid as well as a speed one |

**Identity (S5 R67).** `knowhub-frontend:<git-sha>`, built and scanned in CI and
**never pushed**. No registry, no publishing step, no further tag scheme.

**Excluded (S5 R69).** No registry publishing, deployment definition, Kubernetes
or ingress configuration, cloud runtime design, Terraform or Bicep, environment
promotion, production identity-provider registration, TLS, CDN or WAF design, and
no `deploy/` content. Building and validating a release artifact is not designing
a deployment platform.

## 23. Supply-chain and security gates

| Gate | Selection | Requirement |
|---|---|---|
| Frozen install | `pnpm install --frozen-lockfile` | S5 R55 |
| Lockfile currency | `pnpm install --lockfile-only` then `git diff --exit-code pnpm-lock.yaml` — an independent assertion, because a frozen install alone can mask a stale lock | S5 R56 |
| Dependency vulnerability | `pnpm audit --audit-level=high` — framework-native, no extra tool | S5 R57 |
| Secret scan | gitleaks, container pinned by digest, over the **working tree and full git history**, demonstrated against an **ephemeral synthetic credential** created by the check and discarded afterwards | S5 R58 |
| Container vulnerability | Trivy, pinned by digest, `--severity HIGH,CRITICAL --ignore-unfixed --exit-code 1` | S5 R59, R68 |

**A zero-finding scan has passed** (S5 R60); zero findings is never treated as
evidence that a gate is missing. **No SBOM, provenance, signing or licence
scanning is selected or introduced** (S5 R61). No scanning product forms part of
the architectural contract — each is a reversible plan choice.

## 24. CI topology

**Provider: GitHub Actions**, selected now as S5 R76 requires. §0E.1 permits
Azure DevOps Pipelines or GitHub Actions by enterprise standard; both
repositories are hosted on GitHub. Actions are pinned by commit SHA and container
images by digest. **Jobs are named by outcome**, so moving provider would be a
translation rather than a redesign.

| Job | Gates | Depends on |
|---|---|---|
| `quality` | frozen install; lockfile currency; ESLint; Prettier; `tsc --noEmit`; module boundaries; unsafe-DOM rules | — |
| `test` | Vitest node and jsdom projects — unit, component, contract, structural accessibility | `quality` |
| `contract` | generation determinism; malformed-input failure; **no-network generation**; freshness and drift; contract evidence | `quality` |
| `browser` | verification build; Playwright `chromium-desktop`, `chromium-tablet`, `security`; authoritative accessibility pass; multi-instance | `quality` |
| `security` | `pnpm audit`; gitleaks tree and history with the ephemeral-credential demonstration | `quality` |
| `build` | production build; artifact and source-map inspection; CSP and header verification against the production runtime | `quality` |
| `image` | `docker build -t knowhub-frontend:<sha> .`; Trivy scan | `build` |

**Ordering is constrained only where architecturally meaningful** (S5 R77):
inspection and header verification run against the artifact `build` produced, and
the container scan runs after the image exists. All other ordering and
parallelism is free.

**No-network contract generation (S5 R24; 0B-AC-048, 0B-AC-099).** The generation
step runs with the network namespace removed —
`sudo unshare -n sudo -u "$USER" pnpm contract:verify` — with
`docker run --network none` on the same digest-pinned Node image as the recorded
fallback. Dependencies install **before** the namespace is removed, so this
proves the property S3 R17 states without imposing a general offline requirement
(S5 R5).

**Repository independence (S5 R73).** The complete path runs with only
`knowhub-frontend` checked out. No job, step or script fetches from or depends on
a `knowhub-backend` checkout, source tree, Python package or runtime.
**Exception preserved:** a legitimate pinned backend-generated OpenAPI artifact is
permitted as contract **input** and may retain its recorded provenance — that is
an artifact input, not a source or runtime dependency.

**Published evidence (S5 R74–R75).** `playwright-report/`, Vitest and Playwright
JUnit XML, `a11y-report.json`, `contract-evidence.json`,
`security-headers-report.json`, `bundle-inspection-report.json`, the Trivy report
and the container image. All are labelled **CI evidence artifacts** and none is
offered as satisfying §81.1's telemetry condition, which rests on backend runtime
telemetry (M00-SPEC-003 R8–R10) plus the correlation propagation S3 R34–R36 builds
into it. No browser telemetry SDK, vendor integration or metrics framework is
introduced.

## 25. Dependency-aware implementation phases

Derived from actual dependencies, **not** from specification numbering. A phase
is entered only when every prerequisite phase's completion evidence exists.

| Phase | Name | Tasks | Objective | Entry condition |
|---|---|---|---|---|
| 1 | Toolchain and repository skeleton | T01–T03 | a repository that installs, lints, formats, type-checks and builds an empty App Router application | — |
| 2 | Architecture boundaries and configuration | T04–T05 | the layering gate and the one configuration boundary exist and block | phase 1 green |
| 3 | Design tokens, primitives, shell, UI states | T06–T10 | the bright design system and the data-agnostic shell | phase 2 green |
| 4 | Contract pipeline and API/query boundary | T11–T14 | `generate:api` proves itself against the fixture; the one API boundary and query foundation exist | phase 2 green |
| 5 | BFF, session, OIDC, CSRF, guard, permissions | T15–T21 | the full authentication boundary against the synthetic authority | phases 2 and 4 green |
| 6 | Browser, accessibility, security-negative, multi-instance | T22–T26 | every runtime property executes in Chromium | phases 3 and 5 green |
| 7 | Production hardening and build-artifact verification | T27–T28 | the production runtime emits the baseline and the bundle is clean | phases 3 and 5 green |
| 8 | Container and supply chain | T29–T30 | the release artifact builds and scans | phase 7 green |
| 9 | CI wiring | T31 | every gate blocks in one workflow with only this repository checked out | phases 1–8 green |
| 10 | Acceptance verification | T32 | every 0B-AC evidenced and reconciled | phase 9 green |

**Safe parallelism.**

- **Phases 3 and 4 may run concurrently** once phase 2 is green: the design
  system and the contract pipeline share no file and no dependency.
- **Phases 6 and 7 may run concurrently** once phases 3 and 5 are green, with one
  caveat: T28's artifact inspection asserts the absence of the harness sentinels
  T21 introduces, so T21 must be complete before T28 — which phase 5's entry
  condition already guarantees.
- Within phase 3, T06 precedes T07–T09; T10 may begin as soon as T07 exists.
- Within phase 4, T11 precedes T12; T13 needs T05 and T11; T14 needs T03 and T13.
- Within phase 5, T15 and T16 precede everything else; T17, T19 and T20 may then
  proceed in parallel; **T18 follows T17**, because the OIDC callback consumes
  `assertApprovedRedirect()` and that ordering is made explicit rather than left
  as a hidden forward dependency; T21 follows T18.
- Within phase 6, T22 precedes T23–T26, which may then run in parallel.

**Work is never reordered to follow specification numbering.** 0B-SPEC-001 is not
implemented to completion before 0B-SPEC-002; each phase is a coherent vertical
slice that leaves the repository green.

## 26. Full task dependency graph

```
            T01 ──┬── T02 ──┐
                  └── T03 ──┴── T04 ──┬───────────────► T05
                                      │                 │
        ┌─────────────────────────────┴─────┐           │
        ▼ phase 3                           ▼ phase 4   │
       T06 ── T07 ─┬─ T08                  T11 ── T12   │
                   ├─ T09                   │           │
                   └─ T10                   └── T13 ◄───┘
                                                 │
                                                 └── T14
        phase 5
        T15 ── T16 ─┬─ T17 ── T18 ── T21
                    ├─ T19   (also needs T09, T14)
                    └─ T20   (also needs T13)

        phase 6                          phase 7
        T22 ─┬─ T23                      T27 ── T28   (T28 also needs T21)
             ├─ T24
             ├─ T25   (also needs T17)
             └─ T26   (also needs T18)

        phase 8            phase 9        phase 10
        T29 ── T30         T31            T32
```

| Task | Depends on | Task | Depends on |
|---|---|---|---|
| T01 | — | T18 | T16, T17 |
| T02 | T01 | T17 | T16 |
| T03 | T01 | T19 | T16, T09, T14 |
| T04 | T02, T03 | T20 | T16, T13 |
| T05 | T02, T04 | T21 | T16, T18 |
| T06 | T03, T05 | T22 | T09, T21 |
| T07 | T06 | T23 | T22 |
| T08 | T07 | T24 | T22 |
| T09 | T07 | T25 | T22, T17 |
| T10 | T07 | T26 | T22, T18 |
| T11 | T05 | T27 | T09 |
| T12 | T11 | T28 | T27, T21 |
| T13 | T05, T11 | T29 | T28 |
| T14 | T03, T13 | T30 | T29 |
| T15 | T05 | T31 | T01–T30 |
| T16 | T15 | T32 | T31 |

All 32 identifiers are unique and every dependency resolves to a task in this
set. **Passing an individual task does not imply milestone acceptance**; only
T32 and §28 decide that.

## 27. Acceptance-criteria mapping

All **102** allocated criteria, each mapped to the task that produces its
evidence. Verification types: **INS** inspection · **GATE** static blocking gate ·
**UNIT** · **COMP** component · **A11Y-A** accessibility automation ·
**A11Y-R** recorded accessibility review · **CON** contract · **E2E** browser ·
**SEC** security-negative · **MI** multi-instance · **ART** artifact inspection ·
**SCAN** supply-chain scan · **CI** CI evidence.

| AC | Owner | Type | Task(s) |
|---|---|---|---|
| 0B-AC-001 | S1 | GATE | T01, T30, T31 |
| 0B-AC-002 | S1 | GATE | T01, T30, T31 |
| 0B-AC-003 | S1 | INS, CI | T01, T31 |
| 0B-AC-004 | S1 | GATE | T01, T02 |
| 0B-AC-005 | S1 | GATE, INS | T04, T32 |
| 0B-AC-006 | S1 | COMP, E2E | T03, T10, T22 |
| 0B-AC-007 | S1 | GATE | T04 |
| 0B-AC-008 | S1 | INS, COMP | T03, T04 |
| 0B-AC-009 | S1 | GATE | T04 |
| 0B-AC-010 | S1 | INS | T04, T32 |
| 0B-AC-011 | S1 | GATE | T02, T31 |
| 0B-AC-012 | S1 | INS, SCAN | T01, T30 |
| 0B-AC-013 | S1 | CI | T01, T31, T32 |
| 0B-AC-020 | S2 | COMP | T06, T10 |
| 0B-AC-021 | S2 | COMP, INS | T06, T10 |
| 0B-AC-022 | S2 | INS, GATE | T06 |
| 0B-AC-023 | S2 | COMP | T07, T10 |
| 0B-AC-024 | S2 | COMP, E2E | T09, T10, T22 |
| 0B-AC-025 | S2 | INS, GATE | T04, T09 |
| 0B-AC-026 | S2 | COMP, E2E | T09, T22 |
| 0B-AC-027 | S2 | COMP, E2E | T09, T22 |
| 0B-AC-028 | S2 | COMP | T10, T19 |
| 0B-AC-029 | S2 | COMP | T08, T10 |
| 0B-AC-030 | S2 | COMP | T08, T10 |
| 0B-AC-031 | S2 | A11Y-A, COMP, A11Y-R | T10, T23 |
| 0B-AC-032 | S2 | COMP, A11Y-R | T06, T08, T23 |
| 0B-AC-033 | S2 | E2E | T22 |
| 0B-AC-034 | S2 | E2E | T07, T22 |
| 0B-AC-035 | S2 | E2E | T22 |
| 0B-AC-036 | S2 | A11Y-R | T23 |
| 0B-AC-040 | S3 | INS | T11 |
| 0B-AC-041 | S3 | INS, CI | T31, T32 |
| 0B-AC-042 | S3 | INS | T11 |
| 0B-AC-043 | S3 | GATE | T04, T11 |
| 0B-AC-044 | S3 | INS | T11, T28 |
| 0B-AC-045 | S3 | CON | T11 |
| 0B-AC-046 | S3 | CON | T12 |
| 0B-AC-047 | S3 | CON | T12 |
| 0B-AC-048 | S3 | CON | T12, T31 |
| 0B-AC-049 | S3 | INS, CON | T11, T12 |
| 0B-AC-050 | S3 | CON | T12 |
| 0B-AC-051 | S3 | COMP, GATE | T13 |
| 0B-AC-052 | S3 | COMP | T13 |
| 0B-AC-053 | S3 | COMP | T05, T13 |
| 0B-AC-054 | S3 | COMP | T13 |
| 0B-AC-055 | S3 | INS, ART | T13, T28 |
| 0B-AC-056 | S3 | COMP | T13 |
| 0B-AC-057 | S3 | INS | T13 |
| 0B-AC-058 | S3 | COMP | T14 |
| 0B-AC-060 | S4 | COMP, GATE | T05 |
| 0B-AC-061 | S4 | COMP, ART | T05, T28 |
| 0B-AC-062 | S4 | COMP | T05 |
| 0B-AC-063 | S4 | COMP | T15 |
| 0B-AC-064 | S4 | COMP, INS | T15 |
| 0B-AC-065 | S4 | E2E | T16, T24 |
| 0B-AC-066 | S4 | E2E | T24 |
| 0B-AC-067 | S4 | E2E | T24 |
| 0B-AC-068 | S4 | INS, COMP | T16 |
| 0B-AC-069 | S4 | MI | T26 |
| 0B-AC-070 | S4 | COMP, E2E | T18 |
| 0B-AC-071 | S4 | E2E, MI | T18, T26 |
| 0B-AC-072 | S4 | E2E | T18, T24 |
| 0B-AC-073 | S4 | INS | T16, T18 |
| 0B-AC-074 | S4 | SEC | T17, T25 |
| 0B-AC-075 | S4 | COMP | T15, T16 |
| 0B-AC-076 | S4 | INS, SCAN | T30 |
| 0B-AC-077 | S4 | COMP | T15, T19 |
| 0B-AC-078 | S4 | COMP | T16, T19 |
| 0B-AC-079 | S4 | E2E | T16, T24 |
| 0B-AC-080 | S4 | COMP | T14, T19 |
| 0B-AC-081 | S4 | COMP | T19 |
| 0B-AC-082 | S4 | SEC | T25 |
| 0B-AC-083 | S4 | COMP, SEC | T19, T25 |
| 0B-AC-084 | S4 | SEC | T17, T25 |
| 0B-AC-085 | S4 | E2E | T20, T24 |
| 0B-AC-086 | S4 | INS, ART | T21, T28 |
| 0B-AC-090 | S5 | INS | T10, T32 |
| 0B-AC-091 | S5 | CI | T31 |
| 0B-AC-092 | S5 | GATE, ART | T04, T28 |
| 0B-AC-093 | S5 | CI | T31 |
| 0B-AC-094 | S5 | GATE | T02 |
| 0B-AC-095 | S5 | GATE | T04 |
| 0B-AC-096 | S5 | COMP | T10 |
| 0B-AC-097 | S5 | A11Y-A, A11Y-R | T23 |
| 0B-AC-098 | S5 | E2E | T22 |
| 0B-AC-099 | S5 | CON | T12 |
| 0B-AC-100 | S5 | CON | T12 |
| 0B-AC-101 | S5 | E2E | T24 |
| 0B-AC-102 | S5 | E2E | T24 |
| 0B-AC-103 | S5 | SEC | T25 |
| 0B-AC-104 | S5 | MI | T26 |
| 0B-AC-105 | S5 | SEC | T25 |
| 0B-AC-106 | S5 | E2E | T24 |
| 0B-AC-107 | S5 | ART, E2E | T27 |
| 0B-AC-108 | S5 | ART, E2E | T27 |
| 0B-AC-109 | S5 | GATE | T04 |
| 0B-AC-110 | S5 | ART | T28 |
| 0B-AC-111 | S5 | SCAN | T30 |
| 0B-AC-112 | S5 | SCAN | T30 |
| 0B-AC-113 | S5 | GATE | T28 |
| 0B-AC-114 | S5 | GATE, SCAN | T29, T30 |
| 0B-AC-115 | S5 | CI | T31 |

**Coverage: 102 / 102.** Every allocated criterion has at least one owning task.

**Permanently unallocated identifiers:** 0B-AC-014–019, 037–039, 059, 087–089,
116–119. The invariant is that a permanently unallocated identifier **must never
be assigned to a requirement, task, acceptance mapping or implementation claim**.
Listing them as reserved and unallocated — as this paragraph does — is
documentation, not assignment, so a literal search that finds one here is not a
failure.

**Criteria that are review-based and must never be described as automated
results** (S5 R22): 0B-AC-010, 0B-AC-031 (in part), 0B-AC-032 (in part),
0B-AC-036, 0B-AC-041, 0B-AC-042, 0B-AC-044, 0B-AC-057, 0B-AC-073.

## 28. Milestone-exit evidence requirements

0B exits against its own 102 acceptance criteria and the CI evidence demonstrating
them, together with the §81.1 conditions as they apply at this milestone. 0B
carries no database and no migrations and delivers no product capability.

**AT reconciliation, preserved exactly as the specifications state it and
extended by nothing:**

- **AT-035 — owned by S5 and fully evidenced at 0B** by 0B-AC-107 and 0B-AC-108,
  at the production artifact and runtime boundary, subject to the S5 R49 scope.
- **AT-027 — fully discharged** (0B-AC-066 states it; 0B-AC-101 executes it).
- **AT-029 — fully discharged** (0B-AC-084 states it; 0B-AC-103 executes it).
- **AT-028 — partial.** 0B-AC-106 executes only the browser half against a
  synthetic authority; real server session and token-cache invalidation is M1.
- **AT-026 — partial.** Only the non-leakage half; central token rejection is M1.
- **AT-037 — partial.** Only the negative half; the bootstrap flow is backend/M1.
- **AT-017 — partial.** Evidenced only for the shell, primitives and state
  classes; satisfied by the milestone delivering the workflows it names.
- **AT-025, AT-030, AT-031, AT-042** — frontend preconditions only, not
  discharged.
- All remaining §75 entries are backend or product capability at M1 and later.

**Non-claims that must appear verbatim in the verification report** (S5 R38):
no real backend token rejection, session invalidation, server-side token-cache
invalidation, refresh-token rotation, LOCAL credential verification,
federated-token validation, OIDC tenant integration or backend authorization is
claimed; and no evidence demonstrates compatibility with `knowhub-backend`, the
existence of any backend endpoint, or a frontend/backend version pair.

## 29. M1 and deployment deferrals, and anti-scope-creep controls

Deferrals are listed in §4 and in each specification's own Deferred table; none
is restated as a requirement here.

**Controls that make scope creep mechanically visible, not merely discouraged:**

1. The directory-inventory assertion fails if a directory appears without a
   milestone justification (0B-AC-005).
2. The boundaries rule forbids `src/**` importing `tests/**`, so no fixture can
   quietly become production data.
3. Artifact inspection fails on any test sentinel in the production bundle.
4. The permissions port returns `{ known: false }`, so any capability someone
   adds is invisible until M1 binds a real contract — creep cannot ship silently.
5. `contracts/` holds only a governance note, so a manufactured contract shows up
   as a new file in a governed directory.
6. The published contract-evidence artifact states the absence of a bound product
   contract explicitly, so a later compatibility claim contradicts a published
   record.
7. Every task file carries an explicit **What must NOT be implemented** section.

## 30. Known risks and implementation stop conditions

| # | Risk | Mitigation | Stop condition |
|---|---|---|---|
| 1 | Next 16 nonce propagation with `'strict-dynamic'` behaves differently than documented, or a document under the nonce CSP is statically rendered | verify both in T27 before wiring the gate | if S5 R40 cannot be met without `'unsafe-eval'` — **stop and surface**; never weaken the CSP. If a document cannot be dynamically rendered, it must not be served under the nonce CSP unverified |
| 1b | An `openid-client` API performs federated validation S4 R34 reserves to the backend | only the primitives in §13 are used; `authorizationCodeGrant()` and the other listed APIs are prohibited | if the protocol mechanics 0B owns cannot be performed without a forbidden API, **stop** and replace the library rather than working around the specification |
| 2 | Radix inline style attributes need more than `style-src-attr 'unsafe-inline'` | enumerate every inline style during T07 | if inline `<style>` elements prove unavoidable, surface the trade-off; never widen `style-src` silently |
| 3 | The pinned Chromium build's handling of `Secure`/`__Host-` cookies over `http://localhost` is **unverified until T22 executes the check** | verify against the pinned build; if rejected, the browser harness uses local TLS | never relax `Secure`, never drop the `__Host-` prefix, never create an insecure profile |
| 4 | `openapi-typescript` output is not byte-stable | pin by lockfile, normalise line endings, verify first in T11 | if determinism is unachievable, 0B-AC-046 fails — surface before proceeding |
| 5 | Unprivileged network namespaces unavailable on the runner | `docker run --network none` fallback | — |
| 6 | Cross-instance OIDC needs more than the sealed cookie | — | any need for shared frontend **runtime state**, sticky routing, a durable frontend session store, browser token custody or frontend entitlement authority is an **ADR-027 §84.4 stop** |
| 7 | The authority seam cannot be aliased without a test artifact entering the production build | T28 artifact inspection catches it immediately | if unavoidable, S5 R10 is at risk — **stop and surface** |
| 8 | Type-aware lint performance | project references or scoped type-aware rules | — |
| 9 | Base-image CVEs exceed the Trivy policy | `--ignore-unfixed` plus base-image patch bumps | never widen the severity threshold to pass |
| 10 | Node patch below 24.15 | baseline is 24.21.0 | any downgrade below 24.15 breaks `jsdom@30.1.0` and the component layer |
| 11 | A recorded review is mistaken for automated evidence | every review row labelled `inspection` or `targeted assertion` | never fabricate a metric or express an aesthetic judgement as a threshold (S5 R22) |

**General stop rule (§84.4).** Where a requirement admits two readings, or where a
constraint would have to be made more convenient than it was written, stop and
surface both sources with citations. Never soften a security, authorization,
provenance or auditability constraint to make a task tractable.

## 31. ADR assessment

**No new ADR is required for this plan, and none is created.** The register's
next number remains **ADR-028**, unclaimed.

Assessed against the ADR threshold — new persistent or shared runtime
infrastructure; new durable frontend session or authorization authority; a
changed repository boundary; a new browser telemetry architecture; a changed
deployment model or security authority; anything contradicting an Accepted ADR:

| Candidate | Assessment |
|---|---|
| Sealed category-B transaction cookie | introduces no persistent or shared infrastructure — the key is configuration; carries no identity, permission or session authority; satisfies all eight S4 R30 properties. S4 R30 permits a mechanism and S5 §14 states one is **not** implicated merely because it works across instances. ADR-027 rejected sealed **session** material, which this is not. **No ADR.** |
| Out-of-process synthetic authority | test-only; stands in for the M1 backend; S5 R5 expressly permits locally hosted synthetic authorities; ships in nothing. **No ADR.** |
| Two build profiles | a verification technique. S5 R11 mandates no containment mechanism and leaves it to the plan. **No ADR.** |
| GitHub Actions | §0E.1 already permits it; S5 R76 fixes no provider. **No ADR.** |
| `openid-client`, `jose` | S4 §6 makes the OIDC library a plan choice unless it relocates session authority, requires browser token custody or moves entitlement into the frontend. Neither does. **No ADR.** |
| Versions, scanners, formatter, test paths, CI topology, Docker stages | ordinary plan-level choices under the `knowhub-sdd` test. **No ADR.** |

**Two watch-points that would change this answer** and must halt implementation:

1. the category-B mechanism requiring shared **runtime state** rather than shared
   configuration — ADR-027 §2 and §6 territory, needing a superseding ADR;
2. any authentication or session library forcing a durable frontend session
   store, browser-side token custody, or a frontend entitlement decision —
   breaching ADR-021, ADR-022 or ADR-027.

**No conflict was found** between this plan, the five Approved specifications, the
Accepted ADRs and the blueprint.

## 32. When 0B may be considered accepted

0B is accepted only when **every** item below holds. Existence of files or
classes is never sufficient (§81.1).

1. All five Approved specifications implemented, every normative requirement
   traceable to an implementation reference.
2. All **102** allocated acceptance criteria evidenced by the mechanism §27
   assigns, with review-based criteria identified as such and never presented as
   automated results.
3. Every inherited verification obligation in S5 §2.1 demonstrated by the layer
   that owns its machinery, with no normative ownership transferred.
4. AT reconciliation preserved exactly as §28 states it, with no claim beyond it.
5. A complete green CI run: every S5 R71 gate present and blocking, none
   advisory, no placeholder job, no empty test layer, and each gate demonstrated
   capable of failing.
6. The production build succeeds, with a strict type error and a lint violation
   each blocking it.
7. The production image builds as `knowhub-frontend:<git-sha>` and is scanned.
8. Dependency, secret and container scans pass under the stated policy, a
   zero-finding result recorded as a pass.
9. CSP and security headers verified against the production runtime, with the
   S5 R49 scope statement attached.
10. The production bundle and every emitted source map contain no secret,
    credential, token, resolved provider credential, server-only configuration
    value, or test-only fixture, double, authority or identifier.
11. Repository independence demonstrated: the complete sequence with only
    `knowhub-frontend` checked out.
12. No M1 scope creep: §29's control list verified.
13. Working trees clean across all three repositories; no committed secret,
    token, snapshot or key.
14. `0B-VERIFICATION-REPORT.md` exists in `knowhub-specs`, mapping every
    requirement and every acceptance criterion to its evidence and recording
    every non-claim verbatim.

**Only after acceptance** does this plan move from `04-plans/current/` to
`04-plans/completed/`. It is **not** moved during planning, and 0B is **not**
declared complete by the existence of this document.
