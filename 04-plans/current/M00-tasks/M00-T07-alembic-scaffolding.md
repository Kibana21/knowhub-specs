# M00-T07 — Alembic scaffolding

- **Status:** Not started
- **Depends on:** T02a, T03
- **Blocks:** T08
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.7, T7

## Owning specification requirements

M00-SPEC-004 R9, R10, R11

## Acceptance criteria contributed to

M00-AC-020

## Files created or modified

| Path | Action |
|---|---|
| `alembic.ini` | Create — repository root |
| `migrations/env.py` | Create |
| `migrations/script.py.mako` | Create |
| `migrations/versions/.gitkeep` | Create |

## Implementation steps

1. Place `alembic.ini` at the repository root and point it at `migrations/`.
2. `env.py` obtains the database URL from `knowhub.config`, resolving the
   password through the `SecretRef` resolver. This keeps one source of truth
   for the URL and exercises the secret machinery for real rather than in a
   test-only path.
3. Set `target_metadata = None`. No models exist at M00, so autogenerate is
   correctly unusable until M1.
4. Create `migrations/versions/` with a `.gitkeep` so the empty chain is
   committed. **No revision file.**
5. Ensure the migration command is repeatable — from a fresh database and
   from an already-migrated one — and that `downgrade base` is available.
6. Migration failure must produce a safe error carrying no credential or
   connection string.

## Tests and evidence

Integration tests against the configured PostgreSQL:

- `upgrade head` succeeds on a fresh database.
- `upgrade head` run again against the already-migrated database succeeds.
- `downgrade base` then `upgrade head` succeeds.
- Assert zero revision files exist under `migrations/versions/`.

## Failure paths to exercise

Migration against an unreachable database fails with a safe error that
discloses no credential.

## Telemetry expectations

None. Alembic runs outside the served process at M00.

## Out of scope

Any business schema migration — the canonical data model is M1. Any
SQLAlchemy model or `knowhub.persistence` package. `CREATE EXTENSION vector`.
Editing an applied revision — the chain is extended, never rewritten.

## Completion checklist

- [ ] `env.py` reads the URL from `knowhub.config`; no second configuration path
- [ ] `target_metadata` is `None`
- [ ] Zero revision files; `.gitkeep` present
- [ ] Fresh, repeat and downgrade-upgrade runs all succeed
- [ ] Failure against an unreachable database leaks no credential
- [ ] No model, no persistence package, no extension created
