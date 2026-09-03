# 06 — Privacy and security

[← Back to the case study](../README.md)

![Privacy boundary: bodies, prompt text, credentials, streaming deltas, HMAC key material, and Prompt Trim suggestions stay in memory; only trace identity, model, timing, normalized counts, cost components, keyed fingerprints, and findings are persisted](../assets/diagrams/02-privacy-boundary.svg)

## The decision

Content capture was **rejected**, not deferred and not made opt-in.

An opt-in content store still needs encryption, retention policy, purge tooling, access control,
redaction, export filtering, and an answer for every future request to relax it "just for
debugging". A boundary that says content is never persisted removes all of that, permanently, and
makes the remaining privacy claims small enough to actually verify.

The cost is real: TokenReader cannot show you the prompt that caused an expensive request. That
tradeoff was accepted deliberately, and the desktop phases were held to it. Backups, diagnostic
bundles, licence validation, and update checks were each designed against a written data contract
that lists where every data class may go and where it may never go.

## What crosses the boundary

**Processed in memory, never persisted:** request and response bodies, prompt text, provider
credentials, arbitrary client headers, streaming content deltas, HMAC key material, and Prompt Trim
suggestions and revisions.

**Persisted (allowlisted metadata):** trace and project identity, exact provider and model, route,
processing mode, streaming flag, timing and time-to-first-event, HTTP status and final state,
normalized token counts by category, price-match status and schedule reference, cost components and
totals in micro-USD, keyed equality fingerprints, and advisory findings.

Raw bodies and events, credentials, HMAC keys, provider outputs, and fingerprint values are absent
from the database, logs, management APIs, exports, backups, diagnostic bundles, screenshots, and the
static snapshot. HMAC values additionally never enter exports, backups, or diagnostics.

Two exceptions exist, are narrow, and are labelled: the tracked public-safe synthetic evaluation
corpus, and one approved synthetic Prompt Trim example inside the demo snapshot.

## Credential handling

In the desktop application, provider keys and the local SDK token live only in Windows Credential
Manager. They are read and written by the Electron main process through a repository-owned adapter,
handed to the gateway child over stdin, and never appear in argv, the environment, the renderer, a
log, an export, a backup, or a diagnostic bundle. There is no plaintext fallback: without the vault,
fixture mode continues and live mode is unavailable. Both uninstall choices remove the credential
entries.

In the browser development workflow, placeholder SDK credentials stop at the gateway. They are
stripped before forwarding and never logged, persisted, or exported. Optional live-evaluation keys
are read from standard provider environment variables **only after** the enablement and
acknowledgement gates pass, so a misconfigured run cannot pick up a key it was never authorised to
use.

The HMAC fingerprint key lives in a file outside the database, and outside every API, log, export,
backup, and build artifact. A missing or invalid key **fails safely**; there is no plain-hash
fallback, because a silent downgrade from keyed to unkeyed hashing would turn a privacy control into
a false one.

## Threat model

The gateway and dashboard controls from the MVP:

| Threat | Control | Residual risk |
|---|---|---|
| Credential or body disclosure | Header stripping, allowlisted logging and persistence, sentinel scans, no response accumulation | A same-user or compromised machine can inspect process memory |
| DNS rebinding / unsafe Host | Loopback binding and Host validation | User overrides can weaken local assumptions |
| Cross-site requests to local services | Origin validation on management routes, no wildcard CORS, explicit JSON mutation methods | Non-browser SDK traffic intentionally carries no Origin |
| Oversized bodies | Content-length limits, a smaller Prompt Trim limit, deterministic 413 | Streaming transfer limits depend on server behaviour |
| Database disclosure | Metadata-only schema; key stored separately | **No application-level encryption**; filesystem access reveals metadata |
| Equality leakage | External secret key, explicit disclosure, per-project disable and reset | Equal digests reveal repeated normalized blocks |
| Static artifact leakage | Deterministic allowlisted snapshot, read-only interface, scans, no analytics or storage | Synthetic usage patterns are intentionally public in the artifact |
| Logs, exports, screenshots | Field allowlists, spreadsheet-formula neutralization, automated scans, manual inspection | New fields require review |
| Dependencies and build | Pinned lockfiles, audits, local bundle scan, CSP and no external requests | Supply-chain compromise is not eliminated |

