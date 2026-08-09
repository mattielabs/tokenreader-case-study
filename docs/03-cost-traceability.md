# 03 — Cost traceability

[← Back to the case study](../README.md)

The goal of this subsystem is narrow and testable: **any number on the Overview must be walkable back
to a specific request, a specific token count, and a specific dated rate** — or else be reported as
unavailable with a reason.

## Step 1 — Normalize provider usage

Providers report usage differently, and the differences are billable. Normalization maps each
provider's documented fields into shared fields **without flattening the categories that price
differently**: uncached input, cached input reads, cache writes (including Anthropic's separate
5-minute and 1-hour tiers), output, and reasoning.

Two rules prevent the most common double-counting errors:

- **Reasoning tokens are already inside output** for both providers. TokenOps records the detail but
  never adds it to output or total again.
- **Cached input is a detail of the reported input total**, not a separate addition to it.

There is a third rule that is a judgment call rather than a mapping: Anthropic documents total input
as the sum of uncached input, cache creation, and cache reads, but does not report that sum as a
total. TokenOps **does not** synthesise it into the total field, because a derived figure sitting in
a field named "reported total" is exactly the kind of quiet fiction the project is trying to avoid.

Usage is then classified: `complete` when all core fields are valid, `partial` when a usage object
exists but core values are missing, `missing` when there is no usage object at all, and `invalid`
when a value is negative, malformed, or internally inconsistent. **Missing stays NULL. A reported
zero stays zero.** No token count is ever estimated from text.

## Step 2 — Match an exact price schedule

Prices are small, dated schedules stored in the database, not scraped at runtime. The seeded
schedules were checked against the providers' published pricing pages on 2026-08-07.

Matching requires provider, **exact** returned model (falling back to exact requested model),
processing mode, UTC request date, and context band. Prefix, substring, family, and nearest-model
matching are **forbidden**. Aliases require an explicit registry row.

- Zero matches → `price_unavailable`
- Multiple valid matches → `ambiguous_price`

Both are visible states in the interface rather than silent fallbacks to a "close enough" rate. A
historical estimate retains a foreign key to the exact schedule that produced it, so changing prices
later does not silently rewrite the past.

One consequence worth calling out: OpenAI's verified schedule has no cache-write rate, so TokenOps
cannot invent one. The category simply cannot be billed rather than being defaulted to zero or to a
neighbouring provider's rate.

## Step 3 — Calculate

```text
component USD        = token count × USD per million / 1,000,000
component micro-USD  = round-half-up(component USD × 1,000,000)
total micro-USD      = exact sum of the stored component micro-USD values
```

All arithmetic uses decimal types — never floating point. Components are rounded **independently**
and the total is their exact sum, so the reconciliation shown in the interface adds up precisely
rather than approximately.

Cost status is one of `complete`, `partial`, `price_unavailable`, `usage_unavailable`,
`ambiguous_price`, `invalid_usage`, `not_applicable`, or `not_calculated`.

## The worked trace

![Light request detail view: a four-step reconciliation path followed by safe metadata, normalized usage, estimated-cost components, and exact schedule provenance](../assets/screenshots/demo-request-trace.jpg)

The Overview reports **$0.014696** of complete estimated spend, from
`complete_estimated_cost_micro_usd = 14696` in the checked snapshot. That figure covers 5 complete
requests out of 8; the other 3 are excluded and reported separately as incomplete or unpriced.

Request `trc_demo_01_…` contributes 1,770 micro-USD:

| | |
|---|---|
| Provider / model | `openai` / `gpt-5.5`, standard mode, non-streaming |
| Normalized usage | 72 input tokens · 0 cached input · 47 output tokens · 0 reasoning · 119 total |
| Usage status | `complete` |
| Matched schedule | `openai-gpt-5.5-standard-lt272k-2026-08-07` |
| Match basis | returned model |

| Component | Tokens | Rate (USD / million) | Micro-USD |
|---|---:|---:|---:|
| Input, uncached | 72 | $5.00 | **360** |
| Input, cached | 0 | $0.50 | **0** |
| Output | 47 | $30.00 | **1,410** |
| **Total** | | | **1,770** |

Because the status is `complete`, this request contributes to the Support Ticket Demo advisory
budget. The full evidence — safe metadata, normalized usage, the component breakdown, and the
schedule provenance including its effective date and the date the source was checked — is reachable
at `#/requests/trc_demo_01_…`.

**No prompt or response content appears anywhere in that trace.** Everything above is derived from
provider-reported usage numbers and a stored rate.

## What is deliberately not claimed

- These are **estimates from published rates, not provider invoices**.
- They exclude taxes, credits, negotiated rates, regional adjustments, unsupported fees, and
  non-token charges.
- The underlying request is a deterministic fixture, not production traffic.
- A missing estimate means the evidence was insufficient. It does not mean the request was free.

---

Previous: [02 — Architecture and data flow](02-architecture-and-data-flow.md) · Next: [04 — Budgets and findings](04-budgets-and-findings.md)
