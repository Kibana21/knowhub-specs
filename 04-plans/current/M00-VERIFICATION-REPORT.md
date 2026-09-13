# M00 verification report

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Task:** [M00-T11](M00-tasks/M00-T11-verification-pass.md)
- **Scope:** verification only. No capability was designed or implemented.
- **Plan:** [M00-PLAN-backend-foundation](M00-PLAN-backend-foundation.md)

## 1. Executive verdict

**M00 CONDITIONALLY ACCEPTED.**

Implementation is complete and every locally verifiable gate is green:
90/90 normative requirements are owned, 117/117 tests pass, all quality and
security gates pass, and both provisioning modes are proven equivalent.

**Three acceptance criteria cannot be closed yet.** M00-AC-005, M00-AC-006
and M00-AC-022 assert that gates *fail CI* — a property of the GitHub Actions
workflow, not of the commands it runs. `.github/` is **untracked**, so the
workflow has never been pushed and has never executed. Their underlying
commands are proven locally; the CI execution is not. They are recorded
**EXTERNAL-EVIDENCE-PENDING**, not PASS.

§2.6 of this report lists exactly what moves the verdict to ACCEPTED.

## 2. Requirement coverage — 90/90

Verified programmatically against the approved specifications and task files.

| Specification | Defined | Owned | Uncovered |
|---|---|---|---|
| M00-SPEC-001 | 22 | 22 | none |
| M00-SPEC-002 | 24 | 24 | none |
| M00-SPEC-003 | 16 | 16 | none |
| M00-SPEC-004 | 28 | 28 | none |
| **Total** | **90** | **90** | **none** |

No requirement was silently dropped. Nine requirements are claimed by more
than one task; each is a deliberate shared responsibility recorded in the task
files, not a duplicate claim:

| Requirement | Tasks | Why shared |
|---|---|---|
| SPEC-001 R4–R7 | T01, T03, T04, T05 | Package inventory: each task owns the packages it creates; T05 completes and verifies the whole |
| SPEC-001 R22 | T01, T10 | Builds without the frontend repo — asserted at setup and again in CI |
| SPEC-003 R2, R10 | T04, T05 | Observability provides the contract; the API exercises it at the request boundary |
| SPEC-004 R4 | T02a, T02b | Image pinning vs the minimum for an externally supplied instance |
| SPEC-004 R24 | T02b, T08 | Availability observation vs the rule that it is not an integration test |

**No conflicting product rule was introduced.** The one candidate — a
Local-only telemetry policy — was identified during T03 review as invented and
removed; `enforce_mandatory_floors` now ships as a mechanism with **zero**
production floors, verified by introspection.

**SPEC-002 R18 ownership was corrected** during T08 from T03 to T02b, because
`.env.example` is T02b's file. No requirement remains falsely claimed.

## 3. Acceptance matrix — 23 PASS, 3 pending

