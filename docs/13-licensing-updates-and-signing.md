# 13 — Licensing, updates, and signing

[← Back to the case study](../README.md)

Phase 1F built the commercial foundation and stopped, on purpose, one step before external beta. No
domain was registered, no certificate obtained, no public repository created, no release published.
Everything that depends on those things is built fail-closed with the placeholders refused, so a
release cannot be produced by accident.

## Three trust roots that never touch

| Trust root | Algorithm | Signs | Production keys |
|---|---|---|---|
| Offline licences | Ed25519 | `.tokenreader-license` packages | None. Registry committed empty, fails closed |
| Update manifests | Ed25519 | Release manifests | None. Registry committed empty, fails closed |
| Windows Authenticode | X.509 with RFC 3161 timestamps, SHA-256 only | Every shipped binary and the installer | None. Development builds are unsigned and say so |

A licence key cannot sign a manifest and a manifest key cannot sign a licence. The registries are
separate, the signed payloads carry distinct format markers inside the signed bytes, and each
validator consults only its own registry. Every signed object names a bounded key ID; unknown IDs
fail closed. Rotation is add-alongside-then-remove. Development keys are embedded, clearly named
`-dev-`, and refused by production builds. The private halves of the development keys are committed
as fixtures in a purpose-built seed format with no PEM banner, so they cannot be mistaken for real
credentials by a scanner or a person.

Ed25519 was chosen because both runtimes support it directly, it is deterministic, and its keys and
signatures are small enough for a strict canonical-bytes contract. The Python gateway gained exactly
one new pinned dependency for it.

## The licence

A licence is a portable file of at most 8 KB: canonical JSON, signed offline. The payload holds a
schema version, key ID, licence ID, product ID, edition, a `lifetime` entitlement, an issue instant,
an optional customer display name, optional allowed channels, a terms version, and the algorithm.

It deliberately contains **no** hardware identifier, machine fingerprint, hostname, account, email,
credential, payment detail, usage counter, telemetry identifier, activation nonce, revocation
callback, or hidden mutable state. The validators refuse a licence that carries any such field even
when it is correctly signed. That forbidden-field list is the contract's way of making sure a future
phase cannot quietly add binding.

The canonical-bytes rule is the whole parser: a signed file must equal the canonical re-serialisation
of its own parse. One rule defeats duplicate keys, reordering, whitespace smuggling, trailing data,
and alternate unsigned forms.

**Validation is local, twice, and fail-closed.** The desktop main process validates for the
interface and the cached copy. The gateway re-validates the exact package bytes from its stdin
bootstrap, with the trust channel defaulting to the stricter production setting, and is the
enforcement authority. Without a gateway-accepted licence, every live provider mode downgrades to
fixture, supplied credentials are dropped, and live upstream resolution refuses with a safe 403.
The renderer cannot bypass this, and neither can a desktop UI defect. The same fixture licence
validates in both runtimes; production trust refuses it as development-only; a flipped byte reads as
`tampered`; truncation reads as `malformed`.

**What never locks.** Fixture exploration, all existing metadata and history, exports, backup and
restore, retention, purge, receipts, diagnostics, Settings, and licence import work without any
licence. A valid commercial licence is required only to enable live provider mode and future paid
modules. The product never holds data hostage.

**Portability over enforcement.** The licence is bound to nothing. Recovery is re-importing the
original file on any supported machine, with no support intervention. Sharing a licence file is an
accepted commercial risk for V1, and the decision record says that no hidden binding may be added
later.

Issuing is an offline command-line tool with no network path, deterministic outputs for explicit
inputs, overwrite refusal, deterministic exit codes, and no private-key echo. It is never packaged
into the application.

## Updates

The third-party updater executes the NSIS download and install mechanics. Every policy decision
lives in a narrow main-process service behind typed IPC.

A signed release manifest is the authority. It is fetched from the future update domain (bounded to
8 KB, ten-second timeout, exact host, HTTPS, zero redirects) and verified against the update trust
root before anything is treated as a release. It pins product, channel, version, tag, publish
instant, minimum supported version, exact asset name, SHA-256, SHA-512, byte size, and the exact
public repository. The updater's own feed metadata must then agree exactly on version and SHA-512
or the release is refused. Downgrades, same-version replacement, wrong channels, unknown schemas,
wrong products, unknown keys, and tampering each refuse with a deterministic code.

**Consent everywhere.** Automatic checks default off. When opted in, they run only after
authenticated gateway readiness, at most once per 24 hours, never during a lifecycle operation, and
never delay startup. Manual checks are always available. Download and install each require an
explicit click. Cancellation is safe. Failures never make the application unusable. No usage data
accompanies any request.

**Network policy.** The update service alone may reach the future update domain, `api.github.com`,
`github.com`, and `objects.githubusercontent.com`. No wildcards. Development builds reach only an
explicit loopback test fixture. The distribution contract is baked at build time with no runtime
environment fallback; placeholders and development keys are refused; release mode reports
`configuration_missing` while anything is absent.

**No binary rollback is claimed.** NSIS and the updater cannot provide one safely. Data integrity
relies on the existing staged migration with its verified pre-upgrade backup and rollback, and a
failed post-update startup surfaces as recovery, never as a successful update.

Scenario K exercised the real path in a Sandbox: a test-signed higher-version installer checked,
downloaded, verified, and installed silently, with data, licence, credentials, and port preserved.
It also found the updater's own signature check failing open, which is described in the
qualification document.

## Code signing

A signing inventory enumerates every shipped PE artifact and classifies it. Required binaries (the
desktop executable, the packaged gateway, every native module, any helper, and the installer) must
be Authenticode-signed in release mode with the exact expected publisher subject, a valid signature,
and an RFC 3161 timestamp. Upstream runtime DLLs are recorded by name and hash and covered by the
signed installer envelope. Any new artifact category fails the inventory rather than being skipped.
Development builds are expected unsigned, and the inventory fails a development build that
unexpectedly carries a production-like signature.

Signability was proven with an ephemeral test certificate created in git-ignored temporary space,
never added to any host trust store. The full trusted flow, including the updater's publisher check,
ran only inside a discarded Sandbox. None of it is claimed as production signing.

## Release bundles

A release bundle is generated deterministically into ignored output: the installer, the updater
feed file, the signed manifest, a notes template, hashes, CycloneDX SBOMs for all three npm packages
plus lockfile digests, and a provenance record with the source commit, tree cleanliness, migration
head, trust-root key IDs, signing inventory, and validation and qualification notes, under a
self-describing evidence manifest.

A fail-closed inspector checks an exact artifact allowlist (any unexpected file or executable
fails), hash integrity, a secret scan that knows the project's own key-document format, and the
absence of private source. In release mode it additionally refuses placeholder endpoints,
development trust roots, unsigned or untimestamped inventories, dirty or untracked source trees, and
missing validation or qualification evidence. The desktop validation script runs the development
bundle end to end **and asserts that the release-mode gate refuses it**, so the fail-closed
behaviour is itself tested on every validation run.

## What is blocked, and by what

| Needed for external beta | Exists |
|---|---|
| Production code-signing certificate and publisher subject | No |
| Production licence signing key | No |
| Production update-manifest signing key | No |
| Update domain | No |
| Public releases repository | No (planned name recorded; not created) |
| A qualified signed binary | No; the qualified binary is unsigned |

Each absence is a documented placeholder that the gates refuse. The pipeline exercises the identical
code paths on every validation run, so readiness cannot silently rot while the decisions above wait.

---

Previous: [12 — Clean-machine qualification](12-clean-machine-qualification.md) · Next: [14 — The rename](14-the-rename.md)
