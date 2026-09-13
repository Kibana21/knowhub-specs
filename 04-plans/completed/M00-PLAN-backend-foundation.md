# M00 implementation plan — Backend foundation

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Repository:** `knowhub-backend`
- **Status:** Approved
- **Specifications:** [M00-SPEC-001](../../03-specs/00-foundation/M00-SPEC-001-backend-repository-and-package-boundaries.md),
  [M00-SPEC-002](../../03-specs/00-foundation/M00-SPEC-002-backend-configuration-and-settings.md),
  [M00-SPEC-003](../../03-specs/00-foundation/M00-SPEC-003-backend-observability-baseline.md),
  [M00-SPEC-004](../../03-specs/00-foundation/M00-SPEC-004-backend-build-ci-and-local-dependencies.md)
- **Packet index:** [`00-foundation/README.md`](../../03-specs/00-foundation/README.md)

## 1. Purpose and boundaries

This plan says **how** M00 is built: the tools, versions, file changes and
task order that realise the four approved specifications. It introduces no
requirement. Where this plan and a specification disagree, the
specification wins and this plan is wrong.

Every choice below is **replaceable implementation detail** unless marked
otherwise. Each was tested against the architectural bar — new
infrastructure, framework, datastore, protocol, package boundary or platform
invariant. **None crosses it, so this plan proposes no ADR.**

Reference repositories were not consulted. No M00 requirement was
unresolvable from the specifications and ADRs.

## 2. Applicable ADRs

Constraining M00 directly: ADR-001, ADR-003, ADR-004, ADR-006, ADR-015,
ADR-016, ADR-026. Recorded as boundary constraints without implementation:
ADR-007, ADR-008, ADR-011, ADR-012, ADR-013, ADR-014, ADR-017, ADR-018,
ADR-019, ADR-020. Canonical locations are in the
[ADR register](../../02-adrs/README.md); decision text is not reproduced
here.

## 3. Decisions the specifications delegated to this plan

### 3.1 Language and runtime

| Item | Decision | Rationale | Class |
|---|---|---|---|
| Supported runtime | Python 3.12 — the only tested M00 runtime | SPEC-001 R3 | Fixed by spec |
| `.python-version` | Exact `3.12.x` patch, selected as the latest 3.12 security patch at implementation time | Reproducibility needs an exact pin; the patch number is chosen against the then-current release, not guessed in advance | Replaceable |
| `requires-python` | `">=3.12"` | Absence of 3.13 testing is not a contractual ceiling. 3.12 remains the only tested runtime via `.python-version`, CI and the image | Replaceable |
| Runtime image | Two-stage: `ghcr.io/astral-sh/uv:python3.12-bookworm-slim` builder → `python:3.12-slim-bookworm` runtime, both pinned by tag **and** SHA256 digest | uv builder gives fast reproducible installs; slim runtime keeps the image small; digest pinning is the only real reproducibility | Replaceable |

### 3.2 Dependency services

Provisioning is neutral (SPEC-004 R1, R1.2). The values below define the
**R1.1 reproducible container-based environment**; an instance supplied
outside it must satisfy the stated minimum.

| Service | R1.1 image | Published port | Minimum for an externally supplied instance |
|---|---|---|---|
| PostgreSQL | `pgvector/pgvector:pg15`, pinned by tag + SHA256 digest | 55432 | **PostgreSQL 15 or later, with the `vector` extension available** |
| Redis | `redis:7.4-alpine`, pinned by digest | 56379 | Any reachable Redis answering `PING` with `PONG` |
| Azurite | `mcr.microsoft.com/azure-storage/azurite:3.x`, pinned by digest | 10000 / 10001 / 10002 | Any reachable Azurite blob endpoint |

**PostgreSQL 15 is both the minimum and the clean-room major, deliberately.**
CI must run the minimum supported version: if the clean-room ran a newer
major than the stated floor, a version-specific behaviour could pass CI and
break every developer on the minimum, and the gate would be blind to exactly
the failure it exists to catch. Forward-compatibility coverage, when wanted,
is an **additional** CI matrix entry, not a raised baseline.

Nothing in M00 or the approved specifications needs a PostgreSQL 16+
feature: there is no schema, no query and no extension usage.

