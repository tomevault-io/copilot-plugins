## hook

> Coverage and mutation testing must both pass at 100%. Always run both after making changes.

# CLAUDE.md

## Testing

Coverage and mutation testing must both pass at 100%. Always run both after making changes.

```bash
# Coverage
herd coverage ./vendor/bin/pest --coverage

# Mutation testing
herd coverage ./vendor/bin/pest --mutate --parallel --everything --covered-only
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kristos80)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kristos80)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
