# 0B — Frontend Foundation (§81 milestone 0B)

Specifications for milestone **0B** of the
[delivery roadmap](../../00-start-here/delivery-roadmap.md), the
authoritative build order extracted from master blueprint
[§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph).

This directory is the specification layer for 0B only. It contains no
implementation plan, no tasks and no application code.

## Scope

§81 states milestone 0B as:

> Frontend foundation — `knowhub-frontend`: Next.js/TypeScript, bright design
> system, LOCAL/OIDC BFF session shell, CI, + pinned OpenAPI generation
> pipeline

All five items are owned by exactly one specification below. Nothing in 0B is
owned by two specifications, and no §81 0B item is unowned.

| §81 0B item | Owning specification |
|---|---|
| Next.js/TypeScript | 0B-SPEC-001 |
| bright design system | 0B-SPEC-002 |
| LOCAL/OIDC BFF session shell | 0B-SPEC-004 |
| CI | 0B-SPEC-005 |
| pinned OpenAPI generation pipeline | 0B-SPEC-003 |

## Specifications

| Specification | Purpose | Status |
|---|---|---|
| [0B-SPEC-001 — Frontend repository, toolchain and application architecture](0B-SPEC-001-frontend-repository-toolchain-and-application-architecture.md) | Repository shape, `pnpm` dependency management, language and runtime baseline, source layout, frontend layering and module boundaries, App Router architecture, quality-tooling outcomes and repository independence. | Approved |
| [0B-SPEC-002 — Application shell, design system and accessibility](0B-SPEC-002-application-shell-design-system-and-accessibility.md) | Design tokens and theme policy, the reusable primitive layer, the application shell as a composition mechanism, UI state classes, safe error presentation, accessibility and responsive behaviour. | Approved |
| [0B-SPEC-003 — OpenAPI contract and API client foundation](0B-SPEC-003-openapi-contract-and-api-client-foundation.md) | Contract artifact governance, the reproducible generation pipeline and its freshness property, the single API access boundary, correlation propagation, safe API error normalization, the streaming boundary and the server-state foundation. | Approved |
| [0B-SPEC-004 — Configuration, BFF session and authorization foundations](0B-SPEC-004-configuration-bff-session-and-authorization-foundations.md) | The typed frontend configuration boundary and its server-only versus browser-exposed split, the LOCAL/OIDC/HYBRID provider-selection shell, the browser and BFF session boundary, the OIDC protocol shell, the session state and lifecycle contract, the authorization and route-guard foundation, CSRF and redirect safety, auth and session safe diagnostics, and the test-only identity and session doubles. | Approved |
| [0B-SPEC-005 — Testing, security, production build and CI](0B-SPEC-005-testing-security-production-build-and-ci.md) | Test architecture and deterministic, repository-independent execution, test-only isolation from production, the static quality gates, the component, accessibility, contract, browser, multi-instance and security-negative verification layers, the production browser-hardening baseline delegated by 0B-SPEC-004, dependency and supply-chain security, the production build and container release artifact, and the CI gate set. | Approved |

Specifications are created incrementally, in the order above. A row carrying
`—` has no approved content and no requirements; its title records ownership
of a §81 0B item, nothing more.

## Acceptance criterion index

0B acceptance criteria use milestone-scoped identifiers (`0B-AC-nnn`). They
are deliberately **not** allocated from, and are not aliases for, the
blueprint's global `AT-nnn` catalogue in
[§75](../../../knowhub_master_blueprint.html#acceptance-tests). Where a
foundation behaviour directly contributes to a global `AT-nnn` entry, the
owning specification records that relationship explicitly, stating whether the
evidence is full or partial; the two identifier spaces remain independent.

Because 0B specifications are authored incrementally rather than together,
identifiers are allocated as **reserved blocks** rather than contiguously.
Unused numbers inside a block stay unused; an identifier is never reused or
reassigned.

| Reserved range | Allocated | Owning specification |
|---|---|---|
| 0B-AC-001 – 0B-AC-019 | 0B-AC-001 – 0B-AC-013 | 0B-SPEC-001 |
| 0B-AC-020 – 0B-AC-039 | 0B-AC-020 – 0B-AC-036 | 0B-SPEC-002 |
| 0B-AC-040 – 0B-AC-059 | 0B-AC-040 – 0B-AC-058 | 0B-SPEC-003 |
| 0B-AC-060 – 0B-AC-089 | 0B-AC-060 – 0B-AC-086 | 0B-SPEC-004 |
| 0B-AC-090 – 0B-AC-119 | 0B-AC-090 – 0B-AC-115 | 0B-SPEC-005 |

## Milestone exit

Milestone transitions are gated by
[§81.1](../../../knowhub_master_blueprint.html#implementation-dependency-graph).
§81.1 refers to golden acceptance tests. Some
[§75](../../../knowhub_master_blueprint.html#acceptance-tests) catalogue
entries have full or partial 0B evidence, as recorded by the owning
specification; others depend on capability delivered in M1 or later.

0B therefore exits against its own approved `0B-AC-nnn` acceptance criteria
and the CI evidence that demonstrates them, together with the remaining
§81.1 conditions as they apply at this milestone. 0B carries no database and
no migrations; it delivers no product capability.

## Reading order

1. [`../../02-adrs/README.md`](../../02-adrs/README.md) — the live ADR register.
2. [`../../05-reference/glossary.md`](../../05-reference/glossary.md) — normative terminology.
3. The specifications above, in number order.
