# M00-T04 — Observability package

- **Status:** Completed — reopened for a telemetry-leakage defect, now corrected and regression-tested

> **Implementation defect (factual note).** The JSON formatter emitted
> ``str(exception)`` as ``error.message``. An exception message is
> caller-influenced and routinely carries the values of the failing call —
> database drivers put connection details into ``OperationalError`` — so a
> secret in a message reached log output. This violated **M00-SPEC-003 R12**,
> which already forbids a credential or connection string in a log record.
> The requirement was correct; the implementation was not. **SPEC-003 is not
> amended.**
>
> **Correction.** Exception text is **deny-by-default**: the formatter emits
> the exception *type* only, never the message, ``args``, attributes or
> traceback. Anything further must be passed explicitly as a structured field
> by a caller who has taken responsibility for its safety. Pattern redaction
> is deliberately **not** used as the primary control — a scrubber must
> anticipate every secret shape, and the ones it misses are the ones that
> matter.
>
> **Regression expectation.** T08 owns the committed proof: a synthetic
> canary, and a second connection-string-shaped canary, must be absent from
> log output and every structured log field, while ``error.type``,
> ``correlation_id``, ``trace_id`` and ``span_id`` remain available.
>
> **Second correction — exported spans.** The framework also records the
> exception onto the span, and the installed instrumentation exposes no
> supported way to disable it (verified against the latest published
> versions). ``observability/span_safety.py`` adds a field-level safety
> boundary at the export seam: an allow-list strips ``exception.message``,
> ``exception.stacktrace`` and any other attribute from an ``exception``
> event, and replaces the status *description*, which
> ``trace.use_span`` builds as ``f"{type(exc).__name__}: {exc}"``. Span
> status code, ``exception.type``, span name, attributes and identifiers are
> preserved. Public SDK API only — no monkey-patching, no fork, no pattern
> redaction. Applied to every configured destination, so console and OTLP
> behave identically. **Resolved.**
- **Depends on:** T01, T03
- **Blocks:** T05, T06
- **Plan:** [M00-PLAN-backend-foundation](../M00-PLAN-backend-foundation.md) §3.6, T4

## Owning specification requirements

M00-SPEC-003 R1–R16 inclusive

M00-SPEC-001 R4, R5, R6, R7 as they apply to `knowhub.observability`.

## Acceptance criteria contributed to

M00-AC-014, M00-AC-015, M00-AC-016, M00-AC-017

## Files created or modified

| Path | Action |
|---|---|
| `src/knowhub/observability/__init__.py` | Create — public surface |
| `src/knowhub/observability/bootstrap.py` | Create |
| `src/knowhub/observability/correlation.py` | Create |
| `src/knowhub/observability/logging.py` | Create |
| `docs/telemetry-naming.md` | Create |

## Implementation steps

1. `bootstrap.py` initialises the tracer provider and meter provider from
   settings, before the application serves. Select the exporter by
   configuration — `otlp` via OTLP HTTP, `console`, or `none`. Provide
   shutdown. Initialisation failure raises; the process must never continue
   silently with telemetry disabled when it is configured active.
2. **No custom metric is defined at M00.** The provider exists so later
   milestones attach meters.
3. `correlation.py` owns a `contextvars` correlation identifier with accept,
   validate, generate and accessor functions. Representation is UUIDv4 in
   canonical 36-character lowercase hyphenated form. Validate in order:
   length at most 36, then the UUID pattern; on any failure discard the
   caller value and generate. Never propagate unvalidated caller input.
4. This module must not import FastAPI — the ASGI middleware lives in
   `knowhub.api` (T05) so a future worker process can reuse the correlation
   context. T06 enforces this as a contract.
5. `logging.py` configures standard-library logging through `dictConfig`: a
   `logging.Filter` on the root handler injecting the correlation identifier
   and the current OTel trace and span IDs, and a JSON `Formatter`. **No
   logging framework dependency.** All logging configuration stays in this
   module.
6. An unhandled error marks its span failed with a safe message carrying the
   correlation identifier.
7. Emit structured fields, never interpolated strings.
8. `docs/telemetry-naming.md` records the single naming convention — OTel
   semantic conventions where one applies, a `knowhub.*` namespace where none
   does — and the bounded-cardinality rule.

## Tests and evidence

Unit tests:

- A valid inbound identifier is honoured; an absent one is generated.
- An over-length identifier is rejected and replaced.
- An identifier containing a control character is rejected and replaced.
- A malformed UUID is rejected and replaced.
- Context survives an `await` boundary.
- Every log record carries the correlation identifier and trace and span IDs.
- No secret, credential, token, connection string or authorization header
  reaches a span attribute, log field or metric label.
- Telemetry initialisation failure raises rather than degrading silently.
- Exporter selection changes by configuration with no code change.

## Failure paths to exercise

The four identifier-rejection cases and the telemetry-initialisation failure.

## Telemetry expectations

This task **is** the telemetry baseline. A span must reach the configured
exporter, and the exporter must be switchable between `otlp`, `console` and
`none` by configuration alone.

## Out of scope

The §36 metric catalogue. Durable run identifiers (M2). Propagation into
workers, connectors, parsers or model calls. `usage.py` and `audit.py`. Any
Azure telemetry SDK. Any alerting, dashboard, SLO or retention policy. Any
FastAPI import in this package.

## Completion checklist

- [x] Initialised at bootstrap, before serving; failure is visible
- [x] Vendor-neutral; no Azure telemetry SDK among dependencies
- [x] Exporter selected by configuration only
- [x] Correlation validated before use; invalid input never propagated
- [x] Identifier on every span and every log record
- [x] Standard-library logging only; configuration isolated to `logging.py`
- [x] No FastAPI import anywhere in this package
- [x] Naming convention documented; no unbounded label or span name
