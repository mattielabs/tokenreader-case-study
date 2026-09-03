# 12 — Clean-machine qualification

[← Back to the case study](../README.md)

An installer that works on the machine that built it has proven very little. The development host
has the toolchain, the runtime, the fonts, a console attached to every launch, and years of state.
The rule adopted in Phase 1E was blunt: **host-machine testing is not clean-machine evidence**, and
the installer is never executed on the primary development environment.

## The harness

Every scenario runs in its own fresh Windows Sandbox, rendered from one tracked configuration
template. Networking, vGPU, audio input, video input, printer, and clipboard redirection are all
disabled. Protected Client is enabled. Exactly two folders are mapped: the qualification input
(installer, scripts, fixtures) read-only, and one initially empty output folder read-write for
sanitised evidence. No repository checkout, user profile, AppData, or credential enters the guest.

A host-side generator verifies the installer's hash, stages the input, and renders one
machine-specific `.wsb` per scenario. An in-guest runner executes the scenario, writes one evidence
JSON per scenario against a JSON Schema, runs a prohibited-content scan over the evidence, verifies
its own cleanup, and powers the guest off. A host-side inspector then checks that the bundle
contains no secret, credential, private source, or absolute developer path, and that every evidence
file references the expected installer hash.

The prohibited-content scan fails the run on any API-key shape, 64-hex token, session header, credential
blob, prompt or response content, fingerprint material, or real user path. If a scenario were ever
to leak, the leak would fail the scenario rather than land in evidence.

## The matrix

| Scenario | What it proves |
|---|---|
| A | Fresh install without a desktop shortcut; one Apps & Features entry; launch, fixture sample, clean exit with no orphan process or listener |
| B | Fresh install with the desktop shortcut; no duplicate shortcuts or registrations |
| C | Same-version repair restores a deleted binary and leaves database, state, port, onboarding, and the shortcut choice untouched |
| D | Prior-version upgrade preserves data, credentials, port, and onboarding, and skips the pre-upgrade backup when the schema is already at head |
| E | Uninstall with preservation keeps the data root, removes the application and all three credential targets, leaves an unrelated sentinel credential alone; reinstall reaches readiness over the byte-identical database |
| F | Uninstall with deletion removes only the canonical data root, never follows a junction out of it, leaves an external manual backup untouched, and reports partial cleanup honestly |
| G | Port conflict surfaces the safe state, never inspects or kills the foreign process, and recovers once the port is free |
| H | Long paths and non-ASCII file names; ISO-8601 timestamps; no username or personal path in a diagnostic bundle |
| I | Unsupported-system refusal via mock hooks, recorded explicitly as mock-driven and not real-machine evidence |
| J | Offline licence lifecycle: import, refusal of tampered and development-signed packages under production trust, live mode gated |
| K | A test-signed update is checked, downloaded, verified, and installed by the real NSIS path with data, licence, credentials, and port preserved |
| L | The actual pre-rename build is installed, seeds its own data, port, credentials, and licence, and is upgraded by the TokenReader installer with everything migrated and compared by value |
| M | Fresh TokenReader installation on a machine that never had the old name |

Scenarios A to I were built in Phase 1E, J and K in Phase 1F, and L and M in Phase 1G. The matrix is
permanent: each phase added scenarios and every phase re-ran the whole thing.

## Results

| Run | Installer | Result |
|---|---|---|
| Phase 1E-Q, 2026-08-20/21 | Pre-rename 0.2.0 | Scenarios A–I, **124 / 124** |
| Phase 1E-Q requalification, 2026-08-24 | Pre-rename 0.2.0 after the dashboard re-skin | Scenarios A–I, **124 / 124**; manual accessibility sign-off |
| Phase 1F, 2026-08-24 | Pre-rename 0.2.0 with licensing and updates | Scenarios J and K, **38 / 38** |
| Phase 1G-Q, 2026-08-27/28 | **TokenReader 0.3.0**, sha256 `c520975d…c26e6` | **12 applicable scenarios, 215 / 215**; scenario D not applicable; manual sign-off 2026-08-28 |

Scenario D expects a prior TokenReader release, and none exists, since 0.3.0 is the first build under
that name. It was recorded as not applicable, excluded from every total, and no earlier release was
fabricated to satisfy it. Scenario L exercises the upgrade that actually matters: the pre-rename
product to TokenReader.

