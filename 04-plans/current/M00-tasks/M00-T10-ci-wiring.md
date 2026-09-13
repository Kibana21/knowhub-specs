# M00-T10 — CI wiring

- **Status:** Not started
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

## Out of scope

OpenAPI artifact publication and the compatibility-diff gate — no business
API surface exists. Golden, contract, parity, security and evaluation gates.
Cross-repository compatibility attestation. SBOM generation. Deployment,
promotion or environment pipelines. Any dependency on a developer workstation.

## Completion checklist

- [ ] All nine R13 gates present and blocking
- [ ] Setup runs `uv lock --check` before `uv sync --frozen`
- [ ] Runs with only `knowhub-backend` cloned
- [ ] No reference to `knowhub-frontend` or any workstation service
- [ ] Dependency services come from the T02a Compose definition on the runner
- [ ] Every action pinned by commit SHA; least-privilege permissions
- [ ] Each gate demonstrated able to fail and restored to green
- [ ] No synthetic credential or scratch artefact left in the repository
