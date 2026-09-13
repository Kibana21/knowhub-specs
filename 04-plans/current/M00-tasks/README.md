# M00 implementation tasks

Tasks derived from
[`M00-PLAN-backend-foundation.md`](../M00-PLAN-backend-foundation.md).

These files introduce no requirement. Every requirement reference points at
an approved M00 specification; every acceptance criterion is one of
`M00-AC-001` – `M00-AC-026`. Where a task and the plan disagree, the plan
wins; where the plan and a specification disagree, the specification wins.

## Order

| Task | Title | Depends on |
|---|---|---|
| [T01](M00-T01-repository-skeleton-and-dependency-baseline.md) | Repository skeleton and dependency baseline | — |
| [T02a](M00-T02a-managed-compose-dependency-environment.md) | Managed Compose dependency environment | T01 |
| [T03](M00-T03-configuration-package.md) | Configuration package | T01 |
| [T02b](M00-T02b-configured-endpoint-dependency-verification.md) | Configured-endpoint dependency verification | T02a, T03 |
| [T04](M00-T04-observability-package.md) | Observability package | T01, T03 |
| [T05](M00-T05-minimal-api-foundation.md) | Minimal API foundation | T03, T04 |
| [T06](M00-T06-boundary-enforcement.md) | Boundary enforcement | T03, T04, T05 |
| [T07](M00-T07-alembic-scaffolding.md) | Alembic scaffolding | T02a, T03 |
| [T08](M00-T08-test-harness-and-failure-paths.md) | Test harness and failure-path tests | T02b, T03, T04, T05, T07 |
| [T09](M00-T09-container-image-and-process-mode.md) | Container image and process mode | T05 |
| [T10](M00-T10-ci-wiring.md) | CI wiring | T01–T09 |
| [T11](M00-T11-verification-pass.md) | M00 verification pass | T10 |

T02a and T03 may proceed in parallel after T01. T02b waits on T03 so that
configured endpoints are read through the canonical settings model and no
second configuration mechanism appears — see the plan's sequencing rule.
