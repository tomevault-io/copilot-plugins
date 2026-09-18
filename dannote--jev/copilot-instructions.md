## jev

> - Use `mix ci` for the full validation suite before finishing changes.

# VibeKit quality gate

## Development

```sh
mix deps.get
mix ci
```

## Conventions

- Use `mix ci` for the full validation suite before finishing changes.
- For Phoenix/web apps, keep Phoenix's generated guidance, but treat this VibeKit section as the final quality gate.
- For non-web Elixir projects, VibeKit is the default project baseline.
- Keep changes small, tested, and formatted.

---
> Source: [dannote/jev](https://github.com/dannote/jev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
