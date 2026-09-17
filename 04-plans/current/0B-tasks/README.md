# 0B implementation tasks

Tasks derived from
[`0B-PLAN-frontend-foundation.md`](../0B-PLAN-frontend-foundation.md).

These files introduce **no requirement**. Every requirement reference points at
an Approved 0B specification; every acceptance criterion is one of the 102
allocated identifiers `0B-AC-001` – `0B-AC-115`. Where a task and the plan
disagree, the plan wins; where the plan and a specification disagree, **the
specification wins and implementation stops** until the conflict is surfaced.

Throughout, **`S1`–`S5`** abbreviate 0B-SPEC-001 – 0B-SPEC-005; `S4 R30` means
requirement R30 of 0B-SPEC-004.

> **Passing an individual task does not imply milestone acceptance.**
> A task is complete when its own completion definition is met. Milestone 0B is
> accepted only under §32 of the plan, evidenced by `0B-T32`.

## Ordered task table

| Task | Title | Phase | Depends on |
|---|---|---|---|
| [T01](0B-T01-repository-skeleton-and-toolchain-baseline.md) | Repository skeleton and toolchain baseline | 1 | — |
| [T02](0B-T02-static-quality-gates-and-failure-demonstrations.md) | Static quality gates and failure demonstrations | 1 | T01 |
| [T03](0B-T03-app-router-root-layout-and-provider-composition.md) | App Router root layout and provider composition | 1 | T01 |
| [T04](0B-T04-module-boundary-and-server-only-enforcement.md) | Module-boundary, server-only and unsafe-DOM enforcement | 2 | T02, T03 |
| [T05](0B-T05-typed-configuration-boundary.md) | Typed configuration boundary | 2 | T02, T04 |
| [T06](0B-T06-design-tokens-and-theme-single-source.md) | Design tokens and the theme single source | 3 | T03, T05 |
| [T07](0B-T07-primitive-layer.md) | The primitive layer | 3 | T06 |
| [T08](0B-T08-ui-state-classes-and-error-surfaces.md) | UI state classes and safe error surfaces | 3 | T07 |
| [T09](0B-T09-application-shell.md) | The application shell | 3 | T07 |
| [T10](0B-T10-vitest-harness-and-component-layer.md) | Vitest harness, component layer and structural accessibility | 3 | T07 |
| [T11](0B-T11-synthetic-contract-fixture-and-generation-entry-point.md) | Synthetic contract fixture and the generation entry point | 4 | T05 |
| [T12](0B-T12-contract-freshness-and-drift-detection.md) | Contract freshness, drift detection and offline generation | 4 | T11 |
| [T13](0B-T13-api-boundary-correlation-and-normalized-error.md) | The API boundary, correlation and the normalized error | 4 | T05, T11 |
| [T14](0B-T14-query-client-and-cache-key-convention.md) | Query client and cache-key convention | 4 | T03, T13 |
| [T15](0B-T15-provider-selection-and-canonical-session.md) | Provider-selection shell and the canonical session | 5 | T05 |
| [T16](0B-T16-session-authority-port-and-session-cookie.md) | Session authority port, binding seam and session cookie | 5 | T15 |
| [T17](0B-T17-csrf-origin-validation-and-redirect-allowlist.md) | CSRF, Origin validation and the redirect allowlist | 5 | T16 |
| [T18](0B-T18-oidc-shell-and-sealed-transaction-state.md) | OIDC shell, PKCE and sealed transaction state | 5 | T16, T17 |
| [T19](0B-T19-route-guard-permissions-port-and-access-components.md) | Route/session guard, permissions port and access components | 5 | T16, T09, T14 |
| [T20](0B-T20-auth-and-session-safe-diagnostics.md) | Auth and session safe diagnostics | 5 | T16, T13 |
| [T21](0B-T21-synthetic-authority-and-local-oidc-provider.md) | Synthetic authority and local OIDC provider | 5 | T16, T18 |
| [T22](0B-T22-playwright-harness-and-shell-journeys.md) | Playwright harness and shell journeys | 6 | T09, T21 |
| [T23](0B-T23-accessibility-pass-and-recorded-review.md) | Authoritative accessibility pass and recorded review | 6 | T22 |
| [T24](0B-T24-auth-browser-journeys-and-credential-inspection.md) | Auth browser journeys and credential inspection | 6 | T22 |
| [T25](0B-T25-security-negative-layer.md) | The security-negative layer | 6 | T22, T17 |
| [T26](0B-T26-multi-instance-verification.md) | Multi-instance verification | 6 | T22, T18 |
| [T27](0B-T27-production-csp-and-security-headers.md) | Production CSP and security headers | 7 | T09 |
| [T28](0B-T28-production-build-and-artifact-inspection.md) | Production build and artifact inspection | 7 | T27, T21 |
| [T29](0B-T29-container-image-and-release-artifact-identity.md) | Container image and release-artifact identity | 8 | T28 |
| [T30](0B-T30-supply-chain-gates.md) | Supply-chain gates | 8 | T29 |
| [T31](0B-T31-ci-workflow-wiring-and-evidence-artifacts.md) | CI workflow wiring and evidence artifacts | 9 | T01–T30 |
| [T32](0B-T32-acceptance-verification-pass.md) | 0B acceptance verification pass | 10 | T31 |

