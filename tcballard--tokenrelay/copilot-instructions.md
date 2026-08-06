## tokenrelay

> - Install: `pnpm install`

# TokenRelay agent guide

## Commands

- Install: `pnpm install`
- Full check: `pnpm check`
- Tests: `pnpm test`
- Start PostgreSQL: `docker compose up -d postgres`
- Migrate: `pnpm db:migrate`
- API development: `pnpm dev:api`

## Invariants

- The canonical traded unit is an accepted deliverable for a fiat-denominated task reward. Never describe allowance, model tokens, runs, subscription capacity, prompts, or compute as purchasable marketplace inventory.
- Submission alone MUST NOT create provider earnings. Settlement follows explicit acceptance or an accepted dispute resolution.
- Provider executors and credentials remain provider-controlled. The marketplace records obligations and attestations but does not certify or operate the executor.
- `TokenRelay` is the owner-approved product name. Use it on human-facing
  surfaces and use `tokenrelay` as the canonical CLI command. Retain the
  `agentexchange` CLI alias, `tokenrelay.*` protocol namespace, and
  `TOKENRELAY_*` environment variables during the v0.1 compatibility window;
  removing those interfaces requires an explicit protocol migration.

- Version 0.1.0 uses simulation funding; payment-provider code and live settlement remain disabled.
- Never activate live payments, live payouts, private repositories, or public matching without a new owner-approved release gate written for the outcome marketplace. The older v1/v2 gate documents are historical and superseded.
- Marketplace money uses bigint minor units internally and decimal strings in JSON. Never use floating point for accounting.
- `TOKENRELAY_MARKET_MODE=simulation` MUST NOT enable Stripe, paid leasing, or live execution.
- Never add provider credentials, provider-quota brokerage, or automatic merge or deploy. The GitHub App may push a branch and open a pull request on a buyer's own installed repository (a proposal the buyer reviews); it MUST NOT merge, deploy, force-push, or write to a default branch. The buyer's merge remains the act of applying a deliverable.
- Workers receive only public, allowlisted repositories at exact commit SHAs.
- A deliverable is untrusted until mechanical validation and buyer review succeed.
- Patches come from Git state, never runner stdout.
- Ledger entries are append-only and each transaction balances to zero.
- Simulation balances, buyer balances, and provider earnings are distinct account classes and MUST NOT be converted implicitly.
- State transitions and their events are transactional.
- The public API and private control worker share domain/database packages and MUST NOT call each other over private HTTP.
- PostgreSQL is the initial queue and coordination boundary. Durable work uses the transactional outbox and `SKIP LOCKED`; do not add Redis or an in-memory-only scheduler.
- Every paid lease path uses the central leasing gate. Rematching creates a new generation and stale lease tokens are rejected.
- The API/control plane stores bounded patches but MUST NOT clone or execute repository code.
- Machine output goes to stdout; diagnostics go to stderr.
- Tests must not require a model, provider account, or internet access.

Work on a feature branch and open a draft pull request. Do not commit directly to `main`.

---
> Source: [tcballard/TokenRelay](https://github.com/tcballard/TokenRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
