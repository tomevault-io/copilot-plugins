## openanalytics

> Read this before changing anything. It is short on purpose, and it is the only

# Working in this repository

Read this before changing anything. It is short on purpose, and it is the only
place that carries these rules — Claude Code and most other coding agents load
it automatically, and a human contributor should read it too.

## What this is

OpenAnalytics: privacy-first, self-hostable web analytics. AGPL-3.0. This
repository is the product — collector, worker, query gateway, realtime, API,
dashboard, tracker, and the self-host packaging under `infra/selfhost/`.

**This is a working repository, not a mirror.** Pull requests are reviewed and
merged here, with the contributor's name on the commit. Until August 2026 it
received generated exports from a private monorepo and a merged PR was flattened
by the next one; that is over. The private repository still exists — it holds
the hosted service's commercial half and the operators' own record — but the two
are independent. Nothing syncs automatically in either direction, and porting
between them is done by hand.

One rule about that is not a judgement call: **a security fix goes to both sides
promptly.** Everything else either side may decline.

## What is deliberately not here

Billing, subscriptions, plans, trials, usage metering, the quota gate, Stripe,
and the hosted service's marketing site are **not missing — they were removed on
purpose.** They are the commercial half of the hosted product and they live in
the private repository.

Do not add them back, do not write a replacement "because it looks incomplete",
and do not wire a payment provider into the product layer. A self-hosted install
has no plan, no quota and no expiry, and that is the design.

`pnpm run schema:product` enforces the database half of this in CI: the product
schema must build with none of the commercial tables in it.

## Where decisions are recorded

Not here. The decision record lives in the private repository, and it is not
published.

What that means in practice: **a decision you have to obey is written as a rule
in the document where you would hit it** — this file, `CONTRIBUTING.md`,
`SELF-HOSTING.md` — and not as a record you are expected to go and find. If you
change behaviour that one of those documents describes, change the document in
the same commit. If you find a rule stated in two places and they disagree, that
is a bug worth reporting on its own.

## Rules for changes

- **The contract is the seam.** `apps/web` imports only
  `packages/contracts` — server packages never leak into the frontend.
  `pnpm run boundaries` enforces it and CI runs it first.
- **OpenAPI first.** API changes start in
  `packages/contracts/openapi/openapi.yaml`; `pnpm run contracts:generate`
  regenerates the client and CI fails if the committed client drifts.
- **The tracker has a byte budget.** It runs on other people's sites and its
  size is their cost, paid on every page load. `pnpm run tracker:build` enforces
  the ceiling; a feature that cannot pay for its bytes needs a discussion first.
- **Migrations are forward-only and expand-first.** There are no down
  migrations, by decision: the rollback path is a backup taken before the
  upgrade and a restore to go back — `infra/selfhost/upgrade.sh` and
  `rollback.sh` are that path. A migration that cannot be deployed ahead of the
  code that needs it is a migration to split in two.
- **Releases are tags, and a version number is a promise about upgrading.**
  [RELEASING.md](RELEASING.md) says which changes bump the minor rather than the
  patch. If your change needs an operator to do anything by hand, say so in the
  pull request: that sentence is what the release notes are made of.
- **A behaviour change needs a test that was seen failing first.** A test
  written after the fact, that has never been red, pins nothing.
- **Secrets never reach a log, an error message or a response.** Recipient
  addresses and client IPs are redacted or hashed; the existing code shows the
  shape in every place it matters.

## Before you push

```sh
pnpm install --frozen-lockfile
pnpm run verify              # boundaries, format, lint, OpenAPI, typecheck, tests, tracker budget
node scripts/leak-scan.mjs   # credential patterns; no install needed
```

Both run in CI on every push and pull request, along with integration and
migration suites against real Postgres, ClickHouse and Valkey service
containers. Integration and migration suites need those stores locally too —
`infra/selfhost/docker-compose.yml` brings them up, and the `TEST_*` variables
are in `.github/workflows/ci.yml`.

## If you are an agent

- **Do not add a `Co-Authored-By` trailer.** An assistant in the contributor
  list of an open-source repository is noise at best, and once outside
  contributors exist it cannot be removed without destroying their history. In
  Claude Code: `{"attribution": {"commit": ""}}` in settings.
- **Never force-push `main`.** The history was squashed once, before this
  repository had any contributors, and that was the last time it is possible
  without taking somebody's work with it.
- **Say what you are starting, when you start it.** Two people building the same
  thing in parallel has happened here, and it was caught by luck. A message when
  work begins costs one line; a duplicate implementation costs a migration to
  undo.
- **Prove claims by running them.** A workflow that has never run is not done, a
  metric you have not read is not green, and a comment is not evidence about the
  code under it.

---
> Source: [OpenLabs-so/openanalytics](https://github.com/OpenLabs-so/openanalytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