PostgreSQL 15 is a GA major on Azure Database for PostgreSQL Flexible
Server with pgvector in its allowlisted extension set, satisfying SPEC-004
R3. Two items to confirm rather than assume: available majors should be
checked against the actual subscription and platform policy before M1
provisioning, and community PostgreSQL 15 reaches end of life in **November
2027**, the natural horizon for revisiting this. Raising the minimum later
is a change to this plan, not to a specification — SPEC-004 R4 delegates the
choice here precisely so it stays cheap.

Compose publishes on deliberately non-default ports so the R1.1 environment
can never collide with a service the developer already runs on 5432 or 6379.

**pgvector at M00:** availability only. SPEC-004 R3 and M00-AC-019 require
that the extension *can be* enabled. `CREATE EXTENSION vector` is not
executed at M00 — no M00 code or migration uses it. It belongs to the
milestone that first needs it.

### 3.3 Availability verification (SPEC-004 R1.3)

| Service | Inside R1.1 | Supplied externally |
|---|---|---|
| PostgreSQL | Compose `pg_isready` health check | `scripts/check_dependencies.py`: `psycopg` connect, assert `server_version_num` ≥ 150000, assert `vector` present in `pg_available_extensions` |
| Redis | Compose health check running **`redis-cli PING`** inside the container | Documented primary: `redis-cli -h <host> -p <port> PING`. Script equivalent: raw socket, Redis inline command `PING\r\n`, assert reply `+PONG` |
| Azurite | Compose HTTP health check on the blob endpoint | `scripts/check_dependencies.py`: plain HTTP request via `httpx` |

**Constraints on the Redis check**, which this plan treats as binding:
it lives in `scripts/`, never under `src/knowhub/`; it adds no Redis Python
dependency; it is used only for availability verification; it must not grow
into a reusable Redis client; it performs only the minimum PING/PONG
exchange. A `-NOAUTH` reply is reported clearly as *"Redis reachable,
credentials required"* — never treated as a healthy unauthenticated
application connection. No `AUTH` is implemented at M00.

`scripts/` is a §61-sanctioned location and is not application code, so this
satisfies SPEC-004 R24's requirement that availability is verified outside
application code, and R5/R6's prohibition on a client, fake adapter or
placeholder integration.

### 3.4 Quality tooling

| Item | Decision | Rationale | Class |
|---|---|---|---|
| Lint and format | **Ruff**. `target-version = "py312"`, line length 88, formatter enabled. Rules `E,W,F,I,N,UP,B,A,C4,SIM,RUF,PT,S,TID,ANN`. Per-file ignores: tests (`S101`, relaxed `ANN`), `migrations/versions/*` relaxed | One tool for lint, format, import order, annotation presence and basic security lint | Tool fixed by SPEC-001 R17; configuration replaceable |
| Type checking | **Pyright**, `typeCheckingMode = "standard"`, `pythonVersion = "3.12"`, `include = ["src", "tests"]`. Enable `reportMissingTypeStubs`, `reportUntypedFunctionDecorator`, `reportIncompatibleMethodOverride`, `reportUnnecessaryTypeIgnoreComment`. Leave `reportUnknownMemberType` and `reportUnknownArgumentType` **off** | `strict` fires the unknown-type family across untyped framework surfaces — exactly the noise SPEC-001 R19 forbids forcing. "KnowHub code is annotated" is enforced by Ruff `ANN` on `src/`; Pyright enforces correctness. Each half of R19 lands on the tool that does it well | Tool fixed by SPEC-001 R18; level replaceable |
| Import boundaries | **`import-linter`**, contracts in `[tool.importlinter]` in `pyproject.toml` | Declarative layering and forbidden-import contracts | Replaceable — SPEC-001 R15 binds automated enforcement, not a product |
| Secret scanning | **`gitleaks`** | Broad ruleset, scans history as well as the tree, single binary, no Python dependency | Replaceable — SPEC-004 R15 binds the outcome |
| Dependency scanning | **`pip-audit`** against `uv export --format requirements-txt` | PyPA-maintained, OSV/PyPI advisory sources, no coupling to lockfile format | Replaceable |

**Boundary contracts at M00** — layered, top to bottom: `knowhub.api` →
`knowhub.observability` → `knowhub.config`. Forbidden: `knowhub.config` →
`fastapi`; `knowhub.config` → `opentelemetry`; `knowhub.observability` →
`fastapi`.

The last contract is load-bearing: it keeps the correlation context
framework-free so a future worker process can reuse it, and forces the
ASGI correlation middleware to live in `knowhub.api`. Contracts may
reference only modules that exist — `import-linter` errors on a contract
naming an absent module — which is the partial coverage SPEC-001 R16
anticipates.

