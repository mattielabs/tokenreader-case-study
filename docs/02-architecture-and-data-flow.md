# 02 — Architecture and data flow

[← Back to the case study](../README.md)

![Local request flow: application SDK to local gateway to deterministic fixture upstream, with a separate evidence path through normalization, cost estimation, local SQLite, a typed API, and the dashboard](../assets/diagrams/01-local-request-flow.svg)

## The two paths

The design separates the **request path** from the **evidence path**, and that separation is what
makes the privacy guarantee tractable.

The request path is a pass-through. The application's official SDK is pointed at the local gateway,
which forwards the native request to its configured upstream and streams the native response back
unmodified. TokenReader does not translate OpenAI Responses and Anthropic Messages into a shared
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
- rejects any attempt by the client to select an upstream. The upstream is configuration, not input;
- validates the `Host` header, and additionally validates browser `Origin` on management traffic;
- enforces request-size limits, with a smaller limit for Prompt Trim, returning a deterministic 413;
- uses **zero retries**, so one application request is one upstream attempt and one billable event.

Services bind to the loopback interface. Management responses carry a Content Security Policy, frame
denial, no-referrer, and MIME-sniffing protection, and CORS is never wildcarded. These reduce
browser-driven attack surface; they do not defend a compromised machine.

In the desktop application the same gateway runs as a supervised child process with two more
boundaries: a per-launch session secret required on management routes, and a persistent local SDK
token required on the provider-shaped routes. The [desktop document](11-windows-desktop-application.md)
covers that layer.

## Streaming and finalization

Streaming was the part most likely to quietly corrupt the evidence, so it got explicit rules.

Each received chunk is yielded immediately rather than buffered to completion, so the application
sees a real stream. The parser retains only an incomplete SSE frame plus allowlisted telemetry.
Content deltas pass through and are **never accumulated**. There is no point at which an assembled
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
re-upgrade, including preservation of data written by earlier versions. The migration head is
`0005_privacy_operations`. The revision identifiers still carry the original product name, and they
were kept at the rename because changing them would have invalidated history for every existing
database. SQLite is the store, with a documented path to PostgreSQL that nothing in the design
depends on.

## API contract and the dashboard

The management API contract is **generated from the implementation and drift-checked**, then used
to generate the TypeScript client types. If the backend and the frontend disagree, validation fails
rather than the mismatch surfacing as a runtime bug.

Business calculations are not duplicated in the browser. The dashboard renders values the backend
computed. One transport interface has three implementations: HTTP for browser development, a
snapshot adapter for the static demo, and the preload bridge for the desktop shell. That is what
lets all three modes share every view without the risk of divergent cost logic.

## Architecture decisions on record

The project keeps numbered architecture decision records, twenty-two at the time of writing. The
ones that most shaped the result:

| Decision | Rationale |
|---|---|
| A local gateway rather than importing provider usage exports | Per-request attribution is impossible from an aggregated export after the fact |
| Native provider routes rather than a unified abstraction | An abstraction would have to guess at semantics that differ between providers, and would break as APIs move |
| Deterministic fixtures first | Validation must be reproducible and free; a test suite that needs a paid key is a test suite that stops being run |
| Direct runtime as primary, Docker optional | The supported path should be the one that is actually exercised |
| Exact versioned pricing | Approximate matching produces confidently wrong numbers, which is worse than an honest gap |
| Explicit stream finalization | Usage semantics differ per provider and must be read from the documented terminal signal |
| Advisory budgets | Enforcement in the request path would make TokenReader a point of failure in someone's production system |
| Keyed fingerprints with disclosed leakage | Detecting repetition without storing text is achievable; pretending it leaks nothing is not |
| Deterministic opt-in Prompt Trim | A model-driven rewriter cannot be validated deterministically or safely automated |
| Metadata-only as a permanent product boundary | Making it a boundary rather than a default removes the pressure to relax it later |
| Commercial data and settings contracts before implementation | Every data class got a lifecycle on paper before any desktop code existed, so the code had to satisfy the contract rather than define it |
| Fail-closed desktop lifecycle and offline trust boundaries | Credentials only in the OS vault, updates only after consent, licences validated locally, no placeholder that reports success |
| Windows credentials and the local provider boundary | Keys never reach the renderer or disk; live forwarding goes to exactly two endpoints, per provider, after explicit confirmation |
| Recovery, retention, and privacy operations | Backup, staged restore and upgrade, purge, receipts, and diagnostics under one lifecycle lock |
| Product shell, onboarding, and installer contracts | Seven destinations, two equal first-launch paths, nothing external on first launch, installer behaviour written as testable contracts |
| Fast process exit on shutdown | A native module crashed on DLL detach only when launched without a console; the accepted fix and the three rejected ones are on record |
| Two-theme AA-gated token system | The design handoff was implemented in place, with sub-AA shades corrected and the "persist the theme" note overruled by the no-browser-storage invariant |
| Portable signed offline licences with separated trust roots | Lifetime, no binding, dual validation, nothing local ever locked |
| Signed update manifests, opt-in checks, exact distribution | A manifest the product signs is the authority; the third-party updater's feed must agree or the release is refused |
| Signing-ready pipeline and inspected release bundles | Every shipped binary is enumerated; release mode fails closed on anything unsigned; the fail-closed gate is itself tested |
| TokenReader product identity and migration | One controlled rename with a data-root and credential migration, preserved stable identifiers, and a branding gate |

## Deliberately excluded

Content capture, retention of content, and content purge; user accounts and multi-user or remote
access; automatic prompt modification; budget enforcement; retries; routing; model rankings or
judges; remote databases; workers and queues; telemetry; deployment; and Docker troubleshooting.

---

Previous: [01 — Product overview](01-product-overview.md) · Next: [03 — Cost traceability](03-cost-traceability.md)
