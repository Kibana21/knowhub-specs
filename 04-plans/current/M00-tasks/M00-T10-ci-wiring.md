# M00-T10 — CI wiring

- **Status:** Completed
- **Depends on:** T01, T02a, T02b, T03, T04, T05, T06, T07, T08, T09
- **Blocks:** T11
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.7, T10

## Owning specification requirements

M00-SPEC-004 R12, R13, R14, R15, R16; M00-SPEC-001 R21, R22

## Acceptance criteria contributed to

M00-AC-005, M00-AC-006, M00-AC-021, M00-AC-022, M00-AC-023

## Files created or modified

| Path | Action |
|---|---|
| `.github/workflows/ci.yml` | Create |

## Implementation steps

1. One workflow with jobs `quality` → `unit` → `integration` → `security` →
   `image`. Name jobs by outcome so a later move to another CI platform is a
   translation rather than a redesign.
2. Wire every gate required by SPEC-004 R13 as **blocking**: lint and format,
   type check, package boundaries, unit tests, integration tests, migration
   validation, dependency scan, secret scan, container build. A gate that
   currently finds nothing to check is still wired and blocking.
3. Use `astral-sh/setup-uv` with caching, then this sequence — in this order:

   ```sh
   uv lock --check    # proves pyproject.toml and uv.lock are consistent
   uv sync --frozen   # installs strictly from the committed lock, unmodified
   ```

   Never rely on `--frozen` alone to detect a stale lock: it means *sync
   without updating the lockfile* and exits 0 against a stale one. Pin every
   action by commit SHA.
4. Set `permissions: contents: read` and a concurrency group per ref.
5. Bring dependency services up with the **T02a Compose definition** on the
   ephemeral runner. **No job, step, script or cached artifact may reference
   or reach a workstation-local service.**
6. Dependency scan: `pip-audit` against `uv export --format requirements-txt`.
7. Secret scan: `gitleaks` over tree and history.
8. No job, step or cached artifact may reference `knowhub-frontend` or
   require it to be present.

## Tests and evidence

- A full run completes with only `knowhub-backend` cloned.
- The workflow file shows all nine gates present and blocking.
- A search of the workflow finds no reference to `knowhub-frontend` and no
  workstation endpoint.

## Failure paths to exercise

Each gate proven able to fail, then restored:

- A planted lint violation fails the lint gate.
- A planted type error fails the type gate.
- A planted boundary violation fails the boundary gate.
- A manifest change without a corresponding lock update fails `uv lock --check`.
- A **synthetic, known-test credential placed in an ephemeral verification
  input** — a scratch worktree, fixture or scan target created for the check
  and discarded after it — fails the secret scan. No real or persistent
  credential is committed, and no detectable test credential remains
  afterwards.

## Telemetry expectations

None.

## Watch-item carried forward from T02a

`docker-compose.dev.yml` commits a **local-development sentinel** as the
PostgreSQL password. It is not a real credential — SPEC-004 R8 permits
non-production values as Compose-local configuration — but the secret scanner
will very likely flag it.

When it does:

- **do not** broadly allowlist `docker-compose.dev.yml`;
- **do not** broadly allowlist password-like patterns;
- use the **narrowest possible** allowlist, scoped to that exact known value
  in that exact file context;
- then verify that a *different* synthetic credential placed elsewhere in the
  repository still fails the scan, so the allowlist has not blunted the gate.

This narrows one known non-credential. It is not a precedent for allowlisting
anything else, and never for a real credential.

> **Outcome (T10).** No allowlist was needed. gitleaks v8.30.1 does **not**
> flag `knowhub_local_dev_only` — a scan of the working tree and of git
> history both report "no leaks found". The gate therefore runs with **zero
> allowlist entries**, which is the strongest possible configuration. The
> guidance above still stands if a future ruleset does flag it.
>
> Effectiveness was proved separately: a synthetic GitHub-PAT-shaped
> credential placed in an ephemeral scratch copy fails the scan (exit 1), and
> its removal restores a clean run. Nothing was committed to the repository.

## Out of scope

OpenAPI artifact publication and the compatibility-diff gate — no business
API surface exists. Golden, contract, parity, security and evaluation gates.
Cross-repository compatibility attestation. SBOM generation. Deployment,
promotion or environment pipelines. Any dependency on a developer workstation.

## Completion checklist

- [x] All nine R13 gates present and blocking
- [x] Setup runs `uv lock --check` before `uv sync --frozen`
- [x] Runs with only `knowhub-backend` cloned
- [x] No reference to `knowhub-frontend` or any workstation service
- [x] Dependency services come from the T02a Compose definition on the runner
- [x] Every action pinned by commit SHA; least-privilege permissions
- [x] Each gate demonstrated able to fail and restored to green
- [x] No synthetic credential or scratch artefact left in the repository
