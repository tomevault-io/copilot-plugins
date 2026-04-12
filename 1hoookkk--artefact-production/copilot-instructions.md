## artefact-production

> This rule is triggered when the user types `@README` and activates the README agent persona.


# README Agent Rule

This rule is triggered when the user types `@README` and activates the README agent persona.

## Agent Activation

CRITICAL: Read the full YAML, start activation to alter your state of being, follow startup section instructions, stay in this being until told to exit this mode:

```yaml
Local agent contracts for SpectralCanvas Pro. These take precedence over `docs/agents/`.

## Structure
- `*.contract.yaml` - Core agent definitions  
- `profiles/` - Specialized variants
- `BestOfN.yml` - Scoring framework (mirrors docs/agents/)

## Migration Note
Migrated from `docs/agents/` on 2025-08-20 for better Claude integration.
Original contracts preserved with `source_path` metadata.

## Loading Priority
Commands load contracts in this order:
1. `.claude/agents/` (highest priority)
2. `docs/agents/` (fallback)
3. Warning issued if duplicates found

## Rollback Instructions
Delete this entire `.claude/agents/` directory to revert to `docs/agents/` loading.
No code changes needed - commands auto-fallback.
```

## File Reference

The complete agent definition is available in [.claude/agents/README.md](mdc:.claude/agents/README.md).

## Usage

When the user types `@README`, activate this README persona and follow all instructions defined in the YAML configuration above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1hoookkk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/1hoookkk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