### 3.5 Configuration

| Item | Decision | Rationale | Class |
|---|---|---|---|
| Library | **`pydantic-settings` v2**. Root `Settings`, `env_prefix="KNOWHUB_"`, `env_nested_delimiter="__"`, `extra="forbid"`, nested domain models | `extra="forbid"` combined with a prefix rejects unknown `KNOWHUB_*` keys while ignoring everything outside the namespace — SPEC-002 R2.1 and R2.2 in one mechanism | Replaceable |
| Secret references | KnowHub `SecretRef` carrying `scheme:identifier` (`env:NAME` at M00), a `SecretResolver` protocol, and an env-backed local resolver. Resolved values wrapped in `pydantic.SecretStr` | SPEC-002 R15/R16 require a *reference*, not a redacted value — `SecretStr` alone would store the secret in settings. A raw value fails the grammar at load. A Key Vault resolver later is a resolver change, not a settings-model change | Replaceable |
| Database settings | Discrete `DATABASE__HOST`, `__PORT`, `__NAME`, `__USER`, `__PASSWORD_REF`, composed into a DSN internally | No credential-bearing connection string as a single setting | Replaceable |
| Modules created | `settings.py`, `secrets.py`, `feature_flags.py` | SPEC-002 R8 and R10 give both real M00 content | — |
| `policies.py` | **Deferred.** Not created at M00 | M00 has no policy content (SPEC-002 R9). SPEC-001 R5 and the no-empty-structure rule forbid a module with nothing in it. R1 describes the package's shape as it fills out; the module arrives with the milestone that has real policy content | — |

### 3.6 Observability

| Item | Decision | Rationale | Class |
|---|---|---|---|
| Packages | `opentelemetry-api`, `opentelemetry-sdk`, `opentelemetry-exporter-otlp-proto-http`, `opentelemetry-instrumentation-fastapi`. Console exporter ships in the SDK | Vendor-neutral. OTLP **HTTP/protobuf** over gRPC: fewer native dependencies, simpler in containers. **No Azure telemetry SDK** (SPEC-003 R6) | Replaceable |
| Exporter selection | Platform-domain setting, values `otlp` / `console` / `none`, with an endpoint setting for OTLP | Destination changes by configuration alone (SPEC-003 R7) | Replaceable |
| Providers | Tracer provider and meter provider initialised at bootstrap. **No custom metric defined at M00** | SPEC-003 R1 names metrics; the §36 catalogue belongs to the milestones that deliver what it measures | Replaceable |
| Logging | **Standard library `logging`** — a `logging.Filter` injecting the correlation ID and the current OTel trace/span IDs, plus a JSON `Formatter`, configured through `dictConfig`. **No logging framework dependency** | A root-handler filter covers third-party library logs uniformly, which a processor-pipeline library only achieves through a stdlib bridge that is a known source of inconsistent rendering. A JSON formatter is ~30 lines; the alternative is another dependency, so nothing is saved. Configuration stays isolated in one module, so adopting a framework later is a single-file change | Replaceable |
| Correlation ID | **UUIDv4**, canonical 36-character lowercase hyphenated form. Header **`X-Correlation-ID`**, inbound and on the response | Satisfies every clause of SPEC-003 R9: documented representation, bounded length, control characters impossible | Replaceable |
| Validation order | Length ≤ 36 → UUID pattern → on any failure, discard and generate | Cheap rejection first; caller input never reaches telemetry unvalidated | Replaceable |
| Trace context | W3C `traceparent` handled by the OTel propagator, kept distinct from the KnowHub correlation identifier | They answer different questions and must not be conflated | Replaceable |
| Naming convention | Documented in `docs/telemetry-naming.md`: OTel semantic conventions where one applies, a `knowhub.*` namespace where none does | SPEC-003 R13 requires a single documented convention | Replaceable |

### 3.7 Build, CI and packaging

