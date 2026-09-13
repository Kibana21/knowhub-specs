# M00-SPEC-003 — Backend observability baseline

- **Milestone:** M00 (§81 milestone 0A — Backend foundation)
- **Repository:** `knowhub-backend`
- **Status:** Approved
- **§81 0A item owned:** `OTel`

## 1. Purpose

Define the observability foundation the backend establishes at M00:
OpenTelemetry initialisation, vendor-neutral exporter configuration, the
correlation-identifier propagation contract that later milestones must
carry across process boundaries, and the rules that keep telemetry safe.

M00 establishes the contract. The metric catalogue in §36 is populated by
the milestones that deliver the capabilities it measures.

## 2. Sources

**Blueprint**

- [§36 Observability, evaluation and reliability](../../../knowhub_master_blueprint.html#target-observability)
- [§0E.1 Technology decision matrix](../../../knowhub_master_blueprint.html#runtime-architecture)
  — Observability row
- [§84.1 Mandatory working style](../../../knowhub_master_blueprint.html#coding-harness-instructions)
  — instrument all cross-boundary work; correlation/run IDs must flow
- [§76 Environment & deployment matrix](../../../knowhub_master_blueprint.html#environment-matrix)
  — Telemetry row, Local column
- [§65.3 Security, privacy and compliance constraints](../../../knowhub_master_blueprint.html#non-functional-requirements)
  — auditability; data minimization
- [§65.4](../../../knowhub_master_blueprint.html#non-functional-requirements)
  — every asynchronous operation exposes a stable run ID (forward
  constraint)
- [§69.1 Error taxonomy](../../../knowhub_master_blueprint.html#failure-semantics)
  — failures must be visible
- [§61](../../../knowhub_master_blueprint.html#python-file-spec) —
  `observability/tracing.py`, `metrics.py`, `usage.py`, `audit.py`

**ADRs**

- [ADR-001 — KnowHub is Python-first and API-first](../../02-adrs/ADR-001-python-first-api-first.md)
- [ADR-011 — Permission checks precede LLM reasoning](../../02-adrs/ADR-011-permission-checks-precede-llm-reasoning.md)
  and
  [ADR-014 — No unrestricted shell or network tools](../../02-adrs/ADR-014-no-unrestricted-shell-or-network-tools.md)
  — the reason telemetry must not become a side channel for protected
  content; enforcement belongs to the milestones that introduce evidence

## 3. Normative requirements — instrumentation baseline

**R1.** Observability uses **OpenTelemetry** as the instrumentation
standard for traces, metrics and logs.
*(§0E.1 Observability row; §36)*

**R2.** Observability is initialised during process bootstrap, before the
application begins serving, and applies to every process mode the image can
run.
*(§36; §30.1)*

**R3.** The bootstrap is owned by `knowhub.observability` and is invoked by
the process entry point. Application modules obtain instrumentation through
that package; they do not each configure an exporter or provider.
*(§61)*

**R4.** Telemetry initialisation failure must be visible. The process must
not silently continue with telemetry disabled when telemetry is configured
to be active.
*(§69.1; §81.1 — telemetry exists)*

## 4. Normative requirements — vendor neutrality

**R5.** The M00 telemetry contract is **vendor-neutral OpenTelemetry**.
Application code is instrumented with standard OTel APIs only.
*(§0E.1 Observability row; §36)*

**R6.** No Azure-specific telemetry SDK may be a dependency of
`knowhub-backend` at M00, and application code must not be coupled to one
where standard OTel instrumentation is sufficient. §0E.1 names Application
Insights and Log Analytics as the telemetry backend; that is an **export
destination**, not an instrumentation API.

**R7.** The export destination is selected by configuration, per
[M00-SPEC-002](M00-SPEC-002-backend-configuration-and-settings.md) R7–R8
(Platform domain). At M00 the supported Local destinations are OTLP and
console-compatible local output, and telemetry may be configured off for a
test run. Changing destination must require no code change.
*(§76 Telemetry row, Local column; §77)*

## 5. Normative requirements — correlation and propagation

**R8.** Every unit of work carries a **correlation identifier**. An
inbound identifier supplied by the caller is honoured; when absent, one is
generated at the process boundary.
*(§84.1)*

**R9.** An inbound correlation identifier is **validated before use**. A
caller-supplied value is accepted only when it conforms to a documented
accepted representation, is within a documented bounded maximum length, and
contains no control characters or other content unsafe to place in a span
attribute or log record. An absent or non-conforming identifier results in a
new server-generated identifier; a caller-controlled value is never
propagated into telemetry unvalidated.
*(§69.1 — a malformed request input is a validation error, failed
immediately; §36 — log metadata safely; §84.1; R15 for the equivalent
cardinality constraint)*

*Implementation guidance (non-normative): a UUID representation satisfies
every clause of R9 and is the obvious default, but no blueprint section or
ADR requires a specific identifier format or library, so the concrete
representation and its generator are an implementation-plan choice. Where a
standard propagation format such as W3C Trace Context already carries the
identifier, its own validation rules apply and need not be duplicated.*

**R10.** The correlation identifier appears on every span and every log
record emitted while handling that unit of work, and is returned to the
caller so a client-observed request can be located in telemetry.
*(§84.1; §65.4)*

**R11.** The propagation contract is defined at M00 so that later
milestones extend it rather than redesign it. §84.1 requires correlation
and run identifiers to flow API → Taskiq → connector/parser/LLM/DB. At M00
only the API boundary exists, so M00 specifies:

- how an identifier is accepted, generated and represented;
- that it is carried in the ambient request context rather than threaded
  manually through call signatures;
- that context propagates across `await` boundaries within a process;
- that any future process hop must carry the identifier explicitly in the
  message or call it dispatches.

The durable **run** identifier of §65.4 and the glossary's `Run` concept is
distinct from the correlation identifier and is introduced with the run
model in M2.
*(§84.1; §65.4;
[glossary](../../05-reference/glossary.md))*

## 6. Normative requirements — safe telemetry

**R12.** Telemetry records metadata, never protected content. No secret,
credential, token, connection string, authorization header or resolved
configuration secret may appear in a span attribute, log record, metric
label or error message.
*(§65.3; §36 — log metadata safely without indiscriminately logging
protected evidence;
[M00-SPEC-002](M00-SPEC-002-backend-configuration-and-settings.md) R17)*

**R13.** Attribute and metric naming follows a single documented
convention, applied from M00, so that later milestones' telemetry is
queryable alongside the foundation's rather than diverging per package.
*(§36)*

**R14.** An unhandled error records its span as failed with a safe message
and the correlation identifier, sufficient to locate the failure without
disclosing protected content.
*(§69.1; §65.3)*

**R15.** Cardinality is bounded: an unbounded or caller-controlled value
must not be used as a metric label or span name.
*(§36 — the metric set is defined per area and must remain queryable as
such)*

*Implementation guidance (non-normative): the naming convention should
follow OpenTelemetry semantic conventions where one applies, and reserve a
KnowHub-specific namespace for attributes that have no standard
equivalent. The concrete convention document belongs with the
implementation plan.*

**R16.** Telemetry is not an audit log. §61 and §65.3 place auditability in
`observability/audit.py` as a separate concern with its own durability and
access rules; M00 introduces no privileged action to audit and therefore
implements no audit event.
*(§61; §65.3)*

## 7. Acceptance criteria

| ID | Criterion |
|---|---|
| **M00-AC-014** | OpenTelemetry is initialised during process bootstrap; a request to the foundation liveness endpoint produces a trace through the configured exporter, and a telemetry initialisation failure is visible rather than silent. *(R1, R2, R4)* |
| **M00-AC-015** | An inbound correlation identifier is honoured and one is generated when absent; it appears on every span and log record for that request, is returned to the caller, and propagates across `await` boundaries without being threaded through call signatures. A caller-supplied identifier that exceeds the documented length bound, contains a control character or otherwise fails the documented representation is rejected and replaced by a server-generated identifier rather than propagated. *(R8, R9, R10, R11)* |
| **M00-AC-016** | Instrumentation is vendor-neutral: no Azure-specific telemetry SDK is a dependency, and the export destination — OTLP, console or off — changes by configuration alone with no code change. *(R5, R6, R7)* |
| **M00-AC-017** | No secret, credential, token, connection string or authorization header appears in any span attribute, log record, metric label or error message; an unhandled error marks its span failed with a safe message carrying the correlation identifier; no metric label or span name uses an unbounded caller-controlled value. *(R12, R14, R15)* |

## 8. Deferred

| Deferred from M00 | Owning milestone |
|---|---|
| The §36 metric catalogue — ingestion, graph, Foundry research, artifacts, Q&A, lifecycle, cost and operations metrics | Each owning milestone |
| Durable run identifiers and run-scoped telemetry | M2 |
| Trace propagation into Taskiq workers and the scheduler | M2 |
| Trace propagation into connectors, parsers, LangGraph nodes and model calls | M3–M6 |
| Model usage, token and cost recording (`observability/usage.py`) | M6+ |
| Audit events (`observability/audit.py`) and their retention/access policy | M1 / M12 |
| Application Insights, Log Analytics or any other deployed telemetry backend | Deployed-environment work |
| MLflow, evaluation datasets and nightly regression reporting | M6+ |
| Alerting, dashboards, SLOs and retention policy | Deployed-environment work |
