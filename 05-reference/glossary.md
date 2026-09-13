# KnowHub Glossary & Terminology Contract

Markdown extraction of master blueprint
[§83 Glossary & terminology contract](../../knowhub_master_blueprint.html#glossary).

These meanings are normative. Where this file and §83 differ, §83 wins.

| Term | Normative meaning |
|---|---|
| **Application** | The enterprise product/system scope that groups repositories, delivery records, documents, interfaces, data and other sources. |
| **Source** | A configured external or uploaded knowledge origin such as Git repository, Jira project, SharePoint library or file upload. |
| **Source Revision** | A versioned snapshot/change identity from a source: Git SHA, Jira change cursor/object version, document version/hash, etc. |
| **Evidence** | Source-grounded information with revision, locator, provenance and effective authorization. |
| **Evidence Location** | Precise address inside evidence: file/lines/symbol, page/heading/paragraph, Jira field/comment, workbook/sheet/range, etc. |
| **Entity** | A canonical thing KnowHub can identify and relate: application, service, file, symbol, API, requirement, issue, table, control. |
| **Relationship** | A typed, evidence-backed link between canonical entities. |
| **Knowledge Claim** | A queryable assertion about an entity/value, marked observed, inferred or approved, with provenance and authority. |
| **Authority** | How a claim/source should be interpreted: normative/approved, observed deterministic, delivery/operational, collaborative or inferred. |
| **Artifact** | A generated or curated knowledge product such as architecture document, workflow page, compact context or report. |
| **Knowledge Foundry** | KnowHub's preprocessing → specialist research → composition → validation subsystem derived conceptually from the upstream architecture-documentation engine. |
| **Agent Context** | Compact machine-oriented application map designed for retrieval/onboarding by AI agents; not the same as human docs. |
| **Freshness** | How trustworthy an artifact/evidence projection is relative to source/content/permission/version drift. |
| **Baseline** | The source revision/cursor/version against which freshness and incremental changes are evaluated. |
| **ChangeSet** | Normalized evidence describing what changed between baselines and what knowledge may be affected. |
| **Traceability** | Evidence-backed path across intent/delivery/implementation/testing or other lifecycle stages. |
| **Impact** | Entities/workflows/requirements/tests that are definitely or possibly affected by a proposed/observed change. |
| **Projection** | Rebuildable index/store created from canonical data, e.g. vector/search or Neo4j graph. |
| **Run** | A durable execution record for synchronization, research, indexing, workflow or evaluation independent of worker process state. |
| **Steward** | Authorized human responsible for reviewing/correcting/approving knowledge within a defined scope. |
| **Model Gateway** | The provider-neutral backend boundary that normalizes generation, streaming, tool calls, structured output, embeddings, usage and policy-aware routing across cloud and local models. |
| **Model Profile** | A logical role such as FAST, REASONING, JUDGE, VISION, EMBEDDING or LOCAL_PRIVATE that resolves to an approved endpoint/model and required capabilities under enterprise policy. |
| **AI Program** | A typed model-assisted task contract used by KnowHub. It may be implemented as a conventional typed prompt module or a DSPy program; domain code should not depend directly on the framework. |
