# 07 — Testing and validation

[← Back to the case study](../README.md)

All figures below were recorded against source commit
`1f174203c363e43757010a9af703c04dd7b148b1` on 2026-08-28, on Windows.

## The repository gate

One validation entry point runs the full static gate: PowerShell syntax, Python formatting and lint,
strict type checking, a secret and tracked-artifact scan, the active-branding gate over tracked
source, evaluation-drift and snapshot-drift checks, a static-demo scan, a generated-API-contract
drift check, an isolated migration cycle, the gateway test suite, clean dependency installs and
audits, strict TypeScript across three packages, the dashboard, desktop, and issuer test suites, the
connected and static-demo builds, and then the full desktop validation: packaging, the packaged
smoke and security harnesses, the licence and update harnesses, the signing inventory, the release
bundle and its inspection, and the active-branding gate again over the built surfaces.

| Check | Result |
|---|---|
| Gateway tests | **359 passed** |
| Dashboard tests | **164 passed**, including axe scans over every destination and state and the two-theme contrast gate |
| Desktop tests | **226 passed** |
| Issuer tool tests | **8 passed** |
| Strict mypy | passed |
| Ruff format and lint | passed |
| Strict TypeScript | passed across dashboard, desktop, and issuer tools |
| Migrations | upgrade, downgrade, and re-upgrade passed; head at `0005_privacy_operations` |
| Dependency audits | pip-audit and npm audit clean across dashboard, desktop, and issuer tools |
| Builds | connected, static-demo, and packaged desktop builds passed; unsigned NSIS installer produced |
| Frozen-gateway scenarios | passed with no network, including licensed live fail-closed and unlicensed downgrade |
| Electron security harness | 15 / 15 |
| Packaged smoke | 13 / 13, including the empty-browser-storage assertion |
| Packaged licence lifecycle and loopback update harnesses | passed |
| Signing inventory | development build correctly reported `NotSigned` |
| Test-certificate signing validation | passed |
| Release bundle | generated and inspected; the release-mode gate correctly refused the development-trust bundle |
| Scans | secret, evaluation, snapshot, generated-contract, static bundle, packaging, qualification bundle, and active-branding over source and built surfaces passed |

Runtime and browser workflows are kept separate from the static gate because they start isolated
local services.

## What the tests actually exercise

**Real HTTP, real SDKs, deterministic upstream.** The four SDK paths (OpenAI regular, OpenAI
streaming, Anthropic regular, Anthropic streaming) run the official Python SDKs against the gateway
over actual localhost HTTP, not an in-process test client. The upstream is a deterministic fixture
serving recorded provider JSON and SSE.

This is the design choice that gives the numbers their meaning. The SDK does its real content
negotiation, header handling, streaming parse, and error handling; only the provider's
non-determinism and its bill are removed.

Coverage includes:

- provider usage normalization for both providers, including cached, reasoning, and cache-write
  categories, and the partial, missing, and invalid classifications;
- exact price-schedule matching, including deliberate zero-match and ambiguous-match cases;
- cost calculation, independent component rounding, and total reconciliation;
- streaming finalization, including interrupted streams that must retain only usage already observed;
- advisory budget thresholds at 50%, 80%, and 100%;
- all six finding types against their configured thresholds;
- Prompt Trim safety regressions across every protected structure;
- the full 30-ticket offline evaluation;
- the live-evaluation protocol and its gates, against controlled localhost fake upstreams only;
- Host, Origin, request-size, and defensive-header hardening, with deterministic error responses;
- log, database, sidecar, management-response, export, backup, and diagnostic privacy scans;
- migration upgrade, downgrade, re-upgrade, and preservation of data written by earlier versions;
- the staged upgrade coordinator, backup and restore, retention, purge, receipts, and the lifecycle
  lock;
- the licence contract in both runtimes, with a cross-language fixture, tampered and truncated
  packages, and production trust refusing development keys;
- the update state machine, manifest verification, version policy, and the network allowlist;
- the legacy data-root and credential migrations, the naming contract across all four packages, and
  the deprecated header alias;
- the desktop trust boundary: preload surface, IPC validation, supervisor lifecycle, and credential
  vault behaviour;
- static snapshot generation, drift detection, bundle scanning, and hash-route behaviour.

## Runtime acceptance

A separate runtime pass starts the fixture, gateway, and dashboard on loopback against an isolated
database and fingerprint key, then verifies liveness and readiness, runs the four SDK paths, confirms
that credentials and internal headers stop at the gateway, checks that complete usage and cost
records persisted, and runs privacy scans across the database and its sidecars, the log files,
management responses, and exports. It ends by stopping every service and confirming the ports are
closed.

The desktop equivalent launches the packaged application in isolated data roots and credential
namespaces on the host, runs the smoke, security, licence, and update harnesses, and asserts a clean
exit with no orphan process or listener. Host launches are not clean-machine evidence; that is what
the [Sandbox qualification](12-clean-machine-qualification.md) is for.

## Browser and accessibility review

Desktop (1440×900) and minimum-width mobile (320×800) layouts were reviewed in both themes. At narrow
widths the document scroll width equalled its client width, confirming there was no page-level
horizontal overflow; wide tables and the pill strip keep their own deliberate scroll containers.
Dashboard tests cover loading, empty, error, chart geometry, budget, optimization, Prompt Trim,
selected filter state, focus management, reset states, the command palette, the shell navigation
structure, an axe-core scan of every destination and state, and a contrast gate that parses the
design tokens and asserts WCAG AA for every informational pair in both themes.

![The light Overview at narrow width, showing the stacked evidence cards and the wrapped pill navigation without page-level horizontal overflow](../assets/screenshots/demo-mobile.jpg)

The packaged application was additionally reviewed by hand in an interactive Windows Sandbox:
keyboard-only operation, Narrator, display scaling from 100 to 200 percent, high contrast, reduced
motion, and the installer and uninstaller. This is **tested and reviewed behaviour, not an
accessibility certification**. No formal WCAG conformance audit was performed.

## Drift protection

Four artifacts are generated and checked rather than hand-maintained: the offline evaluation
summary, the static demo snapshot, the API contract, and the branding allowlist's correspondence
with the code. Each has a check mode that fails on drift. The evaluation numbers quoted in this case
study cannot go stale without the build failing first, and a compatibility surface left behind by
the rename cannot outlive the code that justifies it.

## Remote CI

GitHub Actions ran the pinned workflow successfully on the Week 4 source commit, before the desktop
phases began, with backend, frontend, POSIX direct-runtime, and Docker image-build jobs. The Docker
job parsed the Compose configuration and built the images; it did not start a container.

The desktop build, packaging, and Sandbox qualification are Windows-only and were verified locally.
This case study does not cite CI results for commits after Week 4.

## What validation does not establish

- **It is fixture evidence, not production evidence.** No real provider request was made and no real
  key was read.
- **It is not a security audit.** No third-party review or penetration test was performed.
- **It is not an accessibility certification.**
- **Qualification applies to one exact unsigned artifact.** Any product change voids it.
- **Docker remains locally unverified.** Images built in CI once and Compose parsed, but no container
  or composed stack has been started or exercised anywhere.
- **There is no production or real-user evidence** of any kind.
- Test counts measure coverage of intended behaviour. They are not proof of correctness for inputs
  outside the fixture set.

---

Previous: [06 — Privacy and security](06-privacy-and-security.md) · Next: [08 — Static recruiter demo](08-static-recruiter-demo.md)
