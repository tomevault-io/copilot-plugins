## tsgo-after-typescript

> Run tsgo and eslint after TypeScript file changes


# TypeScript changes: run tsgo and eslint

After making changes to TypeScript files (`.ts`), always run these checks from the project root:

1. **Type checker**
   ```bash
   npx tsgo
   ```

2. **Linter**
   ```bash
   npx eslint ./
   ```

- Run both from the project root.
- If either reports errors, fix them before considering the task done.
- Do not skip these steps when you have edited or created any `.ts` file.

---
> Source: [bifravst/video.thingy.rocks](https://github.com/bifravst/video.thingy.rocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
