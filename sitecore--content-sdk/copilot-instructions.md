## cli

> CLI behavior, prompts, init flow, next steps


# CLI

- Drive init via `Initializer.init(args)`
- Clear prompts and defaults
- Install dependencies after scaffolding
- Print next steps

Non-goals:
- No deployments or CI flows
- No global user state changes

Backwards compatibility:
- Avoid breaking arg names
- Additive changes with defaults

Referenced:
@src/initializers/nextjs/index.ts
@src/initializers/nextjs/prompts.ts
@src/initialize.ts
@src/bin.ts

---
> Source: [Sitecore/content-sdk](https://github.com/Sitecore/content-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
