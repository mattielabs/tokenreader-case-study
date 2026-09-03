# 01 — Product overview

[← Back to the case study](../README.md)

## The user

A technically fluent solo developer or small AI team shipping LLM-backed features. They are
comfortable reading a stack trace and a SQL query. They are not looking for a growth dashboard, a
model leaderboard, or a vendor that takes control of their production traffic.

## The problem

The trigger is a vague concern, *"we're spending too many tokens"*, that a provider invoice cannot
resolve. An invoice is a monthly total. It cannot tell you:

- **Attribution.** Which application request consumed which tokens?
- **Composition.** Was that input cached, reasoned, or written to cache? Those bill differently.
- **Confidence.** Is a missing cost estimate genuinely zero, or is the evidence incomplete?
- **Safety.** If you shorten a prompt to save money, did you delete a required instruction?

Each of these is answerable from data the providers already return. Nothing about answering them
requires shipping prompt content to a third party, or shipping it anywhere at all.

## The shape of the answer

TokenReader sits between an application's official provider SDK and the upstream API. It forwards
the native request unchanged and records content-free operational evidence as it passes through.

```text
official SDK -> local gateway -> upstream (fixture by default) -> native JSON / SSE
                      |
                      +-> allowlisted metadata -> SQLite -> typed API -> dashboard
```

On Windows the gateway and the dashboard ship together as one desktop application. The product's
job is to **observe, explain, and help the user decide**. Every consequential action stays with
the user.

## Decisions that were locked early

| Decision | Choice | Why it matters |
|---|---|---|
| Retention | **Metadata only** | Content capture was rejected outright, not offered as opt-in. A feature that does not exist cannot leak |
| Default mode | **Deterministic fixtures** | Validation requires no key, no paid call, and no network. Results are reproducible |
| Budgets | **Advisory** | They warn. They never block, delay, reroute, retry, or modify a request |
| Prompt changes | **Opt-in, in memory** | Nothing is rewritten automatically, and nothing is rewritten in the live request path |
| Provider interfaces | **Native, not abstracted** | OpenAI Responses and Anthropic Messages keep their own shapes, including their own SSE protocols |
| Pricing | **Exact dated schedules** | No runtime price scraping, no prefix or family matching, no nearest-model guessing |
| Missing data | **Never treated as zero** | An unknown cost is reported as unknown, with a reason |
| Runtime | **Direct Python and Node, packaged for Windows** | Docker is optional packaging, not the supported path. The desktop build bundles its own gateway and needs no system Python or Node |
| Scope | **Local, single user** | No accounts, roles, SSO, or remote multi-user access |
| Telemetry | **None** | No usage reporting, no crash upload, no analytics. Licence checks and update checks carry no product data |
| Commercial model | **Signed offline lifetime licence** | Validated locally, no activation server, no hardware binding, and nothing local is ever locked behind it |

## Explicit non-goals

TokenReader deliberately does **not**:

- route requests automatically between providers or models;
- block requests when a budget is reached;
- rewrite prompts in the live request path;
- claim agreement with provider invoices;
- persist prompt or response text, even as an opt-in feature;
- support accounts, teams, roles, SSO, or remote access;
- proxy every endpoint of either provider API;
- support images, audio, files, tools, or batch requests;
- rank models by general intelligence or repeat provider marketing claims;
- present fixture results as production savings or adoption;
- phone home, for licensing, updates, or anything else.

Several of these are the kind of feature that would make the product look more impressive in a demo.
They are excluded because each one either takes control away from the user or manufactures a claim
the evidence cannot support.

## What the dashboard is organised around

| Destination | Question it answers |
|---|---|
| Overview | Where does spend stand against the advisory budget, how is it composed, what changed this week, and what needs inspection? |
| Requests | How does an aggregate figure trace back to one specific attempt? |
| Budgets | What advisory limit applies to this project this UTC month? |
| Optimize | What findings exist, what would Prompt Trim propose, and what did the offline evaluation show? |
| Model Guide | Which exact model IDs were observed, and where did their rates come from? Task guidance is labelled unavailable rather than invented |
| Learn | Structure for lessons, with no fabricated lessons and no remote content service |
| Settings | General options, providers, privacy operations, licence, and updates, with each stateful control appearing exactly once |

Missing values are labelled rather than plotted as zero. Fixture-sourced evidence is labelled as
synthetic on every screen. The visual system is a light and dark token set gated to WCAG AA in both
themes, a sticky top bar with the seven destinations as a pill strip, a command palette over stored
metadata only, Hanken Grotesk for prose and Spline Sans Mono for identifiers and numbers. It should
read as an engineering instrument, not a marketing surface.

## Status

TokenReader 0.3.0, an unsigned development build, recorded against source commit
`1f174203c363e43757010a9af703c04dd7b148b1` on 2026-08-28. The full validation gate passed, and the
installer passed clean-machine qualification in Windows Sandbox on the same day. External beta is
blocked on a production signing certificate, production licence and update keys, an update domain,
and a releases repository, none of which exist yet. Public release of the source is not planned; the
source is proprietary.

---

Next: [02 — Architecture and data flow](02-architecture-and-data-flow.md)