## Phase grouping

| Phase | Name | Tasks |
|---|---|---|
| 1 | Toolchain and repository skeleton | T01, T02, T03 |
| 2 | Architecture boundaries and configuration | T04, T05 |
| 3 | Design tokens, primitives, shell, UI states | T06 – T10 |
| 4 | Contract pipeline and API/query boundary | T11 – T14 |
| 5 | BFF, session, OIDC, CSRF, guard, permissions | T15 – T21 |
| 6 | Browser, accessibility, security-negative, multi-instance | T22 – T26 |
| 7 | Production hardening and build-artifact verification | T27, T28 |
| 8 | Container and supply chain | T29, T30 |
| 9 | CI wiring | T31 |
| 10 | Acceptance verification | T32 |

## Dependency graph

```
            T01 ──┬── T02 ──┐
                  └── T03 ──┴── T04 ──► T05
                                        │
        ┌───────────────────────────────┴──────────┐
        ▼ phase 3                        phase 4 ▼
       T06 ── T07 ─┬─ T08                 T11 ── T12
                   ├─ T09                  │
                   └─ T10                  └── T13 ── T14
        phase 5
        T15 ── T16 ─┬─ T17 ── T18 ── T21
                    ├─ T19        (also needs T09, T14)
                    └─ T20        (also needs T13)

        phase 6                        phase 7
        T22 ─┬─ T23                    T27 ── T28   (T28 also needs T21)
             ├─ T24
             ├─ T25  (also needs T17)
             └─ T26  (also needs T18)

        phase 8           phase 9      phase 10
        T29 ── T30        T31          T32
```

## Tasks that may execute in parallel

- **Phases 3 and 4 run concurrently** once phase 2 is green: the design system
  and the contract pipeline share no file and no dependency.
- **Phases 6 and 7 run concurrently** once phases 3 and 5 are green. One caveat:
  T28 asserts the absence of the harness sentinels T21 introduces, so T21 must be
  complete first — phase 5's entry condition already guarantees this.
- Within phase 1: **T02 and T03 in parallel** after T01.
- Within phase 3: T06 first; then **T07**; then **T08, T09 and T10 in parallel**.
- Within phase 4: T11 first; then **T12 and T13 in parallel**; T14 after T13.
- Within phase 5: T15 then T16; then **T17, T19 and T20 in parallel**; **T18
  after T17** (the OIDC callback consumes `assertApprovedRedirect()`); T21 after
  T18.
- Within phase 6: T22 first; then **T23, T24, T25 and T26 in parallel**.

## Milestone-level gates

A phase is **entered** only when every prerequisite phase's completion evidence
exists, and **left** only when every task in it meets its completion definition
and `pnpm verify` is green.

| After phase | Gate that must hold |
|---|---|
| 1 | `pnpm install --frozen-lockfile`, lint, format, typecheck and build all green on a clean checkout |
| 2 | boundary, server-only and unsafe-DOM gates each demonstrated failing on a deliberate violation and green once removed |
| 3 | component and structural-accessibility suites green; token-change test passes |
| 4 | all four drift cases detected and restored; generation byte-identical and offline |
| 5 | six-state component matrix green; no backend endpoint shape declared anywhere; no forbidden `openid-client` API used; no ID-token nonce-claim validation claimed |
| 6 | Playwright report green on Chromium; recorded accessibility review committed |
| 7 | header and artifact reports green against the **production** build |
| 8 | image builds; all three scans pass under the stated policy |
| 9 | full CI green with only `knowhub-frontend` checked out |
| 10 | `0B-VERIFICATION-REPORT.md` complete; every one of the 102 criteria evidenced |

## Rules for moving between phases

1. **Never reorder work to follow specification numbering.** 0B-SPEC-001 is not
   implemented to completion before 0B-SPEC-002; each phase is a vertical slice.
2. **Leave the repository green.** A phase does not end with a failing gate, a
   skipped test or a disabled rule.
3. **Pair every capability with its gate in the same phase.** A gate introduced
   in a later phase would leave the property unproved in between.
4. **Demonstrate each new gate failing.** A gate that has never failed has not
   been shown to work (S5 R6, R17).
5. **Stop rather than guess.** Where a requirement admits two readings, or a
   constraint would have to be made more convenient than written, stop and
   surface both sources with citations (§84.4).
6. **Never weaken a security, authorization, provenance or auditability
   constraint** to make a task tractable.
