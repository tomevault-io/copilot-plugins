## executioncontrolprotocol

> Prefer breaking clarity over legacy aliases — no default backward compatibility for now


# Compatibility policy (current)

**Default: do not preserve backward compatibility** unless the user explicitly asks to keep legacy behavior.

- **Config and manifests:** use current keys and shapes only (e.g. `plugins`, `loggers`). Do not add loaders, CLI commands, or type aliases whose only purpose is to accept old names or deprecated fields.
- **CLI:** one command path per concern — avoid duplicate topics (e.g. `extensions` vs `plugins`) that do the same thing.
- **Types:** avoid `@deprecated` re-exports of renamed symbols unless there is a strong external API commitment; prefer a single canonical name.
- **When in doubt:** choose the simpler, stricter behavior and document the migration in prose if needed, rather than silently accepting legacy input.

If the user later requests compatibility for a specific migration, implement it narrowly for that case.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/executioncontrolprotocol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
