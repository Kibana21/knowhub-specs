# ADR-025 — Canonical user model is independent of auth provider

- **Status:** Accepted
- **Scope:** product
- **Origin:** master blueprint §82
- **Supersedes:** —
- **Superseded by:** —

## Decision

The canonical KnowHub user and authorization model are independent of authentication provider.

## Rationale / consequence

LOCAL, Entra, Okta, Keycloak and future OIDC identities link to the same internal user/principal, allowing SSO adoption without migrating roles, memberships, approvals, ownership or audit history.

---

Origin: [master blueprint §82](../../knowhub_master_blueprint.html#adr-register) — the frozen origin register.
Decision and rationale above are reproduced from §82 without addition.
Registered in [the ADR register](README.md).