Every qualification applies to the exact artifact it ran against. Any product change produces a
different binary, which voids the evidence and requires a full rerun. Two earlier 0.3.0 binaries
carried the same filename during Phase 1G-Q; their evidence is retained in dated superseded folders,
excluded from all totals, and never confused with the qualified one.

## The manual review

Automated axe and contrast gates cover rule-checkable accessibility. They cannot tell you whether
Narrator reads onboarding in a sensible order. Each qualification therefore ends with a manual
checklist performed by Alex in an interactive Sandbox with the same isolation contract and no
automatic runner: keyboard-only operation through onboarding, all seven destinations, every Settings
section, and a destructive confirmation; Windows Narrator; display scaling at 100, 125, 150, and 200
percent; high contrast and forced colours; reduced motion, with a control check so "no motion"
could not be confused with "no animation exists"; and keyboard operation of the installer and
uninstaller.

Each result was recorded only on an explicit operator response. **No accessibility certification is
claimed.** This is a human observation, recorded as one.

## What the matrix found

Qualification is worth writing about because it found things. Three product defects and one
misdiagnosis are the interesting ones.

**A crash at exit that only users would see.** Every packaged-application exit after a launch
without console standard handles, which is the normal double-click launch, died with `0xC0000409`
inside the native credential-vault module's DLL teardown. It reproduced six times out of six in the
Sandbox and never on the host, because host validation always launched from a console. Three
remedies were tried and rejected, each verified in a fresh Sandbox with a rebuilt installer:
unloading the bindings during orderly shutdown, backfilling the missing standard handles with the
`NUL` device, and upgrading the module. The accepted fix ends an orderly shutdown with
`TerminateProcess`, which skips every DLL's detach handler, with a normal-exit fallback when the
binding is unavailable. It is recorded as an architecture decision with the rejected alternatives,
and scenario A asserts a clean exit without console stdio.

**A migration that reset user state, hidden by its own test.** Scenario L failed, twice, on the
stable localhost port: the legacy port was replaced by a newly bound one. Investigation showed the
loss was wider. Provider modes and onboarding state were also being reset to defaults. The cause was
a single field name: the packaged lifecycle process emits `{ ok, result }`, and the startup migration
read `response.data`, so every successful migration was classified as failed and its results were
overwritten with constructor defaults. Every test passed because the stub returned both fields and
was cast in a way that disabled type checking. The fix typed the reader against the real response
union, so a future divergence is a compile error. With the stub corrected, two existing tests failed
before the production fix, which is what a regression test is supposed to do. The scenario's own
assertion was also strengthened: it had checked that the state file existed, and now compares values.

**An updater that failed open.** In scenario K, the third-party updater's Authenticode check
accepted an unsigned installer nondeterministically when its PowerShell verification helper could
not run. The adapter now verifies the downloaded installer's signature itself, retrying tool
startup and refusing on mismatch, unsigned files, or persistent tool failure, before any installer is
reported as downloaded.

**A misdiagnosis, retracted in writing.** The first Phase 1G attempt launched scenario L three
times, produced no evidence, and concluded that Windows Sandbox was broken at the platform level.
The cause was an unquoted configuration path containing a space, which silently handed the Sandbox
executable two arguments so it never loaded the configuration. The supporting observations from that
triage were real and incidental. No repair was needed or performed. The wrong diagnosis is kept in
the phase record, marked superseded, so a later reader does not repeat it.

Smaller findings that changed the harness rather than the product: a packaged instance left running
held the single-instance lock, so a launch quit with exit 0 having run nothing and was recorded as a
pass, and the runner now treats a zero exit without a harness verdict as a failure. Credentials had
been seeded with a tool that writes UTF-16, an encoding the vault correctly refuses, so the harness
was testing a shape no real installation contains. An update-scenario version had been hard-coded
and could have passed while exercising only a downgrade refusal.

## What qualification does not establish

- It is evidence about one exact unsigned development artifact. It is not production-release
  validation.
- Windows Sandbox is Windows 11 x64 build 26100. Windows 10, ARM64, and real pre-Windows-11
  hardware were not tested; scenario I's refusal is mock-driven and labelled as such.
- It does not test live provider traffic, signing with a production certificate, or a real update
  endpoint, because none of those exist yet.
- Passing assertions measure the declared contracts. They are not proof of correctness for
  situations the matrix does not describe.

---

Previous: [11 — Windows desktop application](11-windows-desktop-application.md) · Next: [13 — Licensing, updates, and signing](13-licensing-updates-and-signing.md)
