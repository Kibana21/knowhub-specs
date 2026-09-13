# ADR-026 — ADR ownership, location, numbering and register

- **Status:** Accepted
- **Scope:** product
- **Date:** 2026-09-13
- **Origin:** this ADR
- **Supersedes:** —
- **Superseded by:** —

## Context

Blueprint [§82](../../knowhub_master_blueprint.html#adr-register) instructs
that ADR files be created *"in the repository that owns the decision"* —
naming `knowhub-backend/docs/adr` for backend platform ADRs and
`knowhub-frontend/docs/adr` for frontend-specific ones — and that its own
table be *"kept synchronized at a summary level"*. [§84.3](../../knowhub_master_blueprint.html#coding-harness-instructions)
lists `docs/adr/` under `knowhub-backend` and, for `knowhub-frontend`, lists
only `docs/ux/`.

Three gaps block that guidance from being followed as written.

1. **Some decisions are owned by neither repository alone.** ADR-016 (the
   two-repository split itself), ADR-017 (OpenAPI as the shared contract),
   ADR-021 (authentication spanning the frontend BFF and the backend session
   model) and ADR-022 (authorization binding both surfaces) have no single
   owning repository. §82 names no home for this case.

2. **The blueprint cannot host a living register.** `knowhub_master_blueprint.html`
   is not tracked by any git repository — it resides in a plain parent
   directory which is itself not a repository. A register kept there cannot be
   diffed, reviewed in a pull request, or relied on as the state of record.
   §82's synchronization instruction assumes the register can be maintained
   where decisions are reviewed; that is not true here.

3. **`knowhub-specs` is absent from the blueprint.** The string
   `knowhub-specs` appears nowhere in the blueprint, so no blueprint section
   assigns it a role. The specification workspace is nonetheless the only
   versioned repository able to see all three repositories, and the three
   repositories are separate GitHub remotes under `Kibana21`.

This ADR resolves all three. It is a governance refinement, deliberately
adopted, not a neutral restatement of §82.

## Decision

### 1. ADR ownership is determined by the scope of the decision

Applied in order, judging the scope of the *decision* rather than where its
implementation or enforcement happens:

1. Scope confined to **backend internals** — persistence, async and
   queueing, model plumbing, backend package boundaries →
   `knowhub-backend/docs/adr/`
2. Scope confined to **frontend internals** — visual language, component
   architecture, routing, client state → `knowhub-frontend/docs/adr/`
3. **Binds both repositories**, defines **product or domain terminology**, or
   states a **platform invariant** every surface must honour →
   `knowhub-specs/02-adrs/`

**Tie-break:** when in doubt, product-wide. A product ADR constrains both
repositories; a repo-local ADR silently fails to bind the other.

A product-wide invariant is a product ADR **even when only the backend can
enforce it**. Repo-local documents may reference a product ADR by number and
link, and explain how they comply — they must never restate its decision
text.

### 2. The live register moves out of the blueprint

This changes §82's register-maintenance convention. Specifically:

- **§82 remains the historical/origin register for ADR-001 through
  ADR-025.** It records that those decisions were made by the blueprint.
- **The live register is [`knowhub-specs/02-adrs/README.md`](README.md).**
- **Future ADRs are registered only in the live register.**
- **Normal development does not edit the blueprint merely to synchronize
  ADRs.**
- **A future formally published blueprint revision may include a snapshot of
  the live register, but the blueprint is not the operational ADR register.**

### 3. One global numbering sequence

ADR numbers are unique across the entire KnowHub workspace regardless of
which repository holds the canonical file. ADR-001 through ADR-025 are
already allocated by §82; new ADRs continue from ADR-026.

The number encodes nothing about location or ownership — only the register
maps number to location. The live register is the allocator: a number is
claimed by adding its register row in the same change that introduces the
ADR. Per-repository sequences and location prefixes are not used.

### 4. Register content is summary-level and link-only

Each row carries: ADR number, title, status, scope and owning repository,
canonical file location, supersedes, superseded-by. Context, decision text,
rationale and consequences live **only** in the canonical ADR file. This is
how §82's "synchronized at a summary level" is honoured.

Exactly one canonical file exists per ADR number, in exactly one location.
No repository holds a copy of another repository's ADR.

Register consistency is checked in `knowhub-specs` only, never as a gate in
the application repositories' CI, because
[§71.3](../../knowhub_master_blueprint.html#package-dependency-rules)
requires each application repository to lint, test and build without
cloning the other.

### 5. Superseding

Reversing a decision requires a superseding ADR explaining evidence,
migration and compatibility impact, written before implementation changes —
as required by §82 and §84.1. Mechanically:

- The superseding ADR takes the next global number and records
  `Supersedes: ADR-XXX`.
- The superseded ADR is never deleted or rewritten. Only its status header
  changes to `Superseded by: ADR-YYY`; its body remains as the historical
  record.
- Files never move. If the new decision's scope differs from the original's,
  the new file lives where its own scope dictates, and the register shows
  both rows with their own locations.
- To supersede one of ADR-001 through ADR-025, write the new ADR normally and
  set the superseded number's register row accordingly. **The blueprint is
  not edited.**
- Changing this convention requires an ADR that supersedes ADR-026 rather
  than an edit to ADR-026.

### 6. This ADR supersedes no ADR

ADR-026 refines a process instruction in §82's prose. It does not reverse any
of ADR-001 through ADR-025, so §84.1's requirement to preserve the ADRs in
§82 is satisfied.

## Consequences

- §82 and the live register **diverge by design** from ADR-026 onward. §82
  must therefore be read as an origin record, which is how both
  [`blueprint-index.md`](../05-reference/blueprint-index.md) and the register
  describe it.
- All of ADR-001 through ADR-025 are extracted to canonical files under this
  convention, so architectural decisions are available as version-controlled
  files without reading the 5,000-line blueprint. Their decision and
  rationale text is reproduced from §82 without addition.
- `knowhub-frontend/docs/adr/` is created. §82 permits it; §84.3's frontend
  listing omits it, which is a gap in a hygiene sketch rather than a
  prohibition. `docs/ux/` retains its §84.3 purpose for decisions below the
  architectural bar.
- Allocating a number requires touching one shared file, so concurrent
  claims collide in review rather than silently producing duplicate numbers.
- `knowhub-specs` holds shared ADRs and the register. It must never become a
  channel for shared source code: §71.3 and §84.4 prohibit a shared
  source-level package between the application repositories, and this ADR
  does not weaken that.
