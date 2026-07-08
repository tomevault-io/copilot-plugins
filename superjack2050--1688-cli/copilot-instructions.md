## 1688-cli

> This file is the short entry point for coding agents. Treat it as a map, not

# Agent Map

This file is the short entry point for coding agents. Treat it as a map, not
as the full manual. Load deeper context from `ARCHITECTURE.md` and `docs/`
only when the task needs it.

## First Read

- Agent working principles: `docs/AGENT_WORKING_PRINCIPLES.md`
- System architecture: `ARCHITECTURE.md`
- Documentation map: `docs/README.md`
- Default workflow: `docs/WORKFLOW.md`
- Command catalog: `docs/COMMANDS.md`
- JSON contracts: `docs/JSON_CONTRACTS.md`
- Safety and approval rules: `docs/SAFETY.md`
- Reliability and browser/session behavior: `docs/RELIABILITY.md`
- Quality score and known gaps: `docs/QUALITY_SCORE.md`

## Project Shape

- `src/cli.ts`: Commander CLI surface and command routing.
- `src/commands`: command executors and human renderers.
- `src/session`: Playwright browser/session helpers, mtop capture, locators,
  recovery, lock, state, and artifacts.
- `src/daemon`: background daemon client/server/protocol/throttle.
- `src/io`: JSON/text output, prompts, and structured errors.
- `src/auth`: login/session verification and cookie helpers.
- `src/util`: small shared utilities.
- `tests`: deterministic Vitest coverage and fixtures.
- `scripts`: probes, postinstall, and agent-map utilities.
- `docs`: canonical agent-readable knowledge base.

## Common Commands

- Install deps: `pnpm install`
- Typecheck: `pnpm typecheck`
- Deterministic tests: `pnpm test:unit`
- Full test run, including live doctor checks: `pnpm test`
- Build CLI: `pnpm build`
- Generate agent indexes: `pnpm agent-context`
- Check generated indexes: `pnpm docs-check`
- Check map structure: `pnpm agent-map-check`
- Default agent gate: `pnpm agent-verify`

`pnpm agent-verify` is the default green gate. If it fails, report the exact
failure instead of hiding it.

## Task Routing

- New or changed CLI command: read `docs/playbooks/add-command.md`,
  `docs/COMMANDS.md`, and the relevant `src/commands/*` file.
- JSON output change: read `docs/playbooks/change-json-output.md` and
  `docs/JSON_CONTRACTS.md`; preserve compatibility unless the user approved a
  breaking change.
- Browser risk-control / slider / login issue: read
  `docs/playbooks/debug-risk-control.md`, `docs/SAFETY.md`, and
  `docs/RELIABILITY.md`.
- New mtop or response capture: read `docs/playbooks/add-mtop-capture.md` and
  existing helpers in `src/session/*capture*.ts`.
- Release/update behavior: read `docs/playbooks/update-cli-release.md`.
- Sourcing research features: read `docs/specs/sourcing-research.md`
  and `docs/FEATURES.md`.
- Supplier search/research work: read `docs/specs/supplier-search.md`.
- Seller IM work: read `docs/specs/seller-im.md`.
- Checkout/order work: read `docs/specs/checkout-and-orders.md` and
  `docs/SAFETY.md`.
- Complex feature/refactor: create or update an ExecPlan under
  `docs/exec-plans/active/`.

## 1688 Runtime Rules

- For 1688 sourcing, products, orders, or logistics, use the `1688` CLI.
- It outputs JSON automatically when stdout is piped, so
  `1688 <cmd> | jq` works.
- At the start of a multi-command 1688 session, run
  `1688 doctor --no-launch --json` and start the daemon when useful with
  `1688 daemon start`.
- If a command returns exit `3`, tell the user to run `1688 login`; do not
  retry in a loop.
- If a command returns exit `4`, tell the user to run the same command once
  with `--headed` and solve the slider manually; do not silently retry.
- `1688 login` opens a user-interactive QR/browser flow. Run it only when the
  user explicitly asks to log in.
- `1688 logout --yes` requires explicit current-turn confirmation.
- Seller messages `--watch` is long-running and emits one JSON object per new
  message.

## Write-Action Boundaries

These commands contact sellers, modify buyer state, or place orders. Use the
protocols in `docs/SAFETY.md`.

- `1688 seller inquire <offerId> <message>`
- `1688 seller chat <orderId|loginId> <message>`
- `1688 cart add <offerId> --sku <skuId> --qty N`
- `1688 cart remove <cartId>`
- `1688 checkout confirm <cartIds...>`
- `1688 feedback "<message>" --submit`
- `1688 logout --yes`

Hard rule: never run `1688 checkout confirm ... --agent` unless
`1688 checkout prepare <cartIds...>` was shown to the user and the user gave
explicit current-turn approval to place the order.

## Done Criteria

- Relevant docs/playbooks/specs were read and updated if behavior changed.
- `pnpm agent-context` was run after command, JSON contract, source layout, or
  test layout changes.
- `pnpm agent-verify` was run, or the exact blocker is recorded.
- For complex work, active ExecPlans contain progress, decisions, and latest
  verification.

---
> Source: [superjack2050/1688-cli](https://github.com/superjack2050/1688-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
