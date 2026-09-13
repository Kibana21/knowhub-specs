# KnowHub Delivery Roadmap

**Source:** master blueprint
[§81 Implementation dependency graph & execution order](../../knowhub_master_blueprint.html#implementation-dependency-graph)
and §81.1 Milestone exit rule.

§81 is the authoritative implementation order for KnowHub. §81.1 gates
every transition between milestones. This document preserves that
sequence and its dependencies; it does not decompose milestones into
implementation tasks.

The blueprint's stated intent for this order: *build in dependency order
so every milestone produces working capability and testable contracts.*

---

## Dependency graph

```
0A. Backend foundation — knowhub-backend
    pyproject/uv, Python package boundaries, CI, settings, OTel,
    local dependencies
          │
          ├───────────────────────────────────────────────────────┐
          ▼                                                       │
0B. Frontend foundation — knowhub-frontend                        │
    Next.js/TypeScript, bright design system,                     │
    LOCAL/OIDC BFF session shell, CI                              │
    + pinned OpenAPI generation pipeline                          │
          │                                                       │
          └──────── waits on/stubs versioned API contract ────────┘
          │
          ▼
1.  Backend Identity + Application Registry + canonical DB model
          │
          ▼
2.  Run model + Taskiq/Redis + Blob abstraction
          │
          ▼
3.  Git connector + SourceRevision/Evidence model
          │
          ▼
4.  Scanner + Tree-sitter + source pack + exact source tools
          │
          ├────────────► 5. Postgres entity/relationship graph
          │
          ▼
6.  Knowledge Foundry core
    preprocess → research → compose → validate
          │
          ▼
7.  Agent context + artifacts + freshness/baselines
          │
          ▼
8.  Retrieval + Ask + citation validation
          │
          ▼
9.  Cross-repo web vertical slice
    Backend APIs stable + frontend
    Application → Sources → Ask/Evidence → Architecture
          │
          ▼
10. Jira + document upload/structured parsing
          │
          ▼
11. Traceability + impact + incremental multi-source refresh
          │
          ▼
12. Enterprise ACL/classification/governance hardening
          │
          ▼
13. Confluence/SharePoint + additional connectors
          │
          ▼
14. Scale/search/graph/runtime backlog
    only when measured need exists
```

---

## Milestones

| ID | Milestone | Scope as stated in §81 | Depends on | Status |
|---|---|---|---|---|
| **0A** | Backend foundation — `knowhub-backend` | pyproject/uv, Python package boundaries, CI, settings, OTel, local dependencies | — | Not started |
| **0B** | Frontend foundation — `knowhub-frontend` | Next.js/TypeScript, bright design system, LOCAL/OIDC BFF session shell, CI, + pinned OpenAPI generation pipeline | 0A (waits on/stubs versioned API contract) | Not started |
| **1** | Backend Identity + Application Registry + canonical DB model | — | 0A | Not started |
| **2** | Run model + Taskiq/Redis + Blob abstraction | — | 1 | Not started |
| **3** | Git connector + SourceRevision/Evidence model | — | 2 | Not started |
| **4** | Scanner + Tree-sitter + source pack + exact source tools | — | 3 | Not started |
| **5** | Postgres entity/relationship graph | — | 4 (branch) | Not started |
| **6** | Knowledge Foundry core | preprocess → research → compose → validate | 4 | Not started |
| **7** | Agent context + artifacts + freshness/baselines | — | 6 | Not started |
| **8** | Retrieval + Ask + citation validation | — | 7 | Not started |
| **9** | Cross-repo web vertical slice | Backend APIs stable + frontend Application → Sources → Ask/Evidence → Architecture | 8, 0B | Not started |
| **10** | Jira + document upload/structured parsing | — | 9 | Not started |
| **11** | Traceability + impact + incremental multi-source refresh | — | 10 | Not started |
| **12** | Enterprise ACL/classification/governance hardening | — | 11 | Not started |
| **13** | Confluence/SharePoint + additional connectors | — | 12 | Not started |
| **14** | Scale/search/graph/runtime backlog | Only when measured need exists | 13 | Not started |

A dash in the scope column means §81 states the milestone by its title
only. No further scope has been added here.

---

## Parallelization rule (§81)

- Frontend work may proceed from mocked/generated OpenAPI contracts as
  soon as a backend endpoint contract is stable.
- Integration acceptance is complete only when the real independently
  deployed frontend and backend pass the same end-to-end test.
- Backend milestones must never depend on React code to execute.

---

## Milestone exit rule (§81.1)

Do not progress because files or classes exist. Each milestone exits only
when:

- its golden acceptance tests pass,
- telemetry exists,
- migrations are repeatable,
- failure paths are exercised, and
- the preceding public contracts remain stable.

---

## Scope and horizon context

These sections provide scope context and are **not** merged into the §81
milestone sequence:

- [§80 Scope boundary — MVP, Enterprise V1 & future](../../knowhub_master_blueprint.html#scope-boundary)
  — per-capability MVP / Enterprise V1 / Future columns.
- [§85 Future enterprise backlog & product readiness](../../knowhub_master_blueprint.html#future-enterprise-backlog)
  — deliberately deferred backlog and product-readiness horizons.

## Related sequencing views — not authoritative here

The blueprint contains several other views of build order. They are listed
for reference only. This document does not reconcile them with §81, and no
content from them has been incorporated above.

- [§0G.1 First useful vertical slice](../../knowhub_master_blueprint.html#implementation-boundaries)
- [§39 Build roadmap — from useful clone to enterprise platform](../../knowhub_master_blueprint.html#target-roadmap)
- [§42 What I would build first — concrete 12-week engineering sequence](../../knowhub_master_blueprint.html#first-build)
- [§63.5 Recommended implementation gates](../../knowhub_master_blueprint.html#porting-dod)
