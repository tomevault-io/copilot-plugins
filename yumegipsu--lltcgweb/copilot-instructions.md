## lltcgweb-cards-effects

> Card abilities and effects — validation and regression requirements


# lltcgweb cards & effects

## Ability changes

1. Update `cards.json` (and `import_from_db.py` if importing from DB).
2. Implement handler in `effects.php` or set-specific `*_effects.php`.
3. Add client prompts in `index.html` or `client/js/` when interactive.
4. Run `php scripts/validate_cards.php` — every `abilities[].type` should map to a known handler.

## Effect registry

Known types are discovered from `src/Game/AbilityResolverSwitch*.php` via `LLTCG\Game\EffectRegistry`. New core `type` strings add a `case` in `resolveAbilityEffectSwitch` or `tryResolveAbilityEffectSwitchOptional` (optional `optional_*` types), or a set module if set-specific.

## Live modifiers

Modifier `then` types (`live_score_bonus`, `blade_bonus`, `grant_bonus_hearts`, etc.) are applied via `applyModifierEffect` in `src/Game/LiveModifiers.php` — not card `abilities[].type` values and not covered by `EffectRegistry`.

## Regression

Engine / prompt lifecycle changes: add or update `tests/fixtures/replays/*.json` and run `composer test`.

## Chiichan parity

Discord card import (`LoveLiveTCG.py`, `tcg_cards_en_bridge.py`) is separate from web `cards.json` deploy — see Chiichan `.cursorrules`.

---
> Source: [Yumegipsu/lltcgweb](https://github.com/Yumegipsu/lltcgweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
