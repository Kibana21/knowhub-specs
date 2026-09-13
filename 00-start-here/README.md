# KnowHub Specification Workspace

`knowhub-specs` is the specification workspace for KnowHub, an enterprise
application knowledge and software-intelligence platform. It holds the
architecture documents, Architectural Decision Records, capability
specifications and implementation plans that are derived from the master
blueprint.

This repository contains no application code. It is the layer where
product intent is turned into reviewable, approved specifications before
anything is implemented.

## Master blueprint

- **`../../knowhub_master_blueprint.html`** — the master product and
  architecture blueprint, and the primary source of product intent and
  target architecture.

It is a single HTML file of 97 top-level sections (§0–§85) with its own
in-page table of contents and full-text search box. It lives in the
surrounding workspace, not inside this repository; paths above are
relative to this file. Use
[`../05-reference/blueprint-index.md`](../05-reference/blueprint-index.md)
to find the right section by topic.

## Implementation repositories

- **`../../knowhub-backend/`** — production Python/FastAPI backend.
  Currently contains only `README.md`; no implementation yet.
- **`../../knowhub-frontend/`** — production Next.js/TypeScript frontend.
  Currently contains only `README.md`; no implementation yet.

## Read-only reference repositories

- **`../../terrain-main/`** — behavioral reference.
- **`../../deepwiki-rs-main/`** — behavioral reference.

Both are **read-only**. Never modify either repository.

## Specification-driven development lifecycle

KnowHub is developed specification-first. Work moves through:

```
Blueprint
  → Architecture / ADR
    → Capability Spec
      → Implementation Plan
        → Tasks
          → Code
            → Tests / Review
```

Two rules govern how this workspace is populated:

- Do not jump directly from the blueprint to large-scale implementation.
- Create capability specifications **incrementally**, according to the
  implementation sequence in the blueprint. Do not generate all future
  specifications in advance.

## Directory layout

| Directory | Contents |
|---|---|
| `00-start-here/` | Entry point: this README and the delivery roadmap. |
| `01-architecture/` | Architecture documents. Empty — not yet created. |
| `02-adrs/` | Architectural Decision Records. Empty — not yet created. |
| `03-specs/00-foundation/` | Capability specifications, grouped by delivery stage. Empty — not yet created. |
| `04-plans/current/` | Implementation plans for work in progress. Empty. |
| `04-plans/completed/` | Implementation plans for delivered work. Empty. |
| `05-reference/` | Navigation and reference material: blueprint index, glossary, requirement map. |

`05-reference/requirement-map.md` exists but is not yet populated.

## Where to go next

| Document | Purpose |
|---|---|
| [`delivery-roadmap.md`](delivery-roadmap.md) | The authoritative build order and milestone sequence. Read this before starting any new specification. |
| [`../05-reference/blueprint-index.md`](../05-reference/blueprint-index.md) | Topic-grouped navigation index for all 97 blueprint sections, with direct anchor links. |
| [`../05-reference/glossary.md`](../05-reference/glossary.md) | The normative terminology contract. Use these terms with these meanings in every specification, ADR and commit. |
