# 11 — Windows desktop application

[← Back to the case study](../README.md)

![Desktop trust boundary: a sandboxed renderer talks through a typed preload bridge to the Electron main process, which brokers requests to a supervised packaged gateway over loopback with a session secret; provider keys live only in Windows Credential Manager, the offline licence is validated by both the main process and the gateway, and live traffic reaches only two official endpoints](../assets/diagrams/04-desktop-trust-boundary.svg)

The browser dashboard and the gateway were built to run as a developer's local services. Turning
them into something a person installs, launches from a shortcut, upgrades, and uninstalls was four
phases of work, each with its own decision record and its own evidence. The visible product changed
less than the trust model did.

## The trust boundary

```text
renderer (untrusted UI)  →  typed preload bridge  →  Electron main  →  loopback HTTP + session secret  →  gateway child
```

The renderer is sandboxed and context-isolated, with no Node integration and no webviews.
Navigation, window creation, permissions, and any network request that is not the application's
own are denied. The bundle is served from a custom `tokenreader://app` protocol with a content
security policy of `default-src 'none'` and no remote anything.

The preload bridge exposes named methods and nothing else: no `ipcRenderer`, no generic invoke or
fetch, no filesystem, process, shell, environment, gateway address, or session secret. A security
test harness launches the packaged application and asserts the exact bridge surface at runtime, so a
method cannot be added to the bridge without the assertion changing with it.

The main process validates every IPC call: sender identity, main frame, allowed origin, view and
operation allowlists, payload shape and size. Errors returned to the renderer carry stable codes and
safe messages only. Raw parser or cryptographic errors never cross the bridge.

The gateway is the existing Python service, frozen with PyInstaller into a `onedir` bundle with its
migrations and fixtures. The packaging inspection refuses credentials, environment files, databases,
and development artifacts in the bundle. The child receives a one-line JSON bootstrap on stdin
(session secret, data directories, requested port, local SDK token, per-provider mode, and the vault
credentials only for live-enabled providers), binds `127.0.0.1`, reports readiness on stdout, and
requires the session header on `/api` and `/readyz` with a constant-time comparison. Nothing
sensitive passes through argv, the environment, or disk.

Shutdown closes the child's stdin and escalates to terminating the owned process handle only after a
bounded grace period. It never looks a process up by PID, port, or name, so an unrelated process can
never be killed by mistake.

## Credentials and the live-provider boundary

Provider keys live only in Windows Credential Manager, in a `MattieLabs.TokenReader` namespace with
an isolated `.Dev` namespace for development builds. The main process reads and writes them through
a repository-owned adapter over the Unicode Win32 credential APIs. There is no plaintext fallback.
If the vault is unavailable, fixture mode continues and live mode is unavailable.

External SDK traffic authenticates to the local proxy with a persistent, rotatable local SDK token:
`Authorization: Bearer` on the Responses route and `x-api-key` on the Messages route. The token
authorizes nothing else, and the management session secret does not authorize provider routes. The
proxy port is persisted after the first authenticated readiness and reused across launches. A
conflict surfaces a recovery screen with retry-same-port and a confirmed choose-new-port action, and
the occupying process is never inspected or killed.

Live mode is per provider, explicitly confirmed by the user, and forwards only to
`https://api.openai.com/v1/responses` and `https://api.anthropic.com/v1/messages`, with TLS
verification, no redirects, no environment proxies, and zero retries. The client's own credential
headers are never forwarded; the vault key is injected per request. Saving a key never enables live
mode and never triggers an automatic verification request. Saved keys report "Saved — not verified",
because the product has no evidence they work and says so.

## Recovery and privacy operations

A commercial data contract, written before any of this was implemented, fixed the lifecycle of every
data class: where it may live, where it may never go, how it is backed up, and how it is deleted.
The implementation then had to satisfy the contract rather than define it.

**Staged schema upgrade.** An existing database upgrades through a staged candidate with a
verified pre-upgrade backup. If the packaged gateway does not reach authenticated readiness over the
candidate, the upgrade rolls back automatically. A database written by a newer build is refused as
`incompatible_newer_schema` and never modified.

