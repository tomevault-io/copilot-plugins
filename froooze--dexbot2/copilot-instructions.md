## dexbot2

> **Pipeline: `test` → `dev` → `main`** (ONE DIRECTION ONLY!)

# Development Context - DEXBot2

## Branch Strategy
**Pipeline: `test` → `dev` → `main`** (ONE DIRECTION ONLY!)

- **test**: Primary development branch (where work happens)
- **dev**: Integration/staging (merged from test)
- **main**: Production-ready (merged from dev)

⚠️ **KEY RULE**: Always merge **test → dev**, NEVER dev → test
⚠️ **KEY RULE**: See **Absolute Git Action Gate** below for all write-action authorization rules.
⚠️ **KEY RULE**: Default to manual merge/push flow for branch promotion when requested, unless the user specifically asks to use one of the sync scripts.

## Absolute Git Action Gate (User-Directed Writes)

**Agent must NOT proactively ask for or execute git write actions.**
The agent only runs git write actions when the user clearly requests them.

Git write actions include:
- `git add`
- `git commit`
- `git commit --amend`
- `git reset` (any mode)
- `git rebase`
- `git merge`
- `git push`
- `git tag`
- `git checkout` / `git switch` to another branch

Branch-promotion scripts (use force-push, not merge):
- `npm run ptest` — push local `test` to `origin/test`
- `npm run pdev` — force-push `test` → `origin/dev` (skips merge)
- `npm run pmain` — force-push `test` → `origin/dev` **and** `origin/main` (skips dev staging and merge entirely)

Read-only git commands are always allowed (for example: `git status`, `git diff`, `git log`, `git show`).

Interpretation rules:
1. If a user clearly asks for a git write action, execute it.
2. Short approvals like "yes", "ok", "do it", or "go ahead" are valid confirmation when they clearly refer to the immediately previous proposed action.
3. If wording is ambiguous, ask one clarifying question before running destructive actions.
4. `git commit --amend` is allowed when explicitly requested by the user.
5. Before a git write action, restate the user authorization in one short line.

See `docs/WORKFLOW.md` for detailed workflow guide.

## Commit Quality Standard
When creating commits, prefer high-context commit messages for non-trivial fixes/features.

- **Subject**: concise conventional prefix (`fix:`, `feat:`, `docs:`) with clear scope.
- **Body required for substantial changes**: explain **why**, not only what changed.
- **Structure**:
  1. Short problem statement/context
  2. Per-fix sections with file path(s) and behavioral impact
  3. Risk/edge-case notes when relevant
  4. Validation/testing notes (commands or scenario checks)
- **Formatting**: use readable markdown headers/bullets in commit body for scanability.
- **CLI formatting safety**:
  - Never use `/n` or literal `\\n` text as a newline placeholder in commit/PR bodies.
  - Always pass real newlines to Git/GitHub (multi-line body), not escaped newline text.
  - Prefer heredocs for reliability when using `git commit` and `gh pr create`.
- **Atomicity**: keep unrelated edits out of the commit; document only included changes.
- **Tone**: use professional, concise language — no personal commentary, verbose explanations, or markdown section headers in commit bodies. Stick to the facts: what changed and why.
- **No personal information in commits, PRs, or docs**: never include real account names, bot names, market/pair names (e.g. live trading pairs), addresses, or other real identifiers in commit messages, PR titles/bodies, release notes, CHANGELOG, README, or any documentation. Describe incidents generically (e.g. "a live market-pair bot") and use placeholders (e.g. `1.2.x`, `account-name`, `<market-pair>`) in examples. Before committing, scan the message for identifiers that slipped in; if one lands in an existing commit, amend (or follow up) to strip it. This applies to all new content and existing entries being edited.
- **Unrelated working-tree files**: if the diff includes pre-existing dirty files the user wants in a separate commit, use `git add <scope>` to stage only the relevant files. NEVER run `git checkout -- <file>` or `git restore <file>` on those files — that destroys the working-tree content. Let them stay dirty for a future commit.

Recommended CLI patterns (newline-safe):

```bash
# Commit message with proper markdown/newlines
git commit -F- <<'EOF'
fix: <short summary>

<context>

## <Fix area>
- Problem:
- Impact:
- Solution:

## Testing Notes
- <test command>
EOF

# PR body with proper markdown/newlines
gh pr create --title "<title>" --body-file - <<'EOF'
## Summary
- <item>

## Testing
- <command>
EOF
```

## Key Files

### Entry Points
- `dexbot.ts` - Main CLI entry point
- `bot.ts` - Alternative bot starter
- `pm2.ts` - PM2 process management
- `unlock.ts` - Single-prompt starter (no PM2)
- `credential-daemon.ts` - Credential daemon

