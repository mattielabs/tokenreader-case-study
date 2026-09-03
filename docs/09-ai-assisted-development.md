# 09 — AI-assisted development

[← Back to the case study](../README.md)

## The disclosure

The TokenReader implementation was **substantially produced with AI coding assistance**.

Alex Mattie defined the problem, the scope, the constraints, the privacy decisions, the phase
boundaries, the acceptance gates, and the release decisions, and directed and reviewed the work
throughout. The code itself was largely generated under that direction rather than hand-written line
by line. Alex personally performed the manual accessibility reviews in Windows Sandbox and recorded
each sign-off.

This is stated up front, in the project's own README, and here.

## Why it is stated this way

A portfolio exists to let someone predict how you will perform on their problems. A portfolio that
misrepresents how its artifact was produced makes that prediction worse for everyone, including the
candidate, who then has to sustain the misrepresentation in a technical interview.

There is a specific reason it would have been incoherent here. The entire product is built around
refusing to overstate evidence: costs are labelled estimates rather than invoices, missing data is
labelled missing rather than zeroed, fingerprint equality leakage is disclosed rather than described
as "hashed and therefore private", a savings figure is withheld because it cannot be justified, and
a qualification that ran against the wrong binary is marked superseded rather than counted. Applying
that standard to provider usage data and then abandoning it when describing one's own contribution
would be the one dishonest claim in an otherwise carefully honest project.

## What the evidence supports

The repository contains traceable evidence of what the system does and how it was checked:

- 359 gateway, 164 dashboard, 226 desktop, and 8 issuer-tool tests over specific, declared
  behaviours;
- twenty-two numbered architecture decision records, including the alternatives that were tried and
  rejected;
- seven phase execution records that carry starting state, decisions, milestone logs, defects found,
  and final validation, with superseded evidence kept and labelled;
- a 30-ticket evaluation corpus with deterministic rubrics and a content-free report;
- generated artifacts that are drift-checked, so documented numbers cannot silently go stale;
- a build log recording exact commands, results, and verdicts for each phase;
- a threat model that names residual risk rather than only listing controls;
- Sandbox qualification evidence with per-scenario run IDs and installer hashes.

That evidence supports claims about **the system**: what it does, what it refuses to do, how it was
validated, and where its limits are.

## What the evidence does not support

It does not establish that every source file was written independently, and it should not be read
that way.

The honest positioning is an **AI-assisted, owner-directed build**. The judgment on display is
product and engineering judgment: deciding that content capture should be rejected rather than made
opt-in, that budgets should warn rather than block, that nearest-model price matching is worse than
an honest gap, that a no-change result is a passing result, that a savings estimate without a
concrete comparison should not be shown at all, that a licence should bind to nothing and lock
nothing local, that the installer is never run on the development machine, and that a rename with
installed users is a migration first.

Those decisions determined what the system is. They are separable from, and were not delegated to,
the generation of the code that implements them.

## The standard being applied

Alex should be able to explain, debug, and modify the important architecture before presenting this
project as independent engineering ability. Until then, the project is presented as what it is.

A reviewer is welcome to probe any of it. The decisions above, and the defects the qualification
matrix found, are the parts worth asking about.

---

Previous: [08 — Static recruiter demo](08-static-recruiter-demo.md) · Next: [10 — Limitations and next steps](10-limitations-and-next-steps.md)
