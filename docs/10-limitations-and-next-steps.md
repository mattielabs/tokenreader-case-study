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
| Quality retention unavailable | No live-provider evaluation was run, so quality-guarded Prompt Trim comparison is unavailable |
| Bounded evaluation | Passing proves declared rule behaviour on 30 fixed synthetic cases; it does not prove semantic equivalence, general model quality, or production savings |
| Approximate counts | Prompt Trim uses an approximate offline counting method; provider-network token counting is not implemented |

## Product boundaries

| Boundary | Why |
|---|---|
| Local, single user | No accounts, roles, SSO, or remote access. Multi-user would need a separate design, not a configuration flag |
| Advisory only | Budgets never block, delay, reroute, retry, or modify a request |
| No automatic prompt changes | Nothing is rewritten without explicit user selection, and nothing is rewritten in the live request path |
| No content, ever | TokenOps cannot show the prompt behind an expensive request. This is the accepted cost of the privacy boundary |
| Narrow provider surface | OpenAI Responses and Anthropic Messages only; no images, audio, files, tools, or batch requests |
| Exact pricing only | An unrecognised model has no price rather than an approximate one |

## Security and operational limits

- SQLite has **no application-level encryption**. Filesystem access reveals metadata.
- Keyed fingerprints disclose repetition. The equality leakage is documented, not eliminated.
- A fully compromised machine, or malicious software running as the same user, is outside the
  protection boundary.
- **No security or accessibility certification** is claimed, and no third-party audit or penetration
  test was performed.
- Docker packaging remains locally unverified. GitHub Actions parses Compose and builds both images,
  but no container or composed stack has been started or exercised anywhere.
- Remote CI has passed all four retained jobs on the current documented source commit. That is build
  and workflow evidence, not deployment or production evidence.
- There is no deployment and no hosted demo.

## Publication status

| Item | Status |
|---|---|
| This case study | Public |
| Application source | Private and unpublished |
| Open-source licence | **Undecided.** Licence selection is a prerequisite for any public software release |
| Release or tag | None |
| Deployment | None |
| Live provider evaluation | Not run |

## What would come next

**Before any public software release.** Select a licence — this is the gating decision and nothing
else moves without it. Decide whether Docker should remain optional packaging; if it does, exercise
the composed stack in a suitable local or controlled environment rather than treating successful CI
image builds as runtime validation.

**To move beyond fixture evidence.** Run the gated live evaluation once, deliberately and within its
caps, to replace `unavailable` with a real quality-retention figure — accepting that the result may
be unflattering, which is the point of running it. Put the gateway in front of a real workload and
find out which findings survive contact with traffic that was not designed to trigger them.

**To learn whether the product is right.** Talk to developers who are actually confused by an LLM
invoice, and find out whether per-request attribution is the thing they wanted or merely the thing
that was buildable. The advisory-only stance is a hypothesis: that people want evidence rather than
enforcement. It has not been tested against anyone's real budget overrun.

**Only after a separate design.** Team or hosted architecture, and a PostgreSQL path, if the local
tool ever earns them. Neither should be inferred from the current design, and neither is a
configuration change.

---

Previous: [09 — AI-assisted development](09-ai-assisted-development.md) · [Back to the case study](../README.md)
