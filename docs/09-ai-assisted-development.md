# 09 — AI-assisted development

[← Back to the case study](../README.md)

## The disclosure

The TokenOps implementation was **substantially produced with AI coding assistance**.

Alex Mattie defined the problem, the scope, the constraints, the privacy decisions, the acceptance
gates, and the release decisions, and directed and reviewed the work throughout. The code itself was
largely generated under that direction rather than hand-written line by line.

This is stated up front, in the project's own README, and here — not buried in a footnote.

## Why it is stated this way

A portfolio exists to let someone predict how you will perform on their problems. A portfolio that
misrepresents how its artifact was produced makes that prediction worse for everyone, including the
candidate, who then has to sustain the misrepresentation in a technical interview.

There is a specific reason it would have been incoherent here. The entire product is built around
refusing to overstate evidence: costs are labelled estimates rather than invoices, missing data is
labelled missing rather than zeroed, fingerprint equality leakage is disclosed rather than described
as "hashed and therefore private", and a savings figure is withheld because it cannot be justified.
Applying that standard to provider usage data and then abandoning it when describing one's own
contribution would be the one dishonest claim in an otherwise carefully honest project.

## What the evidence supports

The repository contains traceable evidence of what the system does and how it was checked:

- 121 backend tests and 11 frontend tests over specific, declared behaviours;
- eleven numbered architecture decision records capturing what was chosen and why;
- a 30-ticket evaluation corpus with deterministic rubrics and a content-free report;
- generated artifacts that are drift-checked, so documented numbers cannot silently go stale;
- a build log recording exact commands, results, and verdicts for each phase;
- a threat model that names residual risk rather than only listing controls.

That evidence supports claims about **the system**: what it does, what it refuses to do, how it was
validated, and where its limits are.

## What the evidence does not support

It does not establish that every source file was written independently, and it should not be read
that way.

The honest positioning is an **AI-assisted, owner-directed build**. The judgment on display is
product and engineering judgment: deciding that content capture should be rejected rather than made
opt-in, that budgets should warn rather than block, that nearest-model price matching is worse than
an honest gap, that a no-change result is a passing result, and that a savings estimate without a
concrete comparison should not be shown at all.

Those decisions determined what the system is. They are separable from, and were not delegated to,
the generation of the code that implements them.

## The standard being applied

Alex should be able to explain, debug, and modify the important architecture before presenting this
project as independent engineering ability. Until then, the project is presented as what it is.

A reviewer is welcome to probe any of it. The decisions above are the parts worth asking about.

---

Previous: [08 — Static recruiter demo](08-static-recruiter-demo.md) · Next: [10 — Limitations and next steps](10-limitations-and-next-steps.md)
