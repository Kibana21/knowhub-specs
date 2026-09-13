# M00-SPEC-001 — Backend repository and package boundaries

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Repository:** `knowhub-backend`
- **Status:** Approved
- **§81 0A items owned:** `pyproject/uv`, `Python package boundaries`

## 1. Purpose

Define the shape of the `knowhub-backend` repository at M00: how
dependencies are managed, which Python packages exist, what each may and
may not import, how those boundaries are enforced, which quality tooling
gates the code, and the minimal bootable application that proves the
foundation runs.

This specification establishes structure. It delivers no business
capability.

## 2. Sources

**Blueprint**

- [§30 Backend codebase — knowhub-backend](../../../knowhub_master_blueprint.html#target-python),
  including §30.1 (runtime processes from one backend codebase) and
  §30.2 (backend owns all security and knowledge truth)
- [§61 Backend Python file-level target specification](../../../knowhub_master_blueprint.html#python-file-spec),
  including §61.1 (hard boundaries between packages), §61.2 (services that
  must stay deterministic) and §61.3 (services allowed to infer/synthesize
  with an LLM)
- [§71 Backend package boundaries & cross-repository dependency rules](../../../knowhub_master_blueprint.html#package-dependency-rules),
  §71.1–§71.4
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Primary language, Backend API, Python tooling and Packaging rows
- [§0E.8 Two-repository delivery model](../../../knowhub_master_blueprint.html#runtime-architecture)
  — **normative**
- [§84.3 Repository hygiene expected from the harness](../../../knowhub_master_blueprint.html#coding-harness-instructions)
- [§84.1 Mandatory working style](../../../knowhub_master_blueprint.html#coding-harness-instructions)
  — vertical slices over empty interfaces; typed contracts

**ADRs** — referenced by number and link only; their decision text lives in
the canonical files.

- [ADR-001 — KnowHub is Python-first and API-first](../../02-adrs/ADR-001-python-first-api-first.md)
- [ADR-016 — Two independently versioned repositories](../../02-adrs/ADR-016-two-independently-versioned-repositories.md)
- [ADR-026 — ADR ownership, location, numbering and register](../../02-adrs/ADR-026-adr-ownership-and-location.md)
- [ADR-017 — OpenAPI is the shared frontend/backend contract](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md)
  (forward constraint; see §6)
- Backend-scoped boundary constraints recorded in §4.4, canonical files in
  `knowhub-backend/docs/adr/` per the
  [ADR register](../../02-adrs/README.md).

## 3. Normative requirements — dependency management and language baseline

**R1.** Backend application and runtime source is Python only. No
React/Next.js source, and no TypeScript application source, exists in
`knowhub-backend`.
*(§30; §0E.8; §84.3; ADR-016)*

**R2.** Dependencies are declared in `pyproject.toml` and resolved with
`uv`. A committed `uv.lock` is the single reproducible resolution for the
repository. No second dependency path (ad-hoc `pip install`, requirements
file, vendored tree) may be required to obtain a working environment.
*(§0E.1 Python tooling row; §30; §61)*

**R3.** The M00 supported runtime baseline is **Python 3.12**.
`.python-version` identifies 3.12 and CI executes on 3.12. The declared
`requires-python` constraint preserves the blueprint's stated Python 3.12+
intent, but no compatibility beyond 3.12 is claimed at M00 because none is
tested.
*(§0E.1 Primary language row)*

*Implementation guidance (non-normative): exact patch level and runtime
image digest pinning belong to the implementation plan and lockfiles, not
to this specification. No blueprint section or ADR requires a specific
patch level.*

## 4. Normative requirements — package structure and boundaries

### 4.1 Target layout versus M00 materialization

**R4.** The package layout in
[§61](../../../knowhub_master_blueprint.html#python-file-spec) is the
**target architecture** for `knowhub-backend`. It is not the M00
deliverable.

**R5.** M00 materializes only packages that carry real M00
responsibility. Empty packages must not be created to mirror the future
tree. In particular, M00 creates no package for retrieval, Knowledge
Foundry, connectors, ingestion, parsing, graph, documents, source, context,
inquiry, traceability, lifecycle, workflows, agents, skills, llm,
ai_programs, identity, authz, access_admin, governance, evaluation,
persistence, domain, services or workers.
*(§84.1 — deliver working vertical slices, not a large set of empty
interfaces)*

### 4.2 M00 package inventory

**R6.** The M00 package inventory under `src/knowhub/` is exactly:

| Package | M00 responsibility | Source |
|---|---|---|
| `knowhub.api` | FastAPI application bootstrap and the foundation liveness/readiness endpoints. Nothing resembling business functionality. | §30; §61; §0E.1 Backend API row |
| `knowhub.config` | Typed settings, feature flags and policy configuration. Contract specified in [M00-SPEC-002](M00-SPEC-002-backend-configuration-and-settings.md). | §61; §77 |
| `knowhub.observability` | OpenTelemetry bootstrap and the correlation-identifier contract. Contract specified in [M00-SPEC-003](M00-SPEC-003-backend-observability-baseline.md). | §61; §36 |

**R7.** A package is added to `src/knowhub/` only when a milestone
delivers code that belongs in it. Adding a package is a structural change
justified by that milestone's specification, not by the §61 target diagram.

### 4.3 The minimal bootable application

**R8.** `knowhub.api` provides application bootstrap and exactly two
endpoints at M00:

- a liveness endpoint (`/healthz`) reporting that the process is running
  and able to serve;
- a readiness endpoint (`/readyz`) reporting whether the process has
  completed startup and its **M00-scoped** prerequisites are satisfied.

*(§30; §61; §0E.1 Backend API row; §81 milestone 0A — the foundation must be
demonstrably bootable)*

**R9.** Readiness at M00 evaluates only foundation concerns — successful
typed-settings load and completed observability initialisation. It must not
introduce a dependency on any capability owned by a later milestone.
Specifically, readiness must not probe the canonical database schema,
identity, the run model, Taskiq/Redis, blob storage or any connector.
*(§81 milestone sequence; §84.1 — do not broaden a milestone)*

**R10.** Neither endpoint exposes secrets, credentials, connection
strings, internal host addresses or configuration values beyond a coarse
status. Failure detail is safe detail.
*(§65.3; §84.3)*

**R11.** No other route, router or API surface is introduced at M00.
*(§81 — API surface belongs to M1 and later)*

### 4.4 Package dependency boundaries

**R12.** The dependency direction and the rule set of
[§71.1 and §71.2](../../../knowhub_master_blueprint.html#package-dependency-rules)
are normative for `knowhub-backend` from M00 onward, and are recorded here
so that enforcement exists before the packages they govern do.

**R13.** The forbidden dependencies enumerated in
[§71.4](../../../knowhub_master_blueprint.html#package-dependency-rules)
are normative. They include, without restating the section: knowledge
synthesis code must not import FastAPI, source-system SDKs or a concrete
model-provider client; connectors must not import UI code; API routes must
not invoke a model directly; neither application repository may import the
other's source packages; and the task result backend must not hold
canonical run or business state.
*(§71.4; ADR-018; ADR-007; ADR-008; ADR-012; ADR-020 — canonical decision
text in the [ADR register](../../02-adrs/README.md))*

**R14.** The determinism split of
[§61.2 and §61.3](../../../knowhub_master_blueprint.html#python-file-spec)
is a package boundary, not a coding convention: services listed as
deterministic must not call a model.
*(§61.2; §61.3; §0G.3;
[ADR-013](../../02-adrs/ADR-013-deterministic-extraction-before-llm-inference.md))*

**R15.** Violation of a boundary in R12–R14 must **fail CI
automatically**. Automated enforcement is the normative requirement; no
specific enforcement product is part of the architectural contract.

*Implementation guidance (non-normative): `import-linter` is the
recommended initial mechanism, with a contract file expressing the §71.1
layering and the §71.4 prohibitions. Ruff's banned-import rules can cover a
subset. The tool choice, contract file and command wiring belong to the
implementation plan and may change without amending this specification, so
long as R15 holds.*

**R16.** Because M00 materializes three packages, boundary enforcement at
M00 is necessarily partial in coverage. The enforcement mechanism must
nonetheless be wired, gating and demonstrably able to fail on a violation,
so that later milestones inherit a working gate rather than build one.
*(§81.1 — preceding contracts remain stable)*

## 5. Normative requirements — quality tooling and repository hygiene

**R17.** Lint and format enforcement uses **Ruff**, and violations fail CI.
*(§0E.1 Python tooling row)*

**R18.** Static type checking uses **Pyright**, and type errors in
KnowHub-owned application code fail CI.
*(§0E.1 Python tooling row, which permits Pyright or mypy; §84.1 — use
typed contracts)*

**R19.** The required type-checking outcome is that KnowHub-owned
application code is strongly typed at its own boundaries: public functions,
service entry points, settings models and returned contracts carry explicit
types. Strictness must not be configured so as to force semantically empty
annotations on third-party framework integration surfaces where the
framework does not supply usable types. Suppressions are narrow, local and
justified; blanket suppression of a whole module or rule class is not an
acceptable way to pass the gate.
*(§84.1)*

**R20.** The repository contains `docs/adr/` for backend-scoped ADRs and
`docs/api/` for OpenAPI guidance and changelogs.
*(§84.3;
[ADR-026](../../02-adrs/ADR-026-adr-ownership-and-location.md))*

**R21.** No generated secret, local token, source-system credential,
production data snapshot or model API key is committed. This is enforced as
a CI gate specified in
[M00-SPEC-004](M00-SPEC-004-backend-build-ci-and-local-dependencies.md).
*(§84.3; §65.3)*

**R22.** `knowhub-backend` must lint, type-check, test and build without
cloning `knowhub-frontend`.
*(§71.3; §0E.8;
[ADR-016](../../02-adrs/ADR-016-two-independently-versioned-repositories.md))*

## 6. Forward constraints recorded, not implemented at M00

- **OpenAPI contract authority.** The backend owns the HTTP contract and
  exports a versioned `openapi.json`; the frontend generates from a pinned
  artifact. M00 introduces no business API surface, so no contract artifact
  is published and no compatibility-diff gate is meaningful yet.
  *(§0E.8;
  [ADR-017](../../02-adrs/ADR-017-openapi-is-the-shared-contract.md))*
- **One image, several process modes.** §30.1's process model is preserved
  by the image structure specified in
  [M00-SPEC-004](M00-SPEC-004-backend-build-ci-and-local-dependencies.md).
  Only the API mode has real implementation at M00.
  *(§30.1; §0E.7)*
- **Backend owns security and knowledge truth.** §30.2 binds every later
  milestone. M00 adds nothing that could relocate such a decision.

## 7. Acceptance criteria

| ID | Criterion |
|---|---|
| **M00-AC-001** | A clean checkout reaches a working environment through the documented `uv` workflow alone, using the committed `uv.lock`. No alternative dependency path is required. *(R2)* |
| **M00-AC-002** | `.python-version` identifies 3.12, the declared `requires-python` preserves the 3.12+ intent, and CI executes on 3.12. *(R3)* |
| **M00-AC-003** | `src/knowhub/` contains exactly the packages in R6. No package exists for a capability owned by a later milestone. *(R5, R6)* |
| **M00-AC-004** | A deliberately introduced import that violates R12–R14 causes the CI boundary gate to fail; removing it restores a green run. *(R15, R16)* |
| **M00-AC-005** | A lint or formatting violation fails CI. *(R17)* |
| **M00-AC-006** | A type error in KnowHub-owned application code fails CI under the agreed Pyright configuration, and the configuration does not require empty annotations on untyped framework surfaces. *(R18, R19)* |
| **M00-AC-007** | `docs/adr/` and `docs/api/` exist; the repository contains no frontend application source and no committed secret, credential, token or data snapshot. *(R1, R20, R21)* |
| **M00-AC-008** | `/healthz` and `/readyz` respond as specified; readiness reflects only settings load and observability initialisation, references no later-milestone dependency, and neither endpoint discloses configuration values or secrets. *(R8, R9, R10)* |

## 8. Deferred

| Deferred from M00 | Owning milestone |
|---|---|
| `domain/`, `persistence/`, `services/` packages; canonical data model; SQLAlchemy mappings and repositories | M1 |
| `identity/`, `authz/`, `access_admin/` packages and all authentication/authorization logic | M1 |
| Business API routers, request/response schemas, the published versioned OpenAPI artifact and its compatibility gate | M1 / 0B |
| `workers/` package, Taskiq broker, tasks and scheduler | M2 |
| `connectors/`, `ingestion/`, `documents/`, `source/` | M3 / M10 |
| `parsing/` and the §61.2 deterministic extraction services | M4 |
| `graph/` | M5 |
| `knowledge_foundry/`, `context/`, `retrieval/`, `inquiry/` | M6–M8 |
| `llm/`, `ai_programs/` and the Model Gateway | M6+ |
| `governance/`, `evaluation/`, `traceability/`, `lifecycle/`, `workflows/` | M7+ |
| Full-coverage boundary contracts for packages that do not yet exist | Each owning milestone |