| AC | Requirements | Task | Evidence | Result | Status |
|---|---|---|---|---|---|
| 001 | S1 R2 | T01 | Clean-checkout `uv sync --frozen` builds a working environment; `uv lock --check` exit 0 | working env, exit 0 | **PASS** |
| 002 | S1 R3 | T01 | `.python-version`=3.12.12; `requires-python=">=3.12"`; venv runs 3.12.12 | matches | **PASS** |
| 003 | S1 R5, R6 | T05 | `ls src/knowhub` → `api config observability`; 0 empty package dirs | exactly 3 | **PASS** |
| 004 | S1 R15, R16 | T06 | Scratch violations (`observability→starlette`, `config→observability`) → `lint-imports` exit 1; restored → 3 kept, 0 broken | fails then recovers | **PASS** |
| 005 | S1 R17 | T10 | `ruff check` proven to fail on a planted violation locally; **CI enforcement unproven** | command proven | **EXTERNAL-EVIDENCE-PENDING** |
| 006 | S1 R18, R19 | T10 | `pyright` proven to fail on a planted type error locally; **CI enforcement unproven** | command proven | **EXTERNAL-EVIDENCE-PENDING** |
| 007 | S1 R1, R20, R21 | T01 | `docs/adr/`, `docs/api/` present; 0 TypeScript/frontend files; gitleaks tree+history clean | clean | **PASS** |
| 008 | S1 R8–R10 | T05 | `/healthz` 200 `{"status":"ok"}`; `/readyz` 200; degraded readiness → 503 `pending:["observability"]`; 10 leak patterns absent | all pass | **PASS** |
| 009 | S2 R2, R2.1, R2.2 | T03 | Unknown `KNOWHUB_*` (4 shapes) rejected; `CI`/`KUBERNETES_SERVICE_HOST`/`AWS_REGION` tolerated | verified | **PASS** |
| 010 | S2 R3, R4 | T03 | Missing required → startup failure naming the setting; no value, no env dump | verified | **PASS** |
| 011 | S2 R11, R12, R14 | T03 | Env overrides default; mandatory-floor mechanism rejects weakening incl. nested | verified | **PASS** |
| 012 | S2 R15–R18 | T03, T02b | `SecretRef` rejects plaintext and unsupported scheme; repr/str/JSON leak nothing; `.env.example` documents 13/13 settings, placeholders only | verified | **PASS** |
| 013 | S2 R20, R21 | T03 | Local profile by configuration alone; no enabled bypass | verified | **PASS** |
| 014 | S3 R1, R2, R4 | T04 | Span emitted for a request; init failure raises `TelemetryInitialisationError` | verified | **PASS** |
| 015 | S3 R8–R11 | T04, T05 | Valid echoed; absent generated; over-length/control-char/malformed replaced; survives `await`; container-verified | verified | **PASS** |
| 016 | S3 R5–R7 | T04 | No Azure/AppInsights/OpenCensus distribution installed; exporter switches by configuration | verified | **PASS** |
| 017 | S3 R12, R14, R15 | T04 | Both canaries absent from spans, events, `exception.message`, `exception.stacktrace`, status description | verified | **PASS** |
| 018 | S4 R1–R1.3, R2, R7 | T02a, T02b | Managed: 3 services healthy. Mode A: native PostgreSQL + existing Redis + Compose Azurite, same checker, exit 0 | both modes | **PASS** |
| 019 | S4 R3, R4 | T02a, T02b | `pgvector_available=1`, `vector_created=0`; images pinned by tag + index digest | verified | **PASS** |
| 020 | S4 R9–R11 | T07 | upgrade/upgrade/downgrade/upgrade all exit 0 on both instances; 0 revision files; only `alembic_version` | verified | **PASS** |
| 021 | S4 R12 | T10 | Workflow contains no `knowhub-frontend` reference except a comment; no workstation endpoint | inspected | **PASS** |
| 022 | S4 R13, R14 | T10 | All nine gates present in the workflow and each proven able to fail locally; **blocking behaviour unproven** | commands proven | **EXTERNAL-EVIDENCE-PENDING** |
| 023 | S4 R15 | T10 | Synthetic GitHub-PAT in ephemeral copy → gitleaks exit 1; removal → exit 0; nothing committed | verified | **PASS** |
| 024 | S4 R5, R6, R23, R24 | T08 | Integration runs against the configured PostgreSQL; 0 Redis/Blob clients in `src/` | verified | **PASS** |
| 025 | S4 R17–R20 | T09 | Image builds; UID 10001; `/healthz` and `/readyz` 200; unknown mode exit 64; no worker/scheduler code | verified | **PASS** |
| 026 | S4 R22, R25 | T08 | `tests/` holds only `unit` and `integration`; 0 empty layers; failure paths exercised | verified | **PASS** |

M00-AC-023 is **PASS**: the synthetic-credential proof is a local property of
the scanner, fully demonstrated. Only the three that assert *CI blocks a
merge* remain pending.

## 4. Quality gate results

| Gate | Result |
|---|---|
| `uv lock --check` | exit 0 |
| `uv sync --frozen` | exit 0 |
| `ruff check .` | exit 0 |
| `ruff format --check .` | exit 0 |
| `pyright` | exit 0 — 0 errors, 0 warnings |
| `lint-imports` | exit 0 — 3 contracts kept, 0 broken |
| `scripts/check_dependencies.py` | exit 0 — all three services |
| `pip-audit` (real set) | exit 0 — no known vulnerabilities |
| `pip-audit` (synthetic `jinja2==2.11.2`) | exit 1 — 10 advisories |
| gitleaks, working tree | exit 0 — no leaks found |
| gitleaks, git history | exit 0 — no leaks found |
| gitleaks, synthetic credential | exit 1 — leaks found: 1 |
| `docker build` | exit 0 |
| Container smoke test | UID 10001, `/healthz` 200, `/readyz` 200, unknown mode 64 |

