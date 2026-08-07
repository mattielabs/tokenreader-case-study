# 08 — Static recruiter demo

[← Back to the case study](../README.md)

![Static demo boundary: a checked synthetic snapshot feeds a data-source adapter into the same React components, producing a read-only browser experience with no gateway, database, credentials, or network request](../assets/diagrams/03-static-demo-boundary.svg)

## The problem it solves

A reviewer should be able to see the actual interface without installing anything, without a
provider key, and without the project owner exposing production data — because there is no production
data, and there never will be under the metadata-only boundary.

The usual answers are bad. A recorded video cannot be explored. A separate marketing mockup drifts
from the real product and quietly becomes a lie. A hosted live instance needs a backend, a database,
credentials, and a deployment the project has explicitly not authorised.

## The approach

Generate a deterministic synthetic snapshot, check it into the repository, and feed it to the **same
React components** the connected dashboard uses. Only the client data adapter changes.

Because business calculations live in the backend and the frontend renders what it is given, the
static build cannot accidentally implement its own version of the cost logic. What a reviewer sees is
the real presentation layer over real API shapes — a genuine artifact rather than an approximation.

## What the snapshot contains

Requests and request detail, normalized usage, exact pricing and cost state, advisory budgets,
findings, privacy state, the offline evaluation summary, and exactly one allowlisted synthetic Prompt
Trim example.

It is built to exercise the states that matter for review rather than a flattering happy path: eight
requests across both providers, five with complete cost estimates and three deliberately incomplete
(one unknown price, two with missing usage), one upstream failure, two streaming and six
non-streaming, and three budgets sitting at 50%, 80%, and 100%.

Showing incomplete and failed states in the demo is the point. A cost tool that only ever displays
clean data has not demonstrated the hard part.

## Read-only by construction

The demo has no gateway, no database, no credentials, no analytics, no browser-storage writes, no
provider call, and no automatic external request.

- Every screen carries a synthetic read-only banner.
- Budget, project, pricing, fingerprint, and comparison mutations are absent or disabled with
  explicit read-only text.
- Exports are unavailable.
- Routes use the URL fragment, so a direct refresh works on a basic static server without a rewrite
  rule.
- Loaded assets stay local; the bundle is scanned to confirm no external request.

## Verification

The snapshot generator has a `--check` mode that detects drift, and a separate scan verifies mode
flags, the 30-ticket evaluation state, uniqueness of the approved Prompt Trim example, paths and
secrets, and the built bundle. A missing or stale snapshot fails the build workflow rather than
silently shipping old numbers.

Build output is ignored by version control and is explicitly **not** publication evidence.

## Where it is not

The demo runs locally. **It is not deployed or hosted anywhere**, and this case study does not link
to a live instance. The screenshots in this repository are the published artifact.

Pricing provenance recorded in the snapshot is rendered as plain text in static mode rather than as
navigation, so the read-only build has nothing to click out to.

---

Previous: [07 — Testing and validation](07-testing-and-validation.md) · Next: [09 — AI-assisted development](09-ai-assisted-development.md)