| Item | Decision | Rationale | Class |
|---|---|---|---|
| CI platform | **GitHub Actions**, one `.github/workflows/ci.yml`. Jobs: `quality` → `unit` → `integration` → `security` → `image`. `astral-sh/setup-uv` with cache, then `uv lock --check` followed by `uv sync --frozen` (`--frozen` alone does not detect a stale lock), actions pinned by commit SHA, `permissions: contents: read`, concurrency group per ref | SPEC-004 R16 states the baseline. Jobs are named by outcome, so an Azure DevOps port is a translation rather than a redesign | Baseline fixed by SPEC-004 R16; structure replaceable |
| CI dependency services | **Always the R1.1 Compose environment**, on an ephemeral runner | One definition for local and CI removes drift. **No CI job, step, script or cached artifact may reference or reach a workstation-local service** (SPEC-004 R12, R1.1) | Replaceable |
| Container entrypoint | `docker-entrypoint.sh` dispatching on `$1`: `api` → uvicorn. Any other value exits non-zero with `unknown process mode: <x>`. **No `worker` or `scheduler` branch exists** | SPEC-004 R18 needs the mechanism; R19 forbids placeholder modes. A dispatcher accepting one mode and rejecting the rest satisfies both. A Python CLI package for a single command would be structure M00 does not need | Replaceable |
| Test layout | `tests/conftest.py`, `tests/unit/`, `tests/integration/` — nothing else. pytest config in `pyproject.toml`: `testpaths`, `--strict-markers`, an `integration` marker | SPEC-004 R22: only layers holding real M00 tests | Replaceable |
| Alembic | `alembic.ini` at repository root; `migrations/` with `env.py`, `script.py.mako`, `versions/.gitkeep`. `target_metadata = None`. `env.py` obtains the database URL from `knowhub.config`, resolving the password through the `SecretRef` resolver | One source of truth for the URL, and it exercises the secret machinery for real rather than in a test-only path. Autogenerate is unusable until M1 adds models — correct, not a gap | Replaceable |

### 3.8 `.env.example`

Documents **both** supported developer paths, with every value presented as
a configuration example rather than an architectural fact, per SPEC-004 R1
and R1.2:

- **Mode A — existing services.** Point KnowHub at instances already
  running on the machine. Example values may show conventional localhost
  endpoints.
- **Mode B — managed Compose.** The R1.1 environment on its non-default
  published ports.

No workstation-specific value is canonical. No personal credential,
machine-specific secret or deployed-environment endpoint is committed
(SPEC-002 R18, SPEC-004 R8). Secrets appear only as `SecretRef`
placeholders.

### 3.9 Dependency footprint

Runtime: `fastapi`, `uvicorn[standard]`, `pydantic`, `pydantic-settings`,
`opentelemetry-api`, `opentelemetry-sdk`,
`opentelemetry-exporter-otlp-proto-http`,
`opentelemetry-instrumentation-fastapi`, `alembic` (pulls SQLAlchemy 2),
`psycopg[binary]`.

Development: `ruff`, `pyright`, `import-linter`, `pytest`, `pytest-asyncio`,
`httpx`, `pip-audit`.

**Absent by design:** no logging framework, no Redis client, no Azure
Storage SDK, no Azure telemetry SDK, no task queue, no LangGraph, no DSPy,
no model provider SDK.

## 4. Expected repository state after M00

```
knowhub-backend/
├── .github/workflows/ci.yml          NEW
├── .dockerignore                     NEW
├── .env.example                      NEW
├── .gitignore                        NEW
├── .python-version                   NEW
├── alembic.ini                       NEW
├── docker-compose.dev.yml            NEW
├── docker-entrypoint.sh              NEW
├── Dockerfile                        NEW
├── pyproject.toml                    NEW
├── uv.lock                           NEW
├── README.md                         MODIFIED
├── docs/
│   ├── adr/                          UNCHANGED (11 existing ADRs)
│   ├── api/README.md                 NEW
│   └── telemetry-naming.md           NEW
├── migrations/
│   ├── env.py                        NEW
│   ├── script.py.mako                NEW
│   └── versions/.gitkeep             NEW
├── scripts/
│   └── check_dependencies.py         NEW
├── src/knowhub/
│   ├── __init__.py
│   ├── api/{__init__,main,health,middleware}.py
│   ├── config/{__init__,settings,secrets,feature_flags}.py
│   └── observability/{__init__,bootstrap,correlation,logging}.py
└── tests/
    ├── conftest.py
    ├── unit/
    └── integration/{conftest.py,test_migrations.py}
```

