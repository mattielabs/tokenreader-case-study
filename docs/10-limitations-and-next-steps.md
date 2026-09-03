# 10 — Limitations and next steps

[← Back to the case study](../README.md)

Most of these limitations are consequences of choices made deliberately, not gaps waiting to be
patched. They are listed together so a reviewer does not have to hunt for them.

## Evidence limitations

| Limitation | Detail |
|---|---|
| Fixture-backed, not production | Every figure comes from deterministic synthetic fixtures. No real provider request was made and no real key was read |
| No users or adoption | There are no customers, no usage, no retention data, and no measured savings |
| Estimates, not invoices | Costs exclude taxes, credits, negotiated rates, regional adjustments, unsupported fees, and non-token charges |
| Live mode never exercised | Live forwarding is implemented and gated, and has never been run against a real provider |
| Quality retention unavailable | No live-provider evaluation was run, so quality-guarded Prompt Trim comparison is unavailable |
| Bounded evaluation | Passing proves declared rule behaviour on 30 fixed synthetic cases; it does not prove semantic equivalence, general model quality, or production savings |
| Approximate counts | Prompt Trim uses an approximate offline counting method; provider-network token counting is not implemented |
| Qualification is per artifact | The Sandbox evidence applies to one exact unsigned development installer. Any product change voids it and requires a full rerun |

## Product boundaries

| Boundary | Why |
|---|---|
| Windows 11 x64 only | No Windows 10, ARM64, macOS, or Linux. The unsupported-system refusal is tested with mock hooks, not on real older hardware |
| Local, single user | No accounts, roles, SSO, or remote access. Multi-user would need a separate design, not a configuration flag |
| Advisory only | Budgets never block, delay, reroute, retry, or modify a request |
| No automatic prompt changes | Nothing is rewritten without explicit user selection, and nothing is rewritten in the live request path |
| No content, ever | TokenReader cannot show the prompt behind an expensive request. This is the accepted cost of the privacy boundary |
| Narrow provider surface | OpenAI Responses and Anthropic Messages only; no images, audio, files, tools, or batch requests |
| Exact pricing only | An unrecognised model has no price rather than an approximate one |
| Portable licence | A licence binds to nothing, so a shared licence file works anywhere. That is an accepted commercial risk, not an oversight |
| No binary rollback | The updater cannot roll a binary back safely, so the product does not claim it. Data integrity relies on the staged migration and its backup |

## Security and operational limits

- SQLite has **no application-level encryption**. Filesystem access reveals metadata.
- Backups are metadata-only, **unencrypted, and unsigned**. The digest detects corruption, not a
  same-user actor.
- Keyed fingerprints disclose repetition. The equality leakage is documented, not eliminated.
- A fully compromised machine, or malicious software running as the same user, is outside the
  protection boundary.
- Development builds are **unsigned**. The current qualified binary is one of them.
- **No security or accessibility certification** is claimed, and no third-party audit or penetration
  test was performed.
- Docker packaging remains locally unverified and is not the supported path.
- There is no deployment, no hosted demo, no release, no tag, and no external beta.

## Publication status

| Item | Status |
|---|---|
| This case study | Public, renamed to `tokenreader-case-study` on 2026-09-03 |
| Application source | Private and unpublished, under its original repository name |
| Open-source licence | **None.** The source is proprietary and no reuse licence is granted |
| Application licence | Contract and tooling implemented; no production key exists and no licence has been issued |
| Code signing | Pipeline ready; no production certificate |
| Release, tag, deployment, external beta | None |
| Live provider evaluation | Not run |

## What would come next

**Before external beta.** Obtain the code-signing certificate, generate the production licence and
update keys offline, register the update domain, and create the releases repository. Each is a
decision with a cost attached, which is why none of them was made inside an engineering phase. Then
build the signed binary, run the full Sandbox matrix against that exact artifact, and repeat the
manual review, because the current qualification covers an unsigned build and nothing else.

**To move beyond fixture evidence.** Enable live mode once, deliberately, with a licence and a real
key, against a small real workload, and find out which findings survive contact with traffic that was
not designed to trigger them. Run the gated live evaluation within its caps to replace `unavailable`
with a real quality-retention figure, accepting that the result may be unflattering. That is the
point of running it.

**To learn whether the product is right.** Talk to developers who are actually confused by an LLM
invoice, and find out whether per-request attribution is the thing they wanted or merely the thing
that was buildable. The advisory-only stance is a hypothesis: that people want evidence rather than
enforcement. It has not been tested against anyone's real budget overrun.

**Carried forward from qualification.** Audit the test stubs that are cast in ways that disable type
checking. One of them hid a real migration defect for two Sandbox runs. The phase record names the
pattern so it is not lost.

**Only after a separate design.** Team or hosted architecture, a PostgreSQL path, and any platform
beyond Windows 11 x64, if the local tool ever earns them. None should be inferred from the current
design, and none is a configuration change.

---

Previous: [09 — AI-assisted development](09-ai-assisted-development.md) · Next: [11 — Windows desktop application](11-windows-desktop-application.md)