### Core Bot
- `modules/dexbot_class.ts` - Core bot class, lifecycle orchestration, and shared runtime wiring
- `modules/dexbot_fill_runtime.ts` - Fill processing runtime and replay-safe accounting helpers
- `modules/dexbot_maintenance_runtime.ts` - Maintenance runtime for sync loops and grid checks
- `modules/constants.ts` - Centralized configuration and tuning parameters
- `modules/bitshares_client.ts` - BitShares connection and node management
- `modules/node_manager.ts` - Multi-node health checking and failover
- `modules/fund_registry.ts` - Shared-account fund registry with cross-bot invariants
- `modules/settings_merge.ts` - Consolidated settings merge (single source of truth)
- `modules/cr_planner.ts` - Pure math / credit planning
- `modules/credit_pricing.ts` - Canonical credit-pricing math (offer orientation, conversion rates, CR, fees) shared by runtime and analyzer
- `modules/credit_runtime.ts` - Credit offer and MPA debt lifecycle management

### Order Management (`modules/order/`)
- `manager.ts` - Order lifecycle and state management
- `grid.ts` - Grid calculation, placement, and management
- `working_grid.ts` - Copy-on-write working grid
- `strategy.ts` - Trading strategy (anchor & refill, consolidation)
- `accounting.ts` - Fee accounting and fund tracking
- `sync_engine.ts` - Blockchain synchronization
- `grid_reconcile.ts` - Startup grid reconciliation
- `logger.ts` - Order logging
- `utils/` - Utilities (math, order, system, validate)

### Market Adapter (`market_adapter/`)
- `market_adapter.ts` - AMA delta threshold, grid price offset, and recalc triggers
- `core/market_adapter_service.ts` - Price adapter service core (offset, bound clamping)
- `ama_signal_runner.ts` - AMA signal processing
- `inputs/` - Kibana and LP data sources
- `candle_utils.ts` - Candlestick utilities

### Blockchain Interaction
- `modules/chain_orders.ts` - Blockchain order operations
- `modules/account_orders.ts` - Account order queries

### Configuration
- `profiles/bots.json` - Bot configuration
- `profiles/general.settings.json` - Global settings (auto-generated on first run; file may not exist until first launch)
- `profiles/market_profiles.json` - Market-specific settings (AMA params, price offset params; auto-generated on first run)

### Claw Integration (`claw/`)
- `claw/index.ts` - Main export combining all modules
- `claw/skills/` - Agent skill packages (bitshares-guide, launcher-ops, margin-trading, etc.)
- `claw/modules/bitshares_client.ts` / `chain_queries.ts` / `chain_broadcast.ts` / `chain_actions.ts` - Core BitShares runtime (reads, writes, broadcast)
- `claw/modules/claw_bridge.ts` / `claw_catalog.ts` - JSON bridge + command dispatch
- `claw/modules/claw_manifest.ts` - Runtime manifest
- `claw/modules/claw_infra.ts` - Shared runtime infrastructure
- `claw/modules/claw_launcher.ts` - Launcher orchestration (PM2, Docker)
- `claw/modules/dexbot_bridge.ts` / `dexbot_profiles.ts` / `dexbot_credential_client.ts` - DEXBot2 integration bridge
- `claw/modules/credit_runtime_adapter.ts` - Credit runtime lifecycle bridge
- `claw/modules/short_mpa_strategy.ts` / `decision_loop.ts` - High-level strategy
- `claw/modules/position_manager.ts` / `position_health.ts` / `position_discovery.ts` - Position tracking and health
- `claw/modules/feed_price_source.ts` / `kibana_price_source.ts` - Price sources
- `claw/modules/honest_ecosystem.ts` / `liquidity_pools.ts` - HONEST asset helpers

### Vendored Libraries
- `analysis/uplot/` - uPlot v1.6.32 charting library (vendored, no CDN dependency)

