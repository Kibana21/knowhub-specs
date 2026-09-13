# M00-T01 — Repository skeleton and dependency baseline

- **Status:** Completed
- **Depends on:** —
- **Blocks:** T02a, T03
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.1, §3.4, T1

## Owning specification requirements

M00-SPEC-001 R1, R2, R3, R17, R18, R19, R20, R22, and R4, R5, R6, R7 as
they apply to the packages this task creates

## Acceptance criteria contributed to

M00-AC-001, M00-AC-002, M00-AC-007 (in part — `docs/` layout and absence of
frontend source; the secret-scan clause is completed by T10)

## Files created or modified

| Path | Action |
|---|---|
| `pyproject.toml` | Create — project metadata, dependencies, Ruff, Pyright, pytest config |
| `uv.lock` | Create — committed in the same change as the manifest |
| `.python-version` | Create |
| `.gitignore` | Create |
| `.dockerignore` | Create |
| `src/knowhub/__init__.py` | Create |
| `docs/api/README.md` | Create |
| `README.md` | Modify |

`[tool.importlinter]` is added by T06, not here.

## Implementation steps

1. Declare project metadata and `requires-python = ">=3.12"` in
   `pyproject.toml`.
2. Pin `.python-version` to an exact 3.12 patch, chosen as the current 3.12
   security release at implementation time.
3. Declare the runtime and development dependency sets from plan §3.9.
   Nothing outside that list.
4. Resolve with `uv` and commit `uv.lock` in the same change.
5. Configure Ruff per plan §3.4 — target `py312`, line length 88, formatter
   enabled, the stated rule selection and per-file ignores.
6. Configure Pyright per plan §3.4 — `standard` mode, `pythonVersion
   "3.12"`, `include ["src", "tests"]`, the four enabled rules, the
   unknown-type family left off.
7. Configure pytest — `testpaths`, `--strict-markers`, the `integration`
   marker.
8. Create `src/knowhub/__init__.py` only. No other package.
9. Write `docs/api/README.md` stating the directory's purpose and that the
   OpenAPI artifact and its contract-update workflow arrive with the first
   business endpoint in M1.
10. Rewrite `README.md`: what the repository is, the Python baseline, and a
    placeholder pointing at the dependency and developer-loop sections T02a
    and T02b will add.

## Tests and evidence

- Clean clone, then `uv sync --frozen` produces a working environment with no
  other command.
- Ruff and Pyright both execute against the empty package without error.

## Failure paths to exercise

- `uv sync --locked` fails when `uv.lock` is stale against `pyproject.toml`.

> **Factual correction, found during implementation.** This line previously
> named `uv sync --frozen`. It does not assert lockfile freshness: `--frozen`
> means *sync without updating the lockfile* and exits 0 against a stale one.
> `--locked` asserts the lockfile will remain unchanged and exits non-zero;
> `uv lock --check` does the same as a standalone check. The requirement is
> unchanged — M00-SPEC-001 R2 still demands one reproducible resolution — only
> the command that demonstrates it was wrong.

## Telemetry expectations

None. No runtime code exists yet.

## Out of scope

Any package beyond `src/knowhub/__init__.py`; import-linter contracts (T06);
Dockerfile (T09); CI workflow (T10); `deploy/` contents; `docs/adr/` changes.

## Completion checklist

- [x] `uv sync --frozen` is the only path to a working environment
- [x] `.python-version` holds an exact 3.12 patch; `requires-python` is `">=3.12"`
- [x] Dependency set matches plan §3.9 exactly — no logging framework, no Redis client, no Azure SDK
- [x] Ruff and Pyright configured as specified and executing
- [x] `docs/adr/` untouched; `docs/api/README.md` has real content
- [x] No TypeScript or frontend source anywhere
- [x] No secret, token, credential or data snapshot committed