**Deliberately not created:** `knowhub.cli`, `knowhub.workers`,
`knowhub.persistence`, `knowhub.domain`, `knowhub.services` or any other §61
package; `config/policies.py`; `api/routers/` (arrives with the first
business router in M1); `deploy/` contents; `tests/contract|golden|parity|security|load/`;
any Redis or Blob client; any worker or scheduler entrypoint; any revision
file under `migrations/versions/`.

`api/middleware.py` is a single module rather than a package, and health
probes live at `api/health.py` rather than `api/routers/health.py`: probes
are not business routers, and creating `routers/` for them would
pre-materialise M1 structure.

## 5. Tasks

| # | Task | Depends on |
|---|---|---|
| T1 | Repository skeleton and dependency baseline | — |
| T2a | Managed Compose dependency environment | T1 |
| T3 | Configuration package | T1 |
| T2b | Configured-endpoint dependency verification | T2a, T3 |
| T4 | Observability package | T1, T3 |
| T5 | Minimal API foundation | T3, T4 |
| T6 | Boundary enforcement | T3, T4, T5 |
| T7 | Alembic scaffolding | T2a, T3 |
| T8 | Test harness and failure-path tests | T2b, T3, T4, T5, T7 |
| T9 | Container image and process mode | T5 |
| T10 | CI wiring | T1–T9 |
| T11 | M00 verification pass | T10 |

**Sequencing rule — one configuration mechanism.** `knowhub.config` is the
single source of configuration semantics. Dependency work is therefore split
so that nothing reads configured endpoints before that mechanism exists:

- **T2a** may run immediately after T1. It establishes the R1.1 Compose
  environment and its **service-native** health checks — `pg_isready`,
  `redis-cli PING` and an HTTP probe, each executing inside or against its
  own container. These read Compose's own values and no KnowHub setting, so
  they introduce no configuration semantics.
- **T3** establishes the canonical settings model.
- **T2b** runs only after T3. Any verification that reads a **configured
  endpoint** uses `knowhub.config` to obtain it.

`scripts/check_dependencies.py` must not reinterpret `KNOWHUB_*` variables
with its own parser, its own precedence, its own defaults or its own
validation. It imports the canonical settings model and consumes resolved
values. Duplicating settings semantics in a script is how the two drift, and
the script would then verify endpoints the application never uses.

### T1 — Repository skeleton and dependency baseline
- **Files:** `pyproject.toml`, `uv.lock`, `.python-version`, `.gitignore`, `.dockerignore`, `docs/api/README.md`, `README.md` (modified), `src/knowhub/__init__.py`
- **Requirements:** SPEC-001 R1, R2, R3, R17, R18, R19, R20, R22
- **Acceptance:** M00-AC-001, M00-AC-002, M00-AC-007 (in part)
- **Tests:** none — a clean-clone environment build is the evidence
- **Failure paths:** `uv sync --locked` fails on a lockfile stale against the manifest (`--frozen` does not assert freshness — factual correction found during T01 implementation)
- **Telemetry:** none

### T2a — Managed Compose dependency environment
- **Files:** `docker-compose.dev.yml`
- **Requirements:** SPEC-004 R1.1, R2, R4 (image pinning), R7, R8
- **Acceptance:** M00-AC-018 (R1.1 clause), M00-AC-019 (pinning clause)
- **Tests:** none — service-native health status is the evidence, per R24
- **Failure paths:** a service that never reports healthy fails bring-up visibly and by name
- **Telemetry:** none
- **Constraint:** health checks are service-native and read no KnowHub setting. No configuration parsing enters this task

### T2b — Configured-endpoint dependency verification
- **Files:** `scripts/check_dependencies.py`, `.env.example` (both modes), `README.md` (dependency section)
- **Requirements:** SPEC-004 R1, R1.2, R1.3, R3, R4 (minimum version), R5 and R6 (availability clauses), R24
- **Acceptance:** M00-AC-018 (externally supplied clause), M00-AC-019 (configured instance clause)
- **Tests:** none — availability observation is the evidence, per R24
- **Failure paths:** the script exits non-zero with a specific reason per service; a PostgreSQL below the minimum or lacking `vector` fails with a named reason; a Redis `-NOAUTH` reply is reported as credentials-required, not as healthy
- **Telemetry:** none
- **Constraint:** imports `knowhub.config` for every configured endpoint. No independent parser, precedence, defaults or validation for `KNOWHUB_*` values

