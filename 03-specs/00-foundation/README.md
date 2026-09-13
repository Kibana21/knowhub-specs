# M00 — Backend Foundation (§81 milestone 0A)

Specifications for milestone **0A** of the
[delivery roadmap](../../00-start-here/delivery-roadmap.md), the
authoritative build order extracted from master blueprint
[§81 Implementation dependency graph & execution order](../../../knowhub_master_blueprint.html#implementation-dependency-graph).

This directory is the specification layer for M00 only. It contains no
implementation plan, no tasks and no application code.

## Scope

§81 states milestone 0A as:

> Backend foundation — `knowhub-backend`: pyproject/uv, Python package
> boundaries, CI, settings, OTel, local dependencies

All six items are owned by exactly one specification below. Nothing in M00
is owned by two specifications, and no §81 0A item is unowned.

| §81 0A item | Owning specification |
|---|---|
| pyproject/uv | M00-SPEC-001 |
| Python package boundaries | M00-SPEC-001 |
| settings | M00-SPEC-002 |
| OTel | M00-SPEC-003 |
| CI | M00-SPEC-004 |
| local dependencies | M00-SPEC-004 |

## Specifications

| Specification | Purpose | Status |
|---|---|---|
| [M00-SPEC-001 — Backend repository and package boundaries](M00-SPEC-001-backend-repository-and-package-boundaries.md) | Repository shape, `uv`/`pyproject` dependency management, Python baseline, the M00 package inventory, package dependency boundaries and their CI enforcement, lint/format/type tooling, and the minimal bootable FastAPI application. | Approved |
| [M00-SPEC-002 — Backend configuration and settings](M00-SPEC-002-backend-configuration-and-settings.md) | Typed settings, configuration domains, precedence, secret-reference indirection and the Local environment profile. | Approved |
| [M00-SPEC-003 — Backend observability baseline](M00-SPEC-003-backend-observability-baseline.md) | OpenTelemetry bootstrap, vendor-neutral exporter configuration, the correlation/run-identifier propagation contract and safe-telemetry rules. | Approved |
| [M00-SPEC-004 — Backend build, CI and local dependencies](M00-SPEC-004-backend-build-ci-and-local-dependencies.md) | Local dependency services, Alembic scaffolding, the backend CI gate set, container image and process-mode structure, and the test-layer layout. | Approved |

## Acceptance criterion index

M00 acceptance criteria use milestone-scoped identifiers (`M00-AC-nnn`).
They are deliberately **not** allocated from the blueprint's global
`AT-nnn` catalogue in
[§75](../../../knowhub_master_blueprint.html#acceptance-tests), which
contains no foundation-level entry.

| Range | Owning specification |
|---|---|
| M00-AC-001 – M00-AC-008 | M00-SPEC-001 |
| M00-AC-009 – M00-AC-013 | M00-SPEC-002 |
| M00-AC-014 – M00-AC-017 | M00-SPEC-003 |
| M00-AC-018 – M00-AC-026 | M00-SPEC-004 |

## Milestone exit

Milestone transitions are gated by
[§81.1](../../../knowhub_master_blueprint.html#implementation-dependency-graph).
§81.1 refers to golden acceptance tests, but
[§75](../../../knowhub_master_blueprint.html#acceptance-tests) defines no
acceptance test at the foundation layer — every `AT-nnn` entry presupposes
capability delivered in M1 or later.

M00 therefore exits against its own approved `M00-AC-nnn` acceptance
criteria and the CI evidence that demonstrates them, together with the
remaining §81.1 conditions as they apply at this milestone. M00 carries no
business schema and publishes no public API contract; see
[M00-SPEC-004](M00-SPEC-004-backend-build-ci-and-local-dependencies.md) for
how migration repeatability is demonstrated on an empty chain.

## Reading order

1. [`../../02-adrs/README.md`](../../02-adrs/README.md) — the live ADR register.
2. [`../../05-reference/glossary.md`](../../05-reference/glossary.md) — normative terminology.
3. The four specifications above, in number order.
