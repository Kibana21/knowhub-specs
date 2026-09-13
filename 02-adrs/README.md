# ADR Register

The live Architectural Decision Record register for the KnowHub workspace.
It covers **every** ADR regardless of which repository holds the file.

Governed by [ADR-026](ADR-026-adr-ownership-and-location.md), which defines
ADR ownership by architectural scope, the canonical locations, the single
global numbering sequence, this register's behaviour and how superseding
works. Read ADR-026 before adding, moving or superseding an ADR.

## How to read this register

- **Summary level only.** Each row carries number, title, status, scope,
  owner, canonical location and supersede links. Context, decision text,
  rationale and consequences live only in the canonical ADR file.
- **One canonical file per number**, in exactly one repository. No repository
  holds a copy of another's ADR.
- **Location** is relative to the owning repository root. Rows owned by
  `knowhub-backend` or `knowhub-frontend` resolve as
  `https://github.com/Kibana21/<repo>/blob/main/<location>`.
- **Numbers are global** and encode nothing about location. Only this table
  maps a number to where its file lives.
- **ADR-001 – ADR-025 originate in blueprint §82**, which remains the frozen
  historical/origin register for them. Their decision and rationale text is
  reproduced from §82 without addition. Future ADRs are registered here
  only; the blueprint is not edited to synchronize ADRs.

## Canonical locations

| Scope | Location |
|---|---|
| product-wide | `knowhub-specs/02-adrs/` |
| backend-specific | `knowhub-backend/docs/adr/` |
| frontend-specific | `knowhub-frontend/docs/adr/` |

## Register

| ADR | Title | Status | Scope / owner | Canonical location | Supersedes | Superseded by |
|---|---|---|---|---|---|---|
| ADR-001 | KnowHub is Python-first and API-first | Accepted | product — `knowhub-specs` | [`ADR-001-python-first-api-first.md`](ADR-001-python-first-api-first.md) | — | — |
| ADR-002 | Application is the primary enterprise scope | Accepted | product — `knowhub-specs` | [`ADR-002-application-is-primary-scope.md`](ADR-002-application-is-primary-scope.md) | — | — |
| ADR-003 | PostgreSQL is canonical | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-003-postgresql-is-canonical.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-003-postgresql-is-canonical.md) | — | — |
| ADR-004 | pgvector and Postgres FTS first | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-004-pgvector-and-postgres-fts-first.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-004-pgvector-and-postgres-fts-first.md) | — | — |
| ADR-005 | Graph stored canonically in Postgres | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-005-graph-canonical-in-postgres.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-005-graph-canonical-in-postgres.md) | — | — |
| ADR-006 | Taskiq and Redis for V1 asynchronous work | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-006-taskiq-and-redis-for-async-work.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-006-taskiq-and-redis-for-async-work.md) | — | — |
| ADR-007 | Task result backend is not business state | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-007-task-result-backend-is-not-business-state.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-007-task-result-backend-is-not-business-state.md) | — | — |
| ADR-008 | LangGraph for reasoning workflows only | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-008-langgraph-for-reasoning-workflows-only.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-008-langgraph-for-reasoning-workflows-only.md) | — | — |
| ADR-009 | Knowledge Foundry is the KnowHub subsystem name | Accepted | product — `knowhub-specs` | [`ADR-009-knowledge-foundry-naming.md`](ADR-009-knowledge-foundry-naming.md) | — | — |
| ADR-010 | Human documentation and agent context are separate | Accepted | product — `knowhub-specs` | [`ADR-010-human-docs-and-agent-context-separate.md`](ADR-010-human-docs-and-agent-context-separate.md) | — | — |
| ADR-011 | Permission checks precede LLM reasoning | Accepted | product — `knowhub-specs` | [`ADR-011-permission-checks-precede-llm-reasoning.md`](ADR-011-permission-checks-precede-llm-reasoning.md) | — | — |
| ADR-012 | Derived stores are rebuildable | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-012-derived-stores-are-rebuildable.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-012-derived-stores-are-rebuildable.md) | — | — |
| ADR-013 | Deterministic extraction before LLM inference | Accepted | product — `knowhub-specs` | [`ADR-013-deterministic-extraction-before-llm-inference.md`](ADR-013-deterministic-extraction-before-llm-inference.md) | — | — |
| ADR-014 | No unrestricted shell or network tools | Accepted | product — `knowhub-specs` | [`ADR-014-no-unrestricted-shell-or-network-tools.md`](ADR-014-no-unrestricted-shell-or-network-tools.md) | — | — |
| ADR-015 | Internal domain events modeled, event bus deferred | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-015-internal-domain-events-modeled-bus-deferred.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-015-internal-domain-events-modeled-bus-deferred.md) | — | — |
| ADR-016 | Two independently versioned repositories | Accepted | product — `knowhub-specs` | [`ADR-016-two-independently-versioned-repositories.md`](ADR-016-two-independently-versioned-repositories.md) | — | — |
| ADR-017 | OpenAPI is the shared frontend/backend contract | Accepted | product — `knowhub-specs` | [`ADR-017-openapi-is-the-shared-contract.md`](ADR-017-openapi-is-the-shared-contract.md) | — | — |
| ADR-018 | All model access crosses the Model Gateway | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-018-model-gateway-for-all-model-access.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-018-model-gateway-for-all-model-access.md) | — | — |
| ADR-019 | Model routing uses logical profiles | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-019-model-routing-by-logical-profiles.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-019-model-routing-by-logical-profiles.md) | — | — |
| ADR-020 | DSPy is a selective optimization layer | Accepted | backend — `knowhub-backend` | [`docs/adr/ADR-020-dspy-as-selective-optimization-layer.md`](https://github.com/Kibana21/knowhub-backend/blob/main/docs/adr/ADR-020-dspy-as-selective-optimization-layer.md) | — | — |
| ADR-021 | Authentication is provider-pluggable | Accepted | product — `knowhub-specs` | [`ADR-021-provider-pluggable-authentication.md`](ADR-021-provider-pluggable-authentication.md) | — | — |
| ADR-022 | Hybrid RBAC/ABAC authorization with default deny | Accepted | product — `knowhub-specs` | [`ADR-022-hybrid-rbac-abac-authorization-default-deny.md`](ADR-022-hybrid-rbac-abac-authorization-default-deny.md) | — | — |
| ADR-023 | Platform Super Admin is not a content bypass | Accepted | product — `knowhub-specs` | [`ADR-023-super-admin-is-not-a-content-bypass.md`](ADR-023-super-admin-is-not-a-content-bypass.md) | — | — |
| ADR-024 | Premium bright evidence-first UI design language | Accepted | frontend — `knowhub-frontend` | [`docs/adr/ADR-024-premium-bright-evidence-first-ui.md`](https://github.com/Kibana21/knowhub-frontend/blob/main/docs/adr/ADR-024-premium-bright-evidence-first-ui.md) | — | — |
| ADR-025 | Canonical user model is independent of auth provider | Accepted | product — `knowhub-specs` | [`ADR-025-canonical-user-model-independent-of-provider.md`](ADR-025-canonical-user-model-independent-of-provider.md) | — | — |
| ADR-026 | ADR ownership, location, numbering and register | Accepted | product — `knowhub-specs` | [`ADR-026-adr-ownership-and-location.md`](ADR-026-adr-ownership-and-location.md) | — | — |

**26 ADRs** — 14 product-wide, 11 backend, 1 frontend.
Next number to allocate: **ADR-027**.
