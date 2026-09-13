# ADR-022 — Hybrid RBAC/ABAC authorization with default deny

- **Status:** Accepted
- **Scope:** product
- **Origin:** master blueprint §82
- **Supersedes:** —
- **Superseded by:** —

## Decision

Authorization is hybrid RBAC + scoped assignments + source ACL + classification policy with default deny.

## Rationale / consequence

Platform/module privilege and evidence entitlement are distinct; frontend visibility never replaces backend enforcement.

---

Origin: [master blueprint §82](../../knowhub_master_blueprint.html#adr-register) — the frozen origin register.
Decision and rationale above are reproduced from §82 without addition.
Registered in [the ADR register](README.md).
