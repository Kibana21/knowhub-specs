# ADR-021 — Authentication is provider-pluggable

- **Status:** Accepted
- **Scope:** product
- **Origin:** master blueprint §82
- **Supersedes:** —
- **Superseded by:** —

## Decision

KnowHub authentication is provider-pluggable: secure built-in LOCAL authentication and generic OIDC/PKCE providers terminate in one internal KnowHub session/access-token model behind a minimal frontend BFF.

## Rationale / consequence

Organizations can start without Entra and later enable Entra/Okta/Keycloak/other OIDC without changing RBAC/ACL logic. Browser JavaScript never persists access/refresh tokens.

---

Origin: [master blueprint §82](../../knowhub_master_blueprint.html#adr-register) — the frozen origin register.
Decision and rationale above are reproduced from §82 without addition.
Registered in [the ADR register](README.md).
