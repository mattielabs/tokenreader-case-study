# 04 — Budgets and findings

[← Back to the case study](../README.md)

## Advisory budgets

A project carries one positive monthly limit, stored in integer micro-USD. The active period is the
half-open UTC calendar month — from the first day at 00:00 up to, but not including, the next first
day.

Spend counts **only** cost estimates with status `complete`. Incomplete and unpriced requests are
reported separately and are never quietly converted to zero, because a budget that silently
undercounts is worse than no budget.

States are: below 50%, warning at 50%, high warning at 80%, and reached or exceeded at 100%.

The important property is what a budget **cannot** do. It never blocks, delays, reroutes, retries, or
modifies a provider request. Changing a limit does not recalculate historical costs. A budget is a
number to look at, not a control in the request path — putting enforcement there would make TokenOps
a new point of failure inside someone else's production system, for the sake of a guardrail the
provider's own limits already offer.

The demo snapshot exercises all three interesting states so the interface can be reviewed in each:
one project at 80% (`warning_80`, $0.014696 of a $0.018370 limit, with 3 incomplete requests reported
alongside), one at 50%, and one at exactly 100%.

## Findings

Six bounded findings run over stored metadata. Each is versioned, linked to a request, idempotent,
labelled as fixture evidence, and states three things: **what was measured**, **why it might matter**,
and **what to verify**.

| Finding | Default threshold | What it measures | Limitation stated in the product |
|---|---:|---|---|
| Repeated static context | 2 equal blocks | Keyed-HMAC equality across requests | Equality, **not** semantic similarity |
| Cache opportunity | repeated block ≥ 400 characters | Size of a repeated block | Requires a provider cache-eligibility review |
| High input-to-output ratio | 4.000 | Ratio of input to output tokens | Length alone is not waste |
| Long instruction block | 1,200 characters | Instruction block size | Length alone is not waste; policy text should be preserved |
| Repeated failure spend | 2 failures | Count of repeated failures | Spend stays uncalculated when the evidence is unavailable |
| Oversized output | 800 tokens | Output length | Crosses an inspection threshold only |

Thresholds are bounded per-project fields, so a finding is always reported against the specific
threshold that produced it rather than an invisible constant.

### The savings claim that is deliberately absent

Until there is a concrete prompt comparison **and** an exact active schedule, estimated savings is
recorded as `potential_reduction_not_calculated`.

This is the single most tempting place in the product to invent a number. "You could save $X per
month" is the sentence a cost tool is expected to produce, and it would have been easy to generate
from a threshold crossing. It is not generated, because a measured 840-character repeated block is
evidence that something is worth checking — not evidence that removing it would be safe, that the
provider would cache it, or that any money would actually be saved.

### Fingerprints and reset

Repeated-context and cache findings are computed from database counts over keyed HMACs, never over
stored text. Disabling fingerprinting for a project stops new HMACs from being written; existing
equality rows remain until an explicit reset.

A confirmed reset generates a new key, removes all HMAC rows and equality-derived findings, and
leaves requests, usage, pricing, costs, projects, and budgets intact — so a privacy action does not
also destroy the cost history the user came for.

![Optimize view: findings, Prompt Trim, and the offline evaluation summary](../assets/screenshots/demo-evaluation.jpg)

---

Previous: [03 — Cost traceability](03-cost-traceability.md) · Next: [05 — Prompt Trim and evaluation](05-prompt-trim-and-evaluation.md)
