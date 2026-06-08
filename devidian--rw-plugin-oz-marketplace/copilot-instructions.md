## rw-plugin-oz-marketplace

> This repository owns the OZ Marketplace Rising World plugin.

# AGENTS.md

## Repository Purpose
This repository owns the OZ Marketplace Rising World plugin.

The plugin provides player-to-player item trading through market zones, local/global listings, Wallet-backed purchases, marketplace fees, and seller sale history.

## Ownership
Owns:
- market zones and fee overrides
- marketplace listing persistence
- listing creation from player inventory
- Wallet-backed buyer payments and seller payouts
- seller sale history
- marketplace command workflow

Does not own:
- wallet balances, currencies, or transaction history
- shared UI, settings, logging, SQLite, or i18n infrastructure
- system shop or plugin-registered purchase actions

## Dependencies
- `rw-plugin-oz-tools` is a hard runtime dependency.
- `rw-plugin-oz-wallet` is required for functional listings and purchases.

If Wallet is missing, the plugin may load but trading must stay disabled and admins must receive a warning.

## Mandatory Workflow Rules
- Keep Marketplace business logic in this repository.
- Use `rw-plugin-oz-tools` for shared infrastructure instead of duplicating helpers.
- Use Wallet through the public API/reflection bridge; do not copy wallet state logic.
- Keep prices and fees as whole-number values.
- Marketplace fees leave the economy.
- Keep listing expiration out of scope for v1 unless explicitly requested.
- Local listings are limited to their market zone.
- Global listings are only usable in zones that allow global trade.
- Follow `.codex/agents.toml` and `docs/policies/repository-policy.md`.
- Keep `README.md`, `HISTORY.md`, and `PLANS.md` aligned with behavior changes.

## Validation
- Run `scripts/verify-plugin-api.sh --summary` for API-impacting work.
- Run `mvn -B -DskipTests package` for build-impacting changes.
- Run `mvn -B test` when tests exist.

---
> Source: [Devidian/rw-plugin-oz-marketplace](https://github.com/Devidian/rw-plugin-oz-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