### Test totals

| Layer | Count | Result |
|---|---|---|
| Unit | 109 | pass |
| Integration | 8 | pass, **0 skipped** |
| **Total** | **117** | **pass** |

Skip-gate evidence: `collected=8 executed=8 skipped=0 failures=0 errors=0`.
With configuration removed the same suite reports 8 skipped and pytest exits
0, and `scripts/assert_test_run.py` exits 1 — a silent skip cannot pass.

## 5. Managed clean-room evidence

Compose up with `--wait`: PostgreSQL, Redis, Azurite all **healthy**.

- `server_version_num=150019`, `pgvector_available=1`, **`vector_created=0`**, `user_tables=0`
- Redis `PING` → `PONG`
- Azurite → `HTTP/1.1 400` (an HTTP-level answer from the service)
- Migrations: upgrade, repeat, downgrade base, upgrade — all exit 0; 0 revision files; only `alembic_version` afterwards
- API: `/healthz` 200, `/readyz` 200, correlation echoed, malformed replaced
- Telemetry: 4 spans, **1** "observability initialised", **0** override warnings, **0** secrets in the log
- Container: build 0, UID 10001, both probes 200, unknown mode 64

Torn down with `down -v`, exit 0.

## 6. Existing-services-mode evidence

Compose PostgreSQL and Redis **stopped**; only Azurite left running.

| Service | Provisioning | Result |
|---|---|---|
| PostgreSQL | native Homebrew 15.12, `127.0.0.1:5432`, `knowhub_db` | `ok postgresql server_version_num 150012, pgvector available (not created)` |
| Redis | pre-existing Docker container `redis:7` on 6379 | `ok redis PING answered +PONG` |
| Object storage | Compose Azurite (the available M00 mechanism) | `ok blob HTTP 400 from endpoint` |

Migrations exit 0; integration suite **8 passed, 0 skipped**; API `/healthz`
and `/readyz` both 200.

**Equivalence, not a second architecture:** one settings model
(`knowhub.config`), **0** modules outside it parsing `KNOWHUB_*`, and **0**
code branches on provenance — the only `docker`/`compose` matches in `src/`
are the words "composed"/"Compose the DSN" in docstrings.

## 7. Security evidence

| Invariant | Result |
|---|---|
| Unknown `KNOWHUB_*` keys fail | PASS |
| Unrelated environment variables tolerated | PASS |
| Inline/plaintext secrets rejected, not echoed | PASS |
| Secrets absent from repr/str/JSON | PASS |
| Configuration errors do not dump the environment | PASS |
| Correlation rejects malformed/unsafe input | PASS |
| Logs emit no arbitrary exception text | PASS — `error` is `{"type": ...}` only |
| Spans export no `exception.message` | PASS — both canaries |
| Spans export no `exception.stacktrace` | PASS — both canaries |
| ERROR status descriptions carry no arbitrary text | PASS — replaced even with no exception event |
| Connection-string canary absent everywhere | PASS |
| gitleaks detects a synthetic credential | PASS — exit 1, removal restores exit 0 |
| pip-audit detects a known-vulnerable requirement | PASS — exit 1, 10 advisories |

84 security-relevant tests pass. Retained diagnostics: `exception.type`,
ERROR status code, route-template span name, correlation/trace/span IDs.

All negative fixtures were ephemeral. Nothing synthetic was committed.

## 8. Architecture evidence

| Invariant | Result |
|---|---|
| Only approved packages | `api`, `config`, `observability` |
| Empty future packages | 0 |
| Empty test layers | 0 — only `unit`, `integration` |
| Config framework-light | 0 fastapi/starlette/opentelemetry imports |
| Observability independent of the web layer | 0 fastapi/starlette imports |
| API is the only web layer | only `api` imports fastapi/starlette |
| Redis client / Azure SDK / Taskiq / Celery | 0 each |
| Worker or scheduler process mode | 0 branches |
| `target_metadata` | `None` |
| Business migrations | 0 |
| `vector` extension created | no |
| Frontend source in the backend repo | 0 files |
| Reference repositories | `terrain-main`, `deepwiki-rs-main` — 0 files changed |

import-linter: **3 contracts kept, 0 broken**.

## 9. Container evidence

