# M00-T06 — Boundary enforcement

- **Status:** Completed
- **Depends on:** T03, T04, T05
- **Blocks:** T10
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.4, T6

## Owning specification requirements

M00-SPEC-001 R12, R13, R14, R15, R16

## Acceptance criteria contributed to

M00-AC-004

## Files created or modified

| Path | Action |
|---|---|
| `pyproject.toml` | Modify — add `[tool.importlinter]` and Ruff `banned-api` entries |

## Implementation steps

1. Add `import-linter` contracts expressing the M00-realisable subset of the
   §71 rules:
   - **Layered**, top to bottom: `knowhub.api` → `knowhub.observability` →
     `knowhub.config`.
   - **Forbidden:** `knowhub.config` → `fastapi`.
   - **Forbidden:** `knowhub.config` → `opentelemetry`.
   - **Forbidden:** `knowhub.observability` → `fastapi`.
2. Contracts may reference only modules that exist — import-linter errors on
   a contract naming an absent module. Partial coverage at M00 is expected
   and is what SPEC-001 R16 anticipates; the mechanism must nonetheless be
   wired, gating and demonstrably able to fail.
3. Add Ruff `flake8-tidy-imports` settings: ban relative imports, and carry a
   `banned-api` subset as cheap defence in depth. import-linter remains the
   gate.
4. Record in the contract file that later milestones extend these contracts
   as their packages arrive.

## Tests and evidence

- The boundary check runs clean against the T03–T05 tree.
- The check is wired into the local validation sweep and, by T10, into CI as
  a blocking gate.

## Failure paths to exercise

Introduce a violating import — for example `knowhub.observability` importing
FastAPI — in an **ephemeral scratch worktree**, confirm the gate fails, then
remove it and confirm a green run. The violation is never committed, mirroring
the synthetic-credential procedure M00-AC-023 requires.

## Telemetry expectations

None.

## Out of scope

Contracts for packages that do not exist yet. Editing a contract to permit an
import that fails — the import is what changes. Any change to the §71 rules
themselves, which would be a specification or ADR question.

## Completion checklist

- [x] Layered contract and three forbidden contracts present
- [x] Every contract references only existing modules
- [x] Gate runs clean on the current tree
- [x] A scratch violation demonstrably fails the gate; removal restores green
- [x] No violation or scratch artefact committed
- [x] No contract loosened to accommodate existing code
