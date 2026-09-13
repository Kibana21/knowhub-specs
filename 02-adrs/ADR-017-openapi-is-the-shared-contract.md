# ADR-017 — OpenAPI is the shared frontend/backend contract

- **Status:** Accepted
- **Scope:** product
- **Origin:** master blueprint §82
- **Supersedes:** —
- **Superseded by:** —

## Decision

OpenAPI is the frontend/backend source-of-contract; no shared source-code DTO library.

## Rationale / consequence

Backend Pydantic models remain authoritative while frontend types/client are generated reproducibly from a pinned contract artifact.

---

Origin: [master blueprint §82](../../knowhub_master_blueprint.html#adr-register) — the frozen origin register.
Decision and rationale above are reproduced from §82 without addition.
Registered in [the ADR register](README.md).
