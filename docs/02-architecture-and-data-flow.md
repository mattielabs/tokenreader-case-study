# 02 — Architecture and data flow

[← Back to the case study](../README.md)

![Local request flow: application SDK to local gateway to deterministic fixture upstream, with a separate evidence path through normalization, cost estimation, local SQLite, a typed API, and the dashboard](../assets/diagrams/01-local-request-flow.svg)

## The two paths

The design separates the **request path** from the **evidence path**, and that separation is what
makes the privacy guarantee tractable.

The request path is a pass-through. The application's official SDK is pointed at the local gateway,
which forwards the native request to its configured upstream and streams the native response back
unmodified. TokenOps does not translate OpenAI Responses and Anthropic Messages into a shared
envelope; each keeps its own request shape, response shape, and SSE protocol.

The evidence path branches off it. From the same in-flight request, the gateway derives a small set
of allowlisted metadata fields, normalizes provider-reported usage, matches a price schedule,
computes an estimate, and commits the record. Nothing on the evidence path can reach back and change
what the application receives.

## Gateway boundaries

SDK input is treated as untrusted. On every request the gateway:

- assigns its own trace identifier rather than trusting a client-supplied one;
- strips credentials and internal headers so they stop at the boundary and never reach the upstream
  or any log;
- rejects any attempt by the client to select an upstream — the upstream is configuration, not input;
- validates the `Host` header, and additionally validates browser `Origin` on management traffic;
- enforces request-size limits, with a smaller limit for Prompt Trim, returning a deterministic 413;
- uses **zero retries**, so one application request is one upstream attempt and one billable event.

Services bind to the loopback interface. Management responses carry a Content Security Policy, frame
denial, no-referrer, and MIME-sniffing protection, and CORS is never wildcarded. These reduce
browser-driven attack surface; they do not defend a compromised machine.

## Streaming and finalization

Streaming was the part most likely to quietly corrupt the evidence, so it got explicit rules.

Each received chunk is yielded immediately rather than buffered to completion, so the application
sees a real stream. The parser retains only an incomplete SSE frame plus allowlisted telemetry.
Content deltas pass through and are **never accumulated** — there is no point at which an assembled
response body exists in the process.

Final usage is read from the documented terminal signal for each provider: OpenAI's completion event
carries the final response usage, while Anthropic supplies input and cache figures at message start
and a cumulative output figure that is retained at the end. An interrupted stream keeps only the
valid numeric usage already observed and is classified honestly as partial. No usage is invented to
fill a gap.

Final states are distinguished rather than collapsed into "error": completed, upstream failure,
timed out, cancelled, client disconnected, and interrupted are separate outcomes because they mean
different things when you are reading cost evidence.

## Persistence

Request, normalized usage, price match, and cost state commit **once, transactionally**. A partially
written record is not a state the dashboard can observe.

Schema evolution is handled by versioned migrations that are tested for upgrade, downgrade, and
re-upgrade, including preservation of data written by earlier versions. The MVP uses SQLite with a
documented path to PostgreSQL; nothing in the design depends on SQLite-specific behaviour.

## API contract and the dashboard

The management API contract is **generated from the implementation and drift-checked in CI-style
validation**, then used to generate the TypeScript client types. If the backend and the frontend
disagree, validation fails rather than the mismatch surfacing as a runtime bug.

Business calculations are not duplicated in the browser. The dashboard renders values the backend
computed, which is what allows the connected dashboard and the static demo to share components
without the risk of two divergent implementations of the cost logic.

## Architecture decisions on record

The project keeps numbered architecture decision records. The ones that most shaped the result:

| Decision | Rationale |
|---|---|
| A local gateway rather than importing provider usage exports | Per-request attribution is impossible from an aggregated export after the fact |
| Native provider routes rather than a unified abstraction | An abstraction would have to guess at semantics that differ between providers, and would break as APIs move |
| Deterministic fixtures first | Validation must be reproducible and free; a test suite that needs a paid key is a test suite that stops being run |
| Direct runtime as primary, Docker optional | The supported path should be the one that is actually exercised |
| Exact versioned pricing | Approximate matching produces confidently wrong numbers, which is worse than an honest gap |
| Explicit stream finalization | Usage semantics differ per provider and must be read from the documented terminal signal |
| Advisory budgets | Enforcement in the request path would make TokenOps a point of failure in someone's production system |
| Keyed fingerprints with disclosed leakage | Detecting repetition without storing text is achievable; pretending it leaks nothing is not |
| Deterministic opt-in Prompt Trim | A model-driven rewriter cannot be validated deterministically or safely automated |
| Metadata-only as a permanent product boundary | Making it a boundary rather than a default removes the pressure to relax it later |

## Deliberately excluded

Content capture, retention, and purge; authentication; multi-user or remote access; automatic prompt
modification; budget enforcement; retries; routing; model rankings or judges; remote databases;
workers and queues; deployment; and Docker troubleshooting.

---

Previous: [01 — Product overview](01-product-overview.md) · Next: [03 — Cost traceability](03-cost-traceability.md)