### T3 — Configuration package
- **Files:** `src/knowhub/config/{__init__,settings,secrets,feature_flags}.py`
- **Requirements:** SPEC-002 R1–R22 (all), excluding the deferred `policies.py` module
- **Acceptance:** M00-AC-009, M00-AC-010, M00-AC-011, M00-AC-012, M00-AC-013
- **Tests:** unit — ill-typed value rejected; unknown `KNOWHUB_*` key rejected; **unrelated host environment variables tolerated**; missing required setting fails startup; failure-message safety; precedence override; raw secret rejected by `SecretRef`; `repr`, `str` and JSON of a settings object render no secret
- **Failure paths:** all five rejection and failure cases above
- **Telemetry:** none — configuration loads before observability exists

### T4 — Observability package
- **Files:** `src/knowhub/observability/{__init__,bootstrap,correlation,logging}.py`, `docs/telemetry-naming.md`
- **Requirements:** SPEC-003 R1–R16 (all)
- **Acceptance:** M00-AC-014, M00-AC-015, M00-AC-016, M00-AC-017
- **Tests:** unit — correlation accepted when valid, generated when absent; **over-length rejected**; **control character rejected**; **malformed UUID rejected**; context survives `await`; every log record carries the identifier and trace/span IDs; no secret reaches a span attribute or log field; initialisation failure raises rather than degrading silently
- **Failure paths:** four rejection cases plus telemetry-initialisation failure
- **Telemetry:** this task is the telemetry — a span reaches the configured exporter, selectable as `otlp` / `console` / `none` by configuration alone

### T5 — Minimal API foundation
- **Files:** `src/knowhub/api/{__init__,main,health,middleware}.py`
- **Requirements:** SPEC-001 R8, R9, R10, R11; SPEC-003 R2, R10
- **Acceptance:** M00-AC-008, and M00-AC-015 end to end over HTTP
- **Tests:** unit via ASGI transport — `/healthz` and `/readyz` shapes; readiness reflects settings load and observability initialisation only; **readiness fails in a controlled, observable way** when observability initialisation did not complete; neither endpoint discloses configuration; `X-Correlation-ID` echoed; a malformed inbound header yields a fresh server-side identifier
- **Failure paths:** degraded readiness; malformed correlation header
- **Telemetry:** every request produces a span carrying the correlation ID; unhandled errors mark the span failed with a safe message

### T6 — Boundary enforcement
- **Files:** `pyproject.toml` (`[tool.importlinter]`), Ruff `banned-api` entries
- **Requirements:** SPEC-001 R12, R13, R14, R15, R16
- **Acceptance:** M00-AC-004
- **Tests:** the gate itself. The deliberate violation is introduced in an **ephemeral scratch worktree** during verification and never committed — the same pattern SPEC-004 R15 and M00-AC-023 mandate for the secret scanner
- **Failure paths:** the scratch violation must fail the gate, and its removal must restore a green run
- **Telemetry:** none

### T7 — Alembic scaffolding
- **Depends on:** T2a, T3
- **Files:** `alembic.ini`, `migrations/env.py`, `migrations/script.py.mako`, `migrations/versions/.gitkeep`
- **Requirements:** SPEC-004 R9, R10, R11
- **Acceptance:** M00-AC-020
- **Tests:** integration — `upgrade head` against a fresh database; run again against the already-migrated database; `downgrade base` then `upgrade head`; assert zero revision files exist
- **Failure paths:** a migration against an unreachable database fails with a safe error carrying no credential
- **Telemetry:** none — Alembic runs outside the served process at M00

### T8 — Test harness and failure-path tests
- **Depends on:** T2b, T3, T4, T5, T7
- **Files:** `tests/conftest.py`, `tests/unit/*`, `tests/integration/conftest.py`
- **Requirements:** SPEC-004 R22, R23, R24, R25
- **Acceptance:** M00-AC-024, M00-AC-026
- **Tests:** the harness — markers, fixtures, and integration-layer gating that skips with a clear message when the configured endpoint is unreachable rather than failing obscurely
- **Failure paths:** aggregates the configuration, boundary, readiness and migration failure paths named in R25
- **Telemetry:** exporter set to `none` for unit tests; asserted present for the API tests

### T9 — Container image and process mode
- **Files:** `Dockerfile`, `docker-entrypoint.sh`
- **Requirements:** SPEC-004 R17, R18, R19, R20, R21
- **Acceptance:** M00-AC-025
- **Tests:** build the image; run `api` mode; probe both endpoints; assert a span is emitted; assert an unknown mode exits non-zero
- **Failure paths:** unknown process mode; a container that fails to start on missing required configuration
- **Telemetry:** the containerised process emits spans identically to the local process