### Analysis Tools (`analysis/`)
Research scripts for parameter tuning — output interactive HTML charts, not used in production.
See `analysis/README.md` for full doc and usage examples.
- `analyze_dynamic_weight.ts` / `trend_detection/dynamic_weight_chart_generator.ts` - Dynamic weight research (AMA + Kalman + Hurst/PE)
- `analyze_volatility.ts` / `trend_detection/volatility_chart_generator.ts` - ATR-based symmetric volatility penalty
- `analyze_regime.ts` / `trend_detection/regime_chart_generator.ts` - Hurst + Permutation Entropy regime classification
- `analyze_regime_windows.ts` - Alternate Hurst/PE window configuration testing
- `analyze_kalman.ts` / `trend_detection/kalman_chart_generator.ts` - Kalman velocity/displacement trend state
- `analyze_derivatives.ts` / `trend_detection/derivative_analyzer.ts` - SMA/MACD/RSI signal analysis
- `analyze_risk_profile.ts` - Inventory risk divergence quantile measurement
- `analyze_trade_heatmap.ts` - 2D trade volume heatmap
- `trade_profitability.ts` - Trade PnL from Kibana fill data (LIFO/FIFO)
- `grid_correction_check.ts` - Grid monotonicity regression gate (sell rising / buy falling between consecutive same-direction fills; `npm run analysis:grid-check`)
- `amafitting/` under `analysis/` (`analysis/ama_fitting/`) - AMA parameter fitting (optimizer, LP fetch, convergence calibration)
- `bot_fitting/` - Grid parameter sweep backtests for AMA winners
- `tradingview/` - Standalone TradingView-style HTML chart exporter
- `bot_usage/` - On-chain bot discovery and Kibana query helpers
- `trend_detection/` - Kalman, Hurst, PE analyzers and tests (see trend_detection READMEs)

### Testing
- `tests/` - Comprehensive test suite (unit, integration, scenario tests)

## Version Management

When bumping the version for a release:

1. Update `version` in `package.json` to the new version.
2. Run `npm run version:sync` — syncs all manifest files (`package-lock.json`,
   `claw/package.json`, `claw/runtimes/openclaw-plugin/*.json`,
   `analysis/ama_fitting/package.json`, `analysis/ama_fitting/package-lock.json`,
   `analysis/trend_detection/package.json`, `claw/tests/test_claw_mcp_transport.ts`,
   `docs/README.md`, `docs/DEXBOT_COMPARISON.md`,
   `docs/FUND_MOVEMENT_AND_ACCOUNTING.md`, `docs/EVOLUTION.md`).
3. Add entries to `CHANGELOG.md` covering **all changes since the last
   release** (review `git log <last-tag>..HEAD`), update `docs/EVOLUTION.md`,
   and refresh the EVOLUTION.md footer (commit count, last updated
   version/date).
4. Create GitHub release with title format `vX.Y.Z - Description` (use ` - ` regular dash, not `—` em dash).
5. Commit the version bump + docs.

```bash
# Check what would change (dry-run)
npm run version:check

# Apply version sync
npm run version:sync
```

## Browser-Safe Surface

**Convention: everything is browser-safe unless listed below as Node-only.**
`package.json` "browser" field is the source of truth — always check it before
adding or removing a classification.

**Node-only** (must not be reached from a browser bundle):
`modules/launcher/*`, `modules/dexbot_class.ts`, `modules/dexbot_maintenance_runtime.ts`, `modules/dexbot_fill_runtime.ts`, `modules/dexbot_cow_runtime.ts`, `modules/dexbot_state_recovery.ts`, `modules/dexbot_startup_runtime.ts`, `modules/runtime_settings.ts`, `modules/credential_runtime.ts`, `modules/dexbot_credential_client.ts`, `modules/node_health_cache.ts`, `modules/daemon_node_health.ts`, `modules/process_discovery.ts`, `modules/graceful_shutdown.ts`, `modules/order/logger.ts`, `modules/order/export.ts`, `modules/order/utils/system.ts`, `modules/order/utils/withPoolRef.ts`, `modules/storage/node_adapter.ts`, `modules/paths.ts`, `modules/key_store.ts`, `unlock.ts`, `bot.ts`, `dexbot.ts`, `pm2.ts`, `credential-daemon.ts`, `market_adapter/market_adapter.ts`, `market_adapter/lp_chart_runner.ts`, `market_adapter/ama_signal_runner.ts`, `market_adapter/utils/chain.ts`, `bitshares-native/transport.ts`, `bitshares-native/signing_client.ts`, `bitshares-native/subscriptions.ts`, `bitshares-native/tx/builder.ts`, `bitshares-native/tx/tx_cache.ts`

**Environment detection** — always go through `modules/env.ts`:
```ts
import { isBrowser, hasProcess } from './env';
```
Do not inline `typeof window` / `typeof globalThis.window` / `typeof process` checks.

## Config Caching Trap (Tests)

`modules/config.ts` snapshots `process.env` values **at module-load time**. Any test setting `process.env.X` after a `require()` that transitively loads `config.ts` will have no effect. Fix: set env var at line 1 before any `require()`, or mutate `Config.X` directly after loading it.

---
> Source: [froooze/DEXBot2](https://github.com/froooze/DEXBot2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
