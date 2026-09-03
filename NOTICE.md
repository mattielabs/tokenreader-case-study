# Notice

## What this repository is

A public, documentation-only technical case study describing **TokenReader**, a local,
metadata-only Windows desktop application for LLM usage, estimated cost, advisory budgets, privacy
controls, and Prompt Trim evaluation. TokenReader was developed as TokenOps and carried the working
name Aura for two phases; documents in this repository that quote historical evidence may use those
names where the evidence was recorded under them.

It exists so that a technical reviewer can assess the engineering judgment behind the project
without the source being published.

## What this repository is not

- It is **not** the TokenReader application. No source code, tests, package manifests, lockfiles,
  environment files, runtime configuration, database schemas, migrations, internal scripts, generated
  API contracts, installers, licence or signing material, or build output are included here.
- It is **not** distributable software. Nothing here can be built, installed, or deployed, and no
  instructions for doing so are provided.
- It is **not** an open-source release. See the [rights notice](RIGHTS.md).

## Data shown here is synthetic

Every figure, screenshot, token count, cost estimate, budget, finding, and evaluation result in this
repository is derived from **deterministic synthetic fixtures**.

- No real provider request was ever made.
- No real API key was ever read.
- No customer, employer, production, or personal data appears anywhere in this repository.
- The evaluation corpus consists entirely of invented, public-safe synthetic tickets.
- Qualification evidence quoted here was produced in disposable Windows Sandbox environments with
  networking disabled and no real credential present.

Fixture-backed results are engineering evidence about the implemented system's behaviour. They are
not production evidence, adoption evidence, or measured cost savings.

## Cost figures are estimates

Cost values are calculated from published, dated provider price schedules that were checked on
2026-08-07. They are **estimates, not provider invoices**, and they exclude taxes, credits,
negotiated rates, regional adjustments, unsupported fees, and non-token charges.

## Evaluation results are bounded

The offline evaluation demonstrates that the implemented deterministic checks behaved as declared on
one fixed synthetic dataset. It does not demonstrate semantic equivalence for arbitrary prompts,
general model quality, or savings in production. Live-provider quality retention is unavailable
because no live run was performed.

## Qualification results are bounded

Clean-machine qualification applies to one exact unsigned development installer, identified by hash
in the case study, and to nothing else. It is not a production-release validation, and no
accessibility or security certification is claimed.

## AI-assisted development

The TokenReader implementation was substantially produced with AI coding assistance, under the
direction and review of Alex Mattie, who defined the problem, scope, constraints, privacy rules,
acceptance gates, and release decisions. This is disclosed in full in
[docs/09-ai-assisted-development.md](docs/09-ai-assisted-development.md).

## Third-party names

OpenAI and Anthropic are trademarks of their respective owners. TokenReader is an independent project
and is not affiliated with, endorsed by, or sponsored by either company. Their names appear here only
to identify the provider APIs and official SDKs the project integrates with, and the published
pricing pages used as sources for dated rate schedules. Windows, Electron, and other product names
mentioned are the property of their respective owners and are used for identification only.

## No warranty

This documentation is provided as-is, without warranty of any kind. It describes a qualified
development build with no production deployment, external beta, security or accessibility
certification, or real-user evidence.

## Contact

Alex Mattie — Mattie Labs
[mattielabs.ai](https://mattielabs.ai) · [mattielabs@gmail.com](mailto:mattielabs@gmail.com)
