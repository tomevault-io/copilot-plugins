## ai-battlegrounds

> - NEVER add backwards compatibility code (e.g., default values, optional parameters for new required fields)

# Project Rules

## Code Changes

- NEVER add backwards compatibility code (e.g., default values, optional parameters for new required fields)
- When adding new required fields to types, make them required everywhere
- Breaking changes are acceptable and preferred over compatibility layers
- Remove deprecated code immediately rather than maintaining it

## Testing

- Run tests in non-watch mode

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jesseduffield) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
