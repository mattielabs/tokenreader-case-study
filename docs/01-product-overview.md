# 01 — Product overview

[← Back to the case study](../README.md)

## The user

A technically fluent solo developer or small AI team shipping LLM-backed features. They are
comfortable reading a stack trace and a SQL query. They are not looking for a growth dashboard, a
model leaderboard, or a vendor that takes control of their production traffic.

## The problem

The trigger is a vague concern — *"we're spending too many tokens"* — that a provider invoice cannot
resolve. An invoice is a monthly total. It cannot tell you:

- **Attribution.** Which application request consumed which tokens?
- **Composition.** Was that input cached, reasoned, or written to cache? Those bill differently.
- **Confidence.** Is a missing cost estimate genuinely zero, or is the evidence incomplete?
- **Safety.** If you shorten a prompt to save money, did you delete a required instruction?

Each of these is answerable from data the providers already return. Nothing about answering them
requires shipping prompt content to a third party, or shipping it anywhere at all.

## The shape of the answer

TokenOps sits between an application's official provider SDK and the upstream API. It forwards the
native request unchanged and records content-free operational evidence as it passes through.

```text
official SDK -> local gateway -> deterministic fixture -> native JSON / SSE
                      |
                      +-> allowlisted metadata -> SQLite -> typed API -> dashboard
```

The product's job is to **observe, explain, and help the user decide**. Every consequential action
stays with the user.

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
| Runtime | **Direct Python and Node** | Docker is optional packaging, not the supported path |
| Scope | **Local, single user** | No accounts, roles, SSO, or remote multi-user access |

## Explicit non-goals

The MVP deliberately does **not**:

- route requests automatically between providers or models;
- block requests when a budget is reached;
- rewrite prompts in the live request path;
- claim agreement with provider invoices;
- persist prompt or response text, even as an opt-in feature;
- support accounts, teams, roles, SSO, or remote access;
- proxy every endpoint of either provider API;
- support images, audio, files, tools, or batch requests;
- rank models by general intelligence or repeat provider marketing claims;
- present fixture results as production savings or adoption.

Several of these are the kind of feature that would make the product look more impressive in a demo.
They are excluded because each one either takes control away from the user or manufactures a claim
the evidence cannot support.

## What the dashboard is organised around

| View | Question it answers |
|---|---|
| Overview | Where does spend stand against the advisory budget, how is it composed, and what needs inspection? |
| Requests | How does an aggregate figure trace back to one specific attempt? |
| Budgets | What advisory limit applies to this project this UTC month? |
| Optimize | What findings exist, what would Prompt Trim propose, and what did the offline evaluation show? |
| Models | Which exact model IDs were observed, and where did their rates come from? |
| Privacy | What is actually implemented, what are the limits, and how do I reset or export? |

Missing values are labelled rather than plotted as zero. Fixture-sourced evidence is labelled as
synthetic on every screen. The redesigned visual system uses an off-white evidence workspace, a
compact 64px rail, strong numeric hierarchy, and restrained purple, magenta, teal, amber, and red
state colours. Instrument Sans carries prose while IBM Plex Mono distinguishes traceable evidence.
At small widths the rail becomes bottom navigation and evidence groups collapse without hiding
essential actions. It should read as an engineering instrument, not a marketing surface.

## Status

Week 4 local MVP, verdict **PASS** under the local metadata-only workflow, recorded against source
commit `4e7aff9c1681a38fb86e1aa1d12905512741cff9`. Public release of the software is blocked pending
a licence decision.

---

Next: [02 — Architecture and data flow](02-architecture-and-data-flow.md)