| Property | Result |
|---|---|
| Build Python | 3.12.14 (recorded in `/app/.venv/BUILD_PYTHON_VERSION`) |
| Runtime Python | 3.12.14 |
| Identical | **yes** |
| UID | 10001 |
| `/app` contents | `.venv` only |
| `.git` / `.env` anywhere | 0 / 0 |
| Project source tree, tests | absent |
| Dev tools | none of pytest, ruff, pyright, pip-audit, import-linter |
| `uv`, compiler | absent |
| Frontend/reference repositories | absent |
| Process modes | `api` only; unknown modes exit 64 |
| Observability initialisation | exactly once; 0 override warnings |

47 distributions, project installed non-editable.

## 10. CI evidence

**Locally verified (A).** Every command the workflow runs was executed here,
and every gate was proven able to fail with the change restored
byte-identically: stale lockfile, Ruff lint, Ruff format, Pyright, import
boundary, unit-test failure, integration skipped, integration failure,
gitleaks synthetic credential, pip-audit vulnerable pin, Docker build failure,
container start failure.

**Not verified (B) — no GitHub Actions run exists.** `.github/` is
**untracked**: the workflow has never been committed, pushed or executed. The
`gh` CLI is unavailable and there are no run artefacts. These remain
**EXTERNAL-EVIDENCE-PENDING**:

1. GitHub parses and executes the workflow (YAML schema, `needs:` wiring)
2. The pinned action SHAs resolve on the runner
3. `astral-sh/setup-uv` behaviour and caching
4. `${{ github.workspace }}` expansion in the gitleaks volume mounts
5. `ubuntu-latest` provides `docker compose` and `curl`, and port 8000 is bindable
6. `fetch-depth: 0` gives gitleaks full history
7. `needs:` short-circuits later jobs when an earlier gate fails

Local execution proves the commands, not the workflow.

## 11. Known limitations and intentional deferrals

- Migrations are **not** in the runtime image. T09 approved only the `api`
  mode. A deployed environment running migrations from this image would need
  them added — a deployment decision, deliberately not taken here.
- `pip` is present in the runtime image, inherited from the Debian base. It
  cannot resolve anything — there is no manifest — but it is attack surface.
- `.python-version` (3.12.12) and the container (3.12.14) differ. Both are
  3.12 and `requires-python` holds; the container is internally consistent.
  uv's managed builds do not yet publish 3.12.14.
- Deferred by specification, not by omission: business schema and the
  canonical data model (M1), identity and authorization (M1), the OpenAPI
  artifact and its compatibility gate (M1/0B), Taskiq workers and the run
  model (M2), the Blob abstraction and Redis client (M2), contract/golden/
  parity/security test layers (M3+), `deploy/` contents and IaC (post-0B).

## 12. Git and worktree state

- **Nothing staged. Nothing committed or pushed by this verification.**
- Modified: `.dockerignore`, `README.md`, `pyproject.toml`,
  `src/knowhub/config/settings.py`
- Untracked: `.env.example`, `.github/`, `Dockerfile`, `alembic.ini`,
  `docker-entrypoint.sh`, `docs/telemetry-naming.md`, `migrations/`,
  `scripts/`, `src/knowhub/api/`, `src/knowhub/observability/`, `tests/`
- Commits: `49fa688`, `28c5443`, `21a1154`. `49fa688` was made by the
  repository owner and contains T01–T03 output; everything after is untracked.
- No generated or scratch files remain: 0 JUnit reports, 0 exported
  requirements, 0 `.env`, 0 stray logs. `.ruff_cache` and `.pytest_cache` are
  git-ignored.
- `knowhub-specs/.DS_Store` remains **tracked** (unmodified). Cleanup was
  offered and not requested; it is untouched.

## 13. Final recommendation

**M00 CONDITIONALLY ACCEPTED.** The foundation is complete, internally
consistent and evidenced. Later milestones can build on it.

To reach **M00 ACCEPTED**, one thing is required:

> Commit and push the M00 work — including `.github/workflows/ci.yml` — and
> let the workflow run once on a pull request or on `main`. A green run closes
> items 1–6 of §10; a run where an earlier job fails and later jobs are skipped
> closes item 7.

That run is also what converts **M00-AC-005**, **M00-AC-006** and
**M00-AC-022** from EXTERNAL-EVIDENCE-PENDING to PASS. No code change is
expected; if the run surfaces a workflow defect, it is fixed and re-run.

The M00 plan should **remain in `04-plans/current/`** until that run exists.
