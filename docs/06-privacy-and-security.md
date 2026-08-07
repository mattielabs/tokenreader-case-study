# 06 — Privacy and security

[← Back to the case study](../README.md)

![Privacy boundary: bodies, prompt text, credentials, streaming deltas, HMAC key material, and Prompt Trim suggestions stay in memory; only trace identity, model, timing, normalized counts, cost components, keyed fingerprints, and findings are persisted](../assets/diagrams/02-privacy-boundary.svg)

## The decision

Content capture was **rejected**, not deferred and not made opt-in.

That distinction is the whole design. An opt-in content store still needs encryption, retention
policy, purge tooling, access control, redaction, export filtering, and an answer for every future
request to relax it "just for debugging". A boundary that says content is never persisted removes
all of that, permanently, and makes the remaining privacy claims small enough to actually verify.

The cost is real: TokenOps cannot show you the prompt that caused an expensive request. That
tradeoff was accepted deliberately.

## What crosses the boundary

**Processed in memory, never persisted:** request and response bodies, prompt text, provider
credentials, arbitrary client headers, streaming content deltas, HMAC key material, and Prompt Trim
suggestions and revisions.

**Persisted (allowlisted metadata):** trace and project identity, exact provider and model, route,
processing mode, streaming flag, timing and time-to-first-event, HTTP status and final state,
normalized token counts by category, price-match status and schedule reference, cost components and
totals in micro-USD, keyed equality fingerprints, and advisory findings.

Raw bodies and events, credentials, HMAC keys, provider outputs, and fingerprint values are absent
from the database, logs, management APIs, exports, screenshots, and the static snapshot.

Two exceptions exist, are narrow, and are labelled: the tracked public-safe synthetic evaluation
corpus, and one approved synthetic Prompt Trim example inside the demo snapshot.

## Credential handling

Placeholder SDK credentials stop at the gateway. They are stripped before forwarding and never
logged, persisted, or exported.

Optional live-mode keys are read from standard provider environment variables **only after** the
enablement and acknowledgement gates pass — the read is deliberately deferred rather than happening
at import time, so a misconfigured run cannot pick up a key it was never authorised to use.

The HMAC fingerprint key lives in a file outside the database, and outside every API, log, export, and
build artifact. A missing or invalid key **fails safely**; there is no plain-hash fallback, because a
silent downgrade from keyed to unkeyed hashing would turn a privacy control into a false one.

## Threat model

| Threat | Control | Residual risk |
|---|---|---|
| Credential or body disclosure | Header stripping, allowlisted logging and persistence, sentinel scans, no response accumulation | A same-user or compromised machine can inspect process memory |
| DNS rebinding / unsafe Host | Loopback binding and Host validation | User overrides can weaken local assumptions |
| Cross-site requests to local services | Origin validation on management routes, no wildcard CORS, explicit JSON mutation methods | Non-browser SDK traffic intentionally carries no Origin |
| Network exposure | Loopback defaults throughout scripts and configuration | Manual non-loopback changes are unsupported |
| Oversized bodies | Content-length limits, a smaller Prompt Trim limit, deterministic 413 | Streaming transfer limits depend on server behaviour |
| Database disclosure | Metadata-only schema; key stored separately | **No application-level encryption**; filesystem access reveals metadata |
| Equality leakage | External secret key, explicit disclosure, per-project disable and reset | Equal digests reveal repeated normalized blocks |
| Static artifact leakage | Deterministic allowlisted snapshot, read-only interface, scans, no analytics or storage | Synthetic usage patterns are intentionally public in the artifact |
| Evaluation fixture leakage | Only invented public-safe text; reports contain no prompt or output | The corpus is source-controlled by design |
| Logs, exports, screenshots | Field allowlists, spreadsheet-formula neutralization, automated scans, manual inspection | New fields require review |
| Dependencies and build | Pinned lockfiles, audits, local bundle scan, CSP and no external requests | Supply-chain compromise is not eliminated |

Management responses carry a Content Security Policy, frame denial, no-referrer, MIME-sniffing
protection, and restricted permissions. Browser storage is inspected and must remain free of content.

## The leakage that is disclosed rather than hidden

Keyed fingerprints reveal that two normalized blocks were **equal**. They do not reveal what either
block said, and they cannot be reversed to recover text. But equality is information: an observer
with database access learns that a user repeats a particular block across requests.

This is stated plainly in the product interface, in the threat model, and here — rather than being
described as "hashed, therefore private". Fingerprinting can be disabled per project, and a confirmed
reset rotates the key and removes every HMAC row and equality-derived finding while leaving cost
history intact.

## Explicit exclusions

TokenOps does not defend against a fully compromised machine, a malicious administrator or local
user, provider-side retention after an intentionally authorised live request, physical disk theft
without OS-level encryption, denial of service by same-user processes, or remote and multi-user
hosting.

**No security certification is claimed.** No third-party audit or penetration test has been
performed.

## Export handling

CSV and JSON exports share bounded request filters and contain metadata only. CSV cells beginning
with spreadsheet formula characters are neutralized to prevent formula injection when a reviewer
opens the file in a spreadsheet. No server-side export file is left behind after generation.

---

Previous: [05 — Prompt Trim and evaluation](05-prompt-trim-and-evaluation.md) · Next: [07 — Testing and validation](07-testing-and-validation.md)