### T10 — CI wiring
- **Files:** `.github/workflows/ci.yml`
- **Requirements:** SPEC-004 R12, R13, R14, R15, R16; SPEC-001 R21, R22
- **Acceptance:** M00-AC-005, M00-AC-006, M00-AC-021, M00-AC-022, M00-AC-023
- **Tests:** the pipeline. Every gate in R13 present and blocking, including those that currently find nothing to check
- **Failure paths:** each gate proven able to fail — lint violation, type error, boundary violation, planted synthetic credential; all introduced ephemerally and removed
- **Telemetry:** none

### T11 — M00 verification pass
- **Files:** none in `knowhub-backend`; evidence recorded against this plan
- **Requirements:** none new — demonstrates the rest
- **Acceptance:** all 26, collected as evidence
- **Tests:** the §6 matrix executed and recorded, in the R1.1 environment, plus one Mode A equivalence run for M00-AC-018's external-service clause
- **Failure paths:** any criterion without evidence blocks milestone exit
- **Telemetry:** trace output captured as evidence for M00-AC-014 and M00-AC-025

## 6. Verification matrix

Requirement → task → evidence → acceptance criterion.

| AC | Requirements | Task | Evidence |
|---|---|---|---|
| M00-AC-001 | SPEC-001 R2 | T1 | Clean-clone `uv sync --frozen` build log |
| M00-AC-002 | SPEC-001 R3 | T1, T10 | `.python-version`, `requires-python`, CI runtime |
| M00-AC-003 | SPEC-001 R5, R6 | T1, T3, T4, T5 | Listing of `src/knowhub/` |
| M00-AC-004 | SPEC-001 R15, R16 | T6 | Scratch-worktree violation fails the gate; removal restores green |
| M00-AC-005 | SPEC-001 R17 | T10 | Planted lint violation fails CI |
| M00-AC-006 | SPEC-001 R18, R19 | T10 | Planted type error fails CI; configuration review shows no forced empty annotations |
| M00-AC-007 | SPEC-001 R1, R20, R21 | T1, T10 | `docs/adr/` and `docs/api/` present; no TypeScript source; secret scan clean |
| M00-AC-008 | SPEC-001 R8, R9, R10 | T5 | Endpoint tests including degraded readiness and non-disclosure |
| M00-AC-009 | SPEC-002 R2, R2.1, R2.2 | T3 | Unit tests: ill-typed rejected, unknown `KNOWHUB_*` rejected, foreign environment variables tolerated |
| M00-AC-010 | SPEC-002 R3, R4 | T3 | Missing-required startup failure plus message-safety assertion |
| M00-AC-011 | SPEC-002 R11, R12, R14 | T3 | Precedence test; mandatory-control non-weakening test |
| M00-AC-012 | SPEC-002 R15, R16, R17 | T3 | `SecretRef` rejection test; representation redaction test |
| M00-AC-012 (`.env.example` clause) | SPEC-002 R18 | T2b | `.env.example` review — placeholders only, every developer-supplied setting documented |
| M00-AC-013 | SPEC-002 R20, R21 | T3 | Local profile selected by configuration alone; no enabled bypass |
| M00-AC-014 | SPEC-003 R1, R2, R4 | T4, T5 | Span emitted for a liveness request; initialisation-failure test |
| M00-AC-015 | SPEC-003 R8, R9, R10, R11 | T4, T5 | Accept, generate and echo tests; over-length, control-character and malformed rejection; `await` propagation |
| M00-AC-016 | SPEC-003 R5, R6, R7 | T4 | Dependency tree shows no Azure telemetry SDK; exporter switches by configuration |
| M00-AC-017 | SPEC-003 R12, R14, R15 | T4 | Telemetry-safety tests; failed-span test; cardinality review |
| M00-AC-018 | SPEC-004 R1, R1.1, R1.2, R1.3, R2, R7 | T2a, T2b | R1.1 bring-up with all services healthy; Mode A equivalence run via the configured-endpoint check; no provisioning-detecting code |
| M00-AC-019 | SPEC-004 R3, R4 | T2a, T2b | `vector` present in `pg_available_extensions`; compose shows tag + digest; minimum documented and asserted by the check |
| M00-AC-020 | SPEC-004 R9, R10, R11 | T7 | Fresh, repeat and downgrade-upgrade runs; zero revision files |
| M00-AC-021 | SPEC-004 R12 | T10 | Full CI run with only the backend cloned; no frontend or workstation reference |
| M00-AC-022 | SPEC-004 R13, R14 | T10 | Workflow shows all nine gates blocking |
| M00-AC-023 | SPEC-004 R15 | T10 | Synthetic credential in an ephemeral input fails the scan; removal restores green; tree clean afterwards |
| M00-AC-024 | SPEC-004 R5, R6, R23, R24 | T8 | PostgreSQL integration test against the configured instance; absence of any Redis or Blob client confirmed |
| M00-AC-025 | SPEC-004 R17, R18, R19, R20 | T9 | Image build; API mode serves both probes and emits telemetry; unknown mode exits non-zero; no worker or scheduler code |
| M00-AC-026 | SPEC-004 R22, R25 | T8 | Test tree holds only `unit/` and `integration/`; failure-path tests pass |