The desktop, lifecycle, and commercial controls added since:

| Threat | Control | Residual risk |
|---|---|---|
| Renderer compromise | Sandboxed, context-isolated renderer; typed allowlisted preload bridge asserted at runtime; navigation, permissions, and network denied; no credentials, filesystem, or trust decisions in the renderer | Electron and Chromium compromise remains material |
| Rogue local client of the gateway | Per-launch 256-bit session secret over stdin; authenticated management and readiness routes; supervised owned child | Same-user process attacks require OS hardening |
| Unauthorised local paid requests | Persistent local SDK token on the provider-shaped routes, constant-time comparison, route-scoped, rotatable | A same-user process that reads the vault or clipboard can use the token |
| Provider destination tampering | Fixed two-endpoint allowlist, TLS verification, redirect refusal, no environment proxies, zero retries, no client upstream selection | Provider-side behaviour after an authorised live request is out of scope |
| Live-mode surprise spend | Fixture default; explicit per-provider confirmation naming charges; no auto-verification; saving a key never enables live; licence required for live mode at all | The user's own live traffic is billed by the provider |
| An upgrade damages the only database | Verified pre-upgrade backup, staged candidate, atomic replacement, automatic rollback if readiness fails; newer-schema databases refused untouched | Validated in Sandbox scenarios, not on customer hardware |
| A hostile backup archive is restored | Exact two-entry allowlist, bounded size and ratio, symlink and traversal refusal, checksum, staged extraction and staged upgrade before replacement | A same-user actor can replace a backup with a valid one they authored. Backups are unencrypted and unsigned |
| Purge misuse or incompleteness | Scope preview, confirmation, bounded categories, secure delete and vacuum, durable status, receipts; a failed backup cleanup reports pending, never complete | SSD wear levelling, snapshots, antivirus quarantine, and user copies are out of reach |
| Diagnostic bundle leaks data | Field allowlist validated at generation, preview from one captured snapshot, bounded archive, no upload path | The user controls the exported file afterwards |
| Forged or tampered licence | Ed25519 signature over canonical bytes, bounded key IDs, forbidden-field refusal, dual validation with the gateway as enforcement authority, production trust empty and fail-closed | Licence-file sharing is an accepted commercial risk; no binding will be added |
| Malicious or substituted update | Signed manifest verified first, feed must agree on version and SHA-512, exact host allowlist with zero redirects, consent before every side effect, adapter-side Authenticode verification of the download | Signing-key compromise requires a recovery design that does not exist yet |
| Tampered or unsigned binary | Signing inventory gate; release mode fails closed on any unsigned required binary | Development builds are unsigned by design and lower assurance |
| Onboarding state forged | Schema-versioned record in main-process runtime state with strict validation; corrupt files preserved and a safe first run re-entered | The record gates no privilege and holds no secret |

Management responses carry a Content Security Policy, frame denial, no-referrer, MIME-sniffing
protection, and restricted permissions. Browser storage is inspected by the demo-bundle scan and the
packaged smoke harness and must remain empty.

## The leakage that is disclosed rather than hidden

Keyed fingerprints reveal that two normalized blocks were **equal**. They do not reveal what either
block said, and they cannot be reversed to recover text. But equality is information: an observer
with database access learns that a user repeats a particular block across requests.

This is stated plainly in the product interface, in the threat model, and here, rather than being
described as "hashed, therefore private". Fingerprinting can be disabled per project, and a
confirmed reset rotates the key and removes every HMAC row and equality-derived finding while leaving
cost history intact.

## Explicit exclusions

TokenReader does not defend against a fully compromised machine, a malicious administrator or local
user, provider-side retention after an intentionally authorised live request, physical disk theft
without OS-level encryption, denial of service by same-user processes, or remote and multi-user
hosting.

**No security certification is claimed.** No third-party audit or penetration test has been
performed.

## Export handling

CSV and JSON exports share bounded request filters and contain metadata only. CSV cells beginning
with spreadsheet formula characters are neutralized to prevent formula injection when a reviewer
opens the file in a spreadsheet. In the desktop application, exports are explicit user-initiated
file downloads brokered through the main process. No server-side export file is left behind after
generation.

---

Previous: [05 — Prompt Trim and evaluation](05-prompt-trim-and-evaluation.md) · Next: [07 — Testing and validation](07-testing-and-validation.md)
