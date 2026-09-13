# M00-SPEC-004 — Backend build, CI and local dependencies

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Repository:** `knowhub-backend`
- **Status:** Approved
- **§81 0A items owned:** `CI`, `local dependencies`

## 1. Purpose

Define how `knowhub-backend` is built, validated and run at M00: the local
dependency services a developer brings up, the Alembic scaffolding that
makes migrations repeatable before any schema exists, the CI gate set, the
container image and its process-mode structure, and the test layout.

## 2. Sources

**Blueprint**

- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**; the Independent CI/CD pipelines table and the Local
  development block
- [§0E.7 Recommended V1 deployment topology](../../../knowhub_master_blueprint.html#runtime-architecture)
  — one versioned Python image started in different modes
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Persistence, Semantic retrieval, Lexical retrieval, Background
  execution, Cache/coordination, Large/raw artifacts, Packaging, CI/CD and
  Testing rows
- [§30.1 Backend runtime processes from one backend codebase](../../../knowhub_master_blueprint.html#target-python)
- [§71.3 Cross-repository rules](../../../knowhub_master_blueprint.html#package-dependency-rules)
  — independent CI
- [§76 Environment & deployment matrix](../../../knowhub_master_blueprint.html#environment-matrix)
  — the Local column — and §76.1 (promotion rules)
- [§75.1 Test layers](../../../knowhub_master_blueprint.html#acceptance-tests)
- [§81.1 Milestone exit rule](../../../knowhub_master_blueprint.html#implementation-dependency-graph)
- [§84.2 Definition of a completed implementation task](../../../knowhub_master_blueprint.html#coding-harness-instructions)
  and [§84.3 Repository hygiene](../../../knowhub_master_blueprint.html#coding-harness-instructions)
- [§65.3 Security, privacy and compliance constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)

**ADRs**

- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
- [ADR-001 — KnowHub is Python-first and API-first](../../02-adrs/ADR-001-python-first-api-first.md)
- ADR-003 (PostgreSQL is canonical), ADR-004 (pgvector and Postgres FTS
  first), ADR-006 (Taskiq and Redis for V1 asynchronous work) and ADR-015
  (internal domain events modeled, bus deferred) — canonical files in
  `knowhub-backend/docs/adr/` per the
  [ADR register](../../02-adrs/README.md)
- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
  (forward constraint; see §8)

## 3. Normative requirements — local dependency services

**R1.** The repository provides a Compose definition that brings up the
backend's local dependency services, and a documented bring-up sequence a
developer can follow on a clean machine. Each service declares a health
check, so that "the dependency is available" is an observable condition in
local development and in CI rather than an assumption.
*(§0E.8 Local development block; §30; §61)*

**R2.** The M00 local dependency set is exactly **PostgreSQL, Redis and
Azurite**.
*(§0E.8 Local development block; §76 Local column)*

**R3.** The local PostgreSQL service must be compatible with the intended
deployment target and must make the **pgvector** extension available.
*(§0E.1 Persistence and Semantic retrieval rows; ADR-003; ADR-004)*

**R4.** No PostgreSQL major version is fixed by this specification, because
neither the blueprint nor an accepted ADR specifies one. The implementation
plan selects the version and image and **must pin them explicitly**;
an unpinned or floating tag is not acceptable.
*(§76.1 — environments are reproducible; §0E.1 Infrastructure as code row)*

**R5.** Redis is present as a local service because it is the V1 broker and
coordination store. M00 implements no broker, queue, task, lock or cache
usage, and must not add a Redis client, a fake adapter or any placeholder
application integration in order to have something to test. Redis is proven
**healthy and reachable** at M00; it is **exercised** by the M2
functionality that first uses it.
*(§0E.1 Cache/coordination row; ADR-006; §84.1 — deliver working vertical
slices, not a large set of empty interfaces)*

**R6.** Azurite is the local object-storage service. M00 implements no Blob
abstraction, adapter or client, and must not add one — nor a fake adapter or
placeholder integration — in order to have something to test. Azurite is
proven **healthy and reachable** at M00; it is **exercised** by the M2
functionality that first uses the Blob abstraction. §76 permits a local
filesystem abstraction as an alternative, and §0E.8's local development
block names Azurite, which M00 follows.
*(§0E.8; §76 Blob row; §0E.1 Large/raw artifacts row; §84.1)*

**R7.** No additional infrastructure may be introduced into the local
dependency set. Neo4j, an enterprise search platform, Service Bus or Kafka
each require an ADR and measured justification.
*(§0E.9; §84.1; ADR-015)*

**R8.** Local services carry non-production credentials supplied by local
configuration per
[M00-SPEC-002](M00-SPEC-002-backend-configuration-and-settings.md) R19,
and no such value is committed in a form usable against any deployed
environment.
*(§84.3; §65.3)*

## 4. Normative requirements — migrations

**R9.** Alembic scaffolding — configuration, environment and an empty
revision chain — exists at M00, because §81.1 gates every milestone
transition on repeatable migrations.
*(§81.1; §0E.1 Persistence row; §61; §84.1 — write migrations with
data-model changes)*

**R10.** M00 contains **no business schema migration**. The canonical data
model is M1 work, and the first real domain migration belongs there.
*(§81 milestone 1)*

**R11.** The migration command must execute successfully and repeatably
against the local PostgreSQL service on the empty chain, from a fresh
database and from an already-migrated one.
*(§81.1; §84.1 — never assume a fresh database is the only deployment
path)*

## 5. Normative requirements — continuous integration

**R12.** `knowhub-backend` CI must run to completion with only
`knowhub-backend` cloned. No job, step, script or cached artifact may
reference `knowhub-frontend` or require it to be present.
*(§71.3; §0E.8; ADR-016)*

**R13.** The following gates are **required and blocking** at M00:

| Gate | Requirement | Source |
|---|---|---|
| Lint and format | Ruff check and format verification | §0E.8 CI table; §0E.1 Python tooling row |
| Type check | Pyright, per [M00-SPEC-001](M00-SPEC-001-backend-repository-and-package-boundaries.md) R18–R19 | §0E.8 CI table |
| Package boundaries | Automated enforcement per [M00-SPEC-001](M00-SPEC-001-backend-repository-and-package-boundaries.md) R15 | §71; §0E.8 CI table |
| Unit tests | Pytest unit layer | §0E.8 CI table; §75.1 |
| Integration tests | Pytest integration layer, executed per R23 against the real services for which M00 has integration code | §0E.8 CI table; §75.1 |
| Migration validation | The R11 migration check | §0E.8 CI table; §81.1 |
| Dependency scan | Known-vulnerability scan of resolved dependencies | §0E.8 CI table |
| Secret scan | Detection of committed credentials and secrets | §84.3; §65.3 — see R15 |
| Container build | The R17 image builds successfully | §0E.8 published outputs |

**R14.** A gate that cannot yet find anything to check must still be wired
and blocking, so later milestones inherit a working gate rather than build
one.
*(§81.1 — preceding contracts remain stable)*

**R15.** Secret scanning is a required security gate: committed
credentials, tokens, keys or secrets must be detected and must fail CI. No
particular scanning product forms part of the architectural contract; the
implementation plan selects the tool.
*(§84.3 — no generated secrets, local tokens, source-system credentials or
model API keys may be committed; §65.3)*

**R16.** CI requirements are stated as outcomes and gates, not as
platform-specific pipeline syntax. **GitHub Actions is the implementation
baseline for the current KnowHub repositories**; §0E.1 permits GitHub
Actions or Azure DevOps Pipelines according to enterprise standard, and a
later move to Azure DevOps must not require amending this specification.
*(§0E.1 CI/CD row; §0E.8)*

## 6. Normative requirements — container image and process modes

**R17.** `knowhub-backend` produces **one** versioned Python runtime
image per commit, started in different modes for the API process, Taskiq
worker pools and the Taskiq scheduler.
*(§0E.7; §0E.8; §30.1; ADR-001 — no Rust/Tauri packaging model)*

**R18.** The image's entry point accepts a process-mode selection, so that
adding a mode later is a new command backed by real implementation rather
than a change to the image contract.
*(§30.1)*

**R19.** Only the **API** process mode has an implementation at M00. No
placeholder, stub or no-op worker or scheduler implementation may be
created to make the future mode list look complete. Taskiq worker and
scheduler implementations are M2 work.
*(§81 milestone 2; §84.1 — deliver working vertical slices, not a large set
of empty interfaces)*

**R20.** The running image serves the foundation liveness and readiness
endpoints specified in
[M00-SPEC-001](M00-SPEC-001-backend-repository-and-package-boundaries.md)
R8–R10, and emits telemetry per
[M00-SPEC-003](M00-SPEC-003-backend-observability-baseline.md).
*(§81.1 — telemetry exists)*

**R21.** No infrastructure-as-code or deployed-environment definition is
required at M00. `deploy/` exists as the location for backend runtime
deployment definitions per §84.3; its contents are later work.
*(§84.3; §0E.1 Infrastructure as code row)*

## 7. Normative requirements — tests

**R22.** The complete §75.1 test-layer model, as it applies to
`knowhub-backend` — unit, contract, integration, golden regression,
security and load/resilience — is the **target** structure for this
repository's test tree. §75.1's end-to-end browser journeys are a
`knowhub-frontend` responsibility under §0E.8 and are not a backend layer.
M00 materializes only the layers that contain real M00 tests, currently unit
and integration. No empty test-layer directory is created to mirror the
target model; a layer is created by the milestone that first supplies real
tests for it. This follows the same principle
[M00-SPEC-001](M00-SPEC-001-backend-repository-and-package-boundaries.md)
R4–R5 applies to Python packages: target architecture may be documented
without materializing empty structure.
*(§75.1; §84.1 — deliver working vertical slices, not a large set of empty
interfaces; §84.3; §61)*

**R23.** M00 integration tests use a real external service **only where
M00 contains real integration code for that dependency**. Where such code
exists, the test runs against the real R2 service rather than a mock of it,
in local development and in CI alike. Concretely at M00:

| Dependency | M00 integration code | M00 test obligation |
|---|---|---|
| PostgreSQL | Yes — the Alembic scaffolding of R9–R11 | Exercised by real integration tests against the local PostgreSQL service |
| Redis | No — deferred to M2 by R5 | Proven healthy and reachable per R1 and R24; no application integration test |
| Azurite | No — deferred to M2 by R6 | Proven healthy and reachable per R1 and R24; no application integration test |

Real Redis integration tests begin with the M2 functionality that uses
Redis. Real Blob/Azurite integration tests begin with the M2 functionality
that uses the Blob abstraction.
*(§75.1 — integration covers Postgres/Redis/Blob, as capabilities arrive;
§0E.8; §81 milestone 2; §84.1)*

**R24.** Availability of a deferred-usage dependency is verified through
its R1 health check in the Local and CI environments, not through
application code. Health-check verification is not an integration test and
must not be described or counted as one.
*(§0E.8; §76 Local column; §84.1)*

**R25.** Failure paths are exercised, not only success paths: §81.1 gates
the milestone on it. At M00 this means at minimum the configuration,
boundary, readiness and migration failure behaviours named in the
acceptance criteria of this packet.
*(§81.1; §84.2)*

## 8. Forward constraints recorded, not implemented at M00

- **OpenAPI contract artifact.** §0E.8 requires backend CI to publish an
  immutable versioned `openapi.json` and to run an OpenAPI compatibility
  diff. M00 introduces no business API surface, so there is no contract to
  publish and no baseline to diff against. Both become required with the
  first business endpoint.
  *(§0E.8;
  [ADR-017](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md))*
- **Cross-repository compatibility attestation.** §0E.8's frontend-version
  ↔ backend-version pairing requires both repositories to produce release
  candidates; it activates at milestone 9.
- **Golden, security, evaluation and parity gates.** §0E.8 lists these
  among backend CI gates. They activate with the capabilities they measure.
- **SBOM and container scanning depth.** §0E.8 requires an SBOM where
  required; the requirement is environment- and policy-driven and is
  settled with deployed-environment work.

## 9. Acceptance criteria

| ID | Criterion |
|---|---|
| **M00-AC-018** | The documented bring-up sequence starts PostgreSQL, Redis and Azurite on a clean machine and each reports healthy through its declared health check; no service outside that set is required. *(R1, R2, R7)* |
| **M00-AC-019** | The pgvector extension can be enabled on the local PostgreSQL service, and the PostgreSQL version and image are explicitly pinned. *(R3, R4)* |
| **M00-AC-020** | The migration command succeeds against the local PostgreSQL on the empty chain, and is repeatable both from a fresh database and from an already-migrated one. No business schema migration exists. *(R9, R10, R11)* |
| **M00-AC-021** | CI completes with only `knowhub-backend` cloned; no job, step, script or cached artifact references `knowhub-frontend`. *(R12)* |
| **M00-AC-022** | Every gate in R13 is present and blocking, including gates that currently find nothing to check. *(R13, R14)* |
| **M00-AC-023** | The secret-scanning gate is demonstrated against a synthetic, known-test credential placed into an ephemeral verification input — a scratch worktree, fixture or scan target created for the check and discarded after it. The scan fails on that input and passes once it is removed. No real or persistent credential is committed, and no detectable test credential remains in the repository after verification. *(R15)* |
| **M00-AC-024** | M00 integration tests run in CI against the real service for every dependency where M00 has integration code — at M00, PostgreSQL via the Alembic scaffolding — and not against mocks of it. Redis and Azurite are verified available through their health checks only; no Redis client, Blob client, fake adapter or placeholder application integration exists in the repository to make them testable. *(R5, R6, R23, R24)* |
| **M00-AC-025** | The image builds, starts in API mode, serves the foundation liveness and readiness endpoints and emits telemetry; the entry point accepts a process-mode selection; no placeholder worker or scheduler implementation exists in the repository. *(R17, R18, R19, R20)* |
| **M00-AC-026** | The test tree contains only layers holding real M00 tests — currently unit and integration; no empty test-layer directory exists for a layer with no tests yet, and the full §75.1 model is recorded as the target rather than materialized. The M00 failure paths named in R25 are exercised. *(R22, R25)* |

## 10. Deferred

| Deferred from M00 | Owning milestone |
|---|---|
| First business schema migration and the canonical data model | M1 |
| OpenAPI artifact publication and the compatibility-diff gate | M1 / 0B |
| Taskiq broker, worker and scheduler implementations and their process modes | M2 |
| Blob abstraction and adapter, and the first real Blob/Azurite integration tests — Azurite is a health-checked local service only at M00 | M2 |
| Redis usage in application code — broker, cache, locks, throttling — and the first real Redis integration tests | M2 |
| Contract, parity, golden and security test layers, their directories and their CI gates — each created by the milestone that first supplies real tests for it | M3+ |
| Deterministic parser test gate | M4 |
| Load and resilience testing | Post-0B |
| `deploy/` contents, Terraform/Bicep, Azure Container Apps or AKS topology | Post-0B |
| DEV, SIT/UAT and PROD pipelines and §76.1 promotion rules | Post-0B |
| Cross-repository compatibility attestation | M9 |
| SBOM generation where enterprise policy requires it | Deployed-environment work |