**Backup and restore.** A manual `.tokenreader-backup` is a logical, metadata-only archive with a
SHA-256 digest. It excludes credentials, content, fingerprints, temporary data, and diagnostic
staging. Restore validates the archive in staging (exact two-entry allowlist, bounded size and
ratio, symlink and traversal refusal, checksum) and runs the staged upgrade before anything replaces
the live database. Backups are unencrypted and unsigned, and the product says so.

**Retention and purge.** Bounded retention can be previewed, run, and changed. Local metadata can
be deleted by project, date range, and exact data category after a confirmed preview. Deletion uses
`secure_delete`, WAL truncation, `VACUUM`, and an integrity re-check, with a sentinel-absence test.
Managed backups that would resurrect purged rows are deleted after the database commit; a failed
backup cleanup reports `backup_cleanup_pending` and is retried rather than reported as complete.
Manual backups outside the managed directory are never searched or touched. The product cannot
promise erasure from disk hardware, snapshots, or copies the user made, and it says that too.

**Receipts and diagnostics.** Every privacy operation writes a durable, exportable receipt. A
diagnostic bundle is generated from a field allowlist validated at generation time, previewed from
one captured snapshot, bounded in size, and exported only where the user chooses. There is no upload
path because there is no upload service.

One operating-system-held exclusive lock serialises every destructive operation. Contention returns
`lifecycle_busy` and is never queued.

## The product shell

The shell has seven destinations: Overview, Requests, Budgets, Optimize, Model Guide, Learn, and
Settings. Request Detail nests under Requests. Old routes redirect and unknown routes render a
deterministic not-found state.

Two of the seven are deliberately unfinished in a way that is stated rather than hidden. Model Guide
reuses observed exact model IDs and pricing provenance and labels task-based guidance unavailable.
Learn has the structure of a lessons destination and no fabricated lessons. Both would have been
easy to fill with plausible text. Neither was.

First-launch onboarding offers two equal paths: explore with fixture data, or connect providers.
Nothing external happens on first launch. No provider request, no automatic sample, no automatic
live mode, and no credential prompt unless the provider path is chosen. The explicit fixture sample
is sent by the main process through the authenticated local route with a constant synthetic prompt,
and it is refused while the provider is in live mode, so it can never incur a charge. The onboarding
record lives in schema-versioned main-process runtime state; browser storage is never authoritative.

Settings consolidates general options, providers, privacy operations, licence, and updates. Every
stateful control appears exactly once. Controls that are unavailable in a given mode say so rather
than existing as dead buttons.

## Installer contracts

The installer is per-user NSIS with no elevation, no directory page, and no network. It is unsigned
in development builds and labelled as the development identity `TokenReader Dev`.

| Contract | Behaviour |
|---|---|
| Supported system | Windows 11 (build ≥ 22000) native x64. Anything else gets a plain refusal and exit code 3. Development-only mock flags can force a refusal for the test harness but can never force a pass |
| Desktop shortcut | A user checkbox, default off, remembered in the per-user install key, silent flag `/DESKTOPSHORTCUT=1\|0` |
| Repair | Same-version reinstall preserves all user data, state, credentials, port, onboarding, and the shortcut choice |
| Upgrade | Advances binaries in place. Data, registration, shortcuts, and credentials are preserved. The NSIS upgrade GUID contains no branding and survived the rename for exactly this reason |
| Uninstall, default | Preserves the data root. Application, shortcuts, and registration are removed |
| Uninstall, delete data | A clearly labelled page choice (silent flag `--delete-tokenreader-data`). Removes only the literal canonical data root through a deleter that never enters a reparse point, and reports partial cleanup honestly with exit code 5 |
| Credentials on uninstall | The three Credential Manager targets are always removed, under both choices. Unrelated credentials are untouched |

Every row of that table is a scripted assertion in the qualification matrix, which is where the
contracts stop being documentation and start being evidence.

## What the desktop work did not add

No telemetry, no accounts, no cloud, no auto-start, no background service, no content capture, no
automatic update, no automatic provider request. The list of things that could have been added to
make a desktop app feel finished is long; each one would have relaxed a boundary the product was
built to keep.

---

Previous: [10 — Limitations and next steps](10-limitations-and-next-steps.md) · Next: [12 — Clean-machine qualification](12-clean-machine-qualification.md)