### Requirement coverage

| Specification | Requirements | Owning tasks |
|---|---|---|
| M00-SPEC-001 (22) | R1–R3, R17–R22 | T1, T10 |
| | R4–R7 | T1, T3, T4, T5 |
| | R8–R11 | T5 |
| | R12–R16 | T6 |
| M00-SPEC-002 (24) | R1–R22 including R2.1, R2.2, except R18 | T3 |
| | R18 | T2b |
| M00-SPEC-003 (16) | R1–R7, R12–R16 | T4 |
| | R8–R11 | T4, T5 |
| M00-SPEC-004 (28) | R1.1, R2, R4, R7, R8 | T2a |
| | R1, R1.2, R1.3, R3, R5, R6 | T2b |
| | R9–R11 | T7 |
| | R12–R16 | T10 |
| | R17–R21 | T9 |
| | R22–R25 | T8 |

All 90 normative requirements across the four specifications are owned by
at least one task. All 26 acceptance criteria have named evidence.

## 7. Rollback and recovery

| Area | Failure | Recovery |
|---|---|---|
| Local setup | Service unhealthy, port or volume collision | In the R1.1 environment, `docker compose down -v` resets cleanly; the named unhealthy service identifies the culprit. Published ports are non-default precisely to avoid collision with existing services |
| Local setup | Environment drifted from the lockfile | `uv sync --frozen` is idempotent and authoritative. Never repair by installing packages by hand |
| Migrations | Chain fails mid-run | The chain is empty at M00, so `downgrade base` is always available |
| Migrations | Database needs resetting | **Operate on the dedicated KnowHub database only, never the server or cluster.** No documented command drops a database the developer has not explicitly named. An externally supplied PostgreSQL may host unrelated databases |
| Migrations | Wrong URL or unreachable service | Fails fast with a safe error. Fix the configuration, never `env.py` |
| Container build | Build fails after a digest bump | Digests are in version control — revert the line. Keep the previous image tag until the new one passes |
| Container build | Image starts but does not serve | The entrypoint exits non-zero naming the mode; check required configuration before suspecting code |
| CI | A gate fails on a legitimate change | Fix the code. Never disable, allowlist or relax a gate |
| CI | A gate itself is broken | Pin the tool version down and raise an issue. The gate stays blocking — a broken gate is a blocked merge, not a removed gate |
| CI | Ephemeral verification artefact left behind | T11 re-checks that the tree is clean after M00-AC-004 and M00-AC-023 |

## 8. Known risks

| # | Risk | Mitigation |
|---|---|---|
| 1 | An externally supplied PostgreSQL below 15, or without pgvector, passes M00 (which has no schema) and breaks at M1 | `scripts/check_dependencies.py` asserts the minimum major and pgvector availability, and is a documented step in both modes |
| 2 | Recovery steps could touch a developer's server holding unrelated databases | Recovery is scoped to the dedicated KnowHub database; see §7 |
| 3 | The R1.1 environment rots because everyday work uses existing services | SPEC-004 R1.1 names it the verification environment, M00-AC-018 requires evidence from it, and CI only ever runs it |
| 4 | PostgreSQL 15 reaches community end of life in November 2027 | Recorded in §3.2. Raising the minimum is a plan change, not a specification change |
| 5 | The exact Python patch and image digests are chosen at implementation time | Recorded in the lockfile, `.python-version` and the Compose and Docker files as part of T1 and T2 |
