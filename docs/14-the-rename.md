# 14 — The rename

[← Back to the case study](../README.md)

The product was built as TokenOps. The dashboard re-skin and the licensing phase carried the working
name Aura. Neither was final, and two identities in flight had a real cost: an installation created
under an old name must not be stranded, duplicated, or merged when the new one arrives. Phase 1G
made **TokenReader** the only active identity in one controlled migration, and it treated the job as
what it is: a data migration with a text change attached, not the other way around.

## Inventory before editing

The tracked tree held 349 files. 246 of them contained at least one old-brand token, about 1,630
occurrences in total, across the gateway, scripts, docs, desktop shell, dashboard, issuer tools,
fixtures, and the repository root. The identity literals were enumerated individually: application
ID, custom protocol, NSIS upgrade GUID, installer and product names, data roots, database filename,
three credential namespaces, thirty environment variables, six request headers, five file
extensions, the Python package, the frozen binary name, the licence and manifest product IDs, two
trust-root key IDs, the planned releases repository, and the design-system identifiers.

Every occurrence was then classified before any edit:

| Class | Treatment |
|---|---|
| Active identity | Rename |
| Compatibility or migration adapter | Preserve intentionally, test, and allowlist with a reason |
| Historical evidence | Preserve without rewriting: phase records, the build log, ADRs 0001–0021, qualification hashes |
| Third-party terminology | Preserve: OpenAI, Anthropic, Electron, NSIS, and the rest |
| Generated artifact | Regenerate from renamed source: API contract, demo snapshot, SBOMs, provenance, bundles |
| Ambiguous | Resolve before editing |

Two ambiguous items were resolved by keeping them. The NSIS upgrade GUID contains no branding and is
the mechanism by which the installer recognises an existing installation; changing it would create a
second Apps & Features entry and a second installation rather than an upgrade. The migration revision
identifiers carry the old name and were kept because renaming them would invalidate migration
history for every existing database.

A raw search-and-replace was rejected explicitly. The text migration ran as 29 ordered rules at byte
level, most specific identifier first, with the factual repository names shielded from the generic
rules and historical documents excluded by an explicit file list rather than a directory glob. It
touched 224 files and 1,371 lines. Five blanket-rule artifacts were then corrected by hand.
Historical records received a short header note stating that the product was later renamed; their
evidence, hashes, and terminology are untouched.

## Migrating installed users

An installation under the old name has a data root, a database, a stable proxy port, onboarding
state, provider modes, three Credential Manager entries, managed backups, receipts, and possibly a
cached licence. All of it had to arrive under the new name, once, without a support call.

**Data root.** One versioned, idempotent migration runs before the gateway starts, inside the
one-shot lifecycle process that already holds the operating-system lifecycle lock. The desktop main
process, the only path authority, supplies candidate legacy roots; nothing is discovered by scanning.
The legacy root is copied through a staging directory, the database is renamed and verified by size,
and the result is promoted only after verification. A populated TokenReader root plus a populated
legacy root is `recovery_required`: both are left untouched and no completed record is written. The
migration never merges and never overwrites.

**Credentials.** Exactly the three known slots are read at their exact legacy target names; the
Credential Manager is never enumerated. Each value is written to the new namespace and read back
before the legacy target is considered replaceable. Conflicting values are refused, never resolved.
A failed write is rolled back rather than reported as success.

**Rollback material.** The legacy root and the legacy credential targets survive until the
TokenReader gateway reaches authenticated readiness. Only then are they released. Any failure before
that point leaves a complete, working legacy installation behind.

Three compatibility surfaces were kept, each narrow, tested, and allowlisted with a reason. The old
project header is accepted as a deprecated alias with identical validation, and a request that sends
both headers with different values is refused rather than resolved. Legacy backup archives restore
through the normal staged, hash-verified path, and the receipt records the legacy source so a legacy
import is never presented as an ordinary one. A cached Aura licence found during migration is carried
across under the new filename specifically so the application reads it and **refuses** it, instead
of reporting "no licence imported". Silently converting a licence across product identities would
have undermined the licence contract, so an Aura licence must be reissued.

Fresh development-only Ed25519 keypairs were generated offline and both registries rotated. The
previous development keys are gone from the tree, so an old licence or manifest is refused on two
independent grounds: unknown key ID and wrong product ID. Production registries remained empty
throughout.

## Gating the result

A new active-branding gate scans every tracked file, with no directory exempted wholesale, plus the
built surfaces when present (packaged dashboard, production and demo bundles, release bundle, SBOMs,
provenance) and the names of built artifacts. Historical documents are exempted only by explicit,
reasoned file entries. An allowlist entry that stops matching is itself a failure, so a compatibility
surface cannot outlive the code that justifies it. The gate runs in the source validation pass and
again after packaging.

New tests covered the migration end to end: twelve for the data-root migration, nineteen for the
naming contract across all four packages, seven for the header alias, three legacy-archive cases,
eleven for the credential migration, and nine for startup orchestration. The gateway suite rose from
318 to 359 and the desktop suite from 203 to 223, later 226 after the qualification fix described
below.

## What qualification then found

Two Sandbox scenarios joined the permanent matrix. Scenario L installs the actual pre-rename build,
lets it seed its own data root, port, credentials, and a valid Aura licence, then upgrades with the
TokenReader installer and asserts the migration end to end. Scenario M is a fresh installation on a
machine that never had the old name.

Scenario L failed twice. The stable port was replaced by a newly bound one, and closer inspection
showed provider modes and onboarding state were also being reset. The migration itself was correct.
The startup code that read its result used the wrong field name for the lifecycle response
envelope, so every successful migration was classified as failed and its results were discarded.
The tests had passed because the stub returned both field names and was cast in a way that disabled
type checking. Before that, the validation gates had already caught the application minting a fresh
local SDK token in the new namespace before the credential migration ran, which produced an
unresolvable conflict that would have blocked startup for every upgrading user. The token is now
minted only after the migration has adopted any legacy credential.

The fixed build passed the full matrix: **215 of 215 assertions across 12 applicable scenarios**,
with scenario L at 43 of 43 and its decisive line reading `legacy=49675 current=49675`, the exact
port that had shifted in both earlier runs. The manual review confirmed that Apps & Features showed
exactly one entry, `TokenReader Dev 0.3.0`, with no TokenOps or Aura entry beside it.

## What was deliberately not renamed

The private source repository keeps its original name, `mattielabs/tokenops`, and the source
repository's own documentation continues to name it accurately. This public case study was renamed
on 2026-09-03; the old URL redirects. Registering the update domain, creating the releases
repository, and obtaining the signing certificate remain deferred with the rest of the external-beta
work.

---

Previous: [13 — Licensing, updates, and signing](13-licensing-updates-and-signing.md) · [Back to the case study](../README.md)
