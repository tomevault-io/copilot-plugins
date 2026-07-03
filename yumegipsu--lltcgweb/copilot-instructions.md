## lltcgweb-php

> PHP conventions for lltcgweb — src modules, shims, tests


# lltcgweb PHP

## Where code goes

- **New modules:** `src/` under namespace `LLTCG\` (register in `composer.json` autoload).
- **Root `*.php`:** HTTP entry shims and legacy includes only — avoid adding large new functions at root.
- **Set effects:** `*_effects.php` at root until further split; wire through `effects.php`.

## Game engine modules (`src/Game/`)

Procedural `require_once` modules loaded from `effects.php` (global functions, not PSR-4 classes):

| File | Role |
|------|------|
| `ZoneMovement.php` | Deck/hand/WR zone moves |
| `LiveModifiers.php` | `live_modifiers` state, `applyModifierEffect` |
| `LiveScoreBonus.php` | `getLiveScoreBonus`, `getLiveScoreBonusBreakdown`, `collectActiveContinuousEffects` — Performance score + HUD |
| `AbilityResolver.php` | `resolveAbilityEffect` orchestrator + set-module delegation |
| `AbilityResolverSwitch.php` | Core `switch` over non-optional ability `type` values |
| `AbilityResolverSwitchOptional.php` | `tryResolveAbilityEffectSwitchOptional` — optional yes/no, pay, discard, wait branches |
| `PromptResolver.php` | `actionResolvePrompt` — client `resolve_prompt` completion |
| `ActivateAbility.php` | `actionActivateAbility` — client `activate_ability` and WR on-enter activate |
| `LiveStartEffects.php` | `beginLiveStartEffectPhase`, `actionLiveStartChoice`, `finishLiveStartEffects`, optional Live Start queue |
| `PromptLifecycle.php` | Small helpers (`finishPromptEffects`, pick prompts) |

## Paths and config

- Load runtime dirs via `config/paths.php` — no new hardcoded `__DIR__ . '/games/'`.
- CORS: `config/cors.php` allowlist — never restore `Access-Control-Allow-Origin: *`.
- Secrets: env (`TCG_LLR_AUTH_FILE`, `TCG_SYNC_*`) or gitignored local files.

## Database

- Schema changes: add `migrations/NNN_description.sql` + rely on `LLTCG\Db\Migrator`.
- Keep one-time data migrations in `db.php` only when they need PHP logic.

## Before changing `effects.php`

Run `composer test` (especially `tests/Replay/`). Engine changes need regression coverage.

---
> Source: [Yumegipsu/lltcgweb](https://github.com/Yumegipsu/lltcgweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
