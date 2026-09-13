# ADR-016 — Two independently versioned repositories

- **Status:** Accepted
- **Scope:** product
- **Origin:** master blueprint §82
- **Supersedes:** —
- **Superseded by:** —

## Decision

KnowHub is delivered as two independently versioned repositories: `knowhub-backend` and `knowhub-frontend`.

## Rationale / consequence

Pure-Python backend and UI-focused TypeScript frontend have clean ownership, independent CI/CD and independent scaling/release cadence.

---

Origin: [master blueprint §82](../../knowhub_master_blueprint.html#adr-register) — the frozen origin register.
Decision and rationale above are reproduced from §82 without addition.
Registered in [the ADR register](README.md).
