## pi-ext

> 1. Create `extensions/<name>/<name>.ts` (or `index.ts` for multi-file extensions)

# Adding Extensions

1. Create `extensions/<name>/<name>.ts` (or `index.ts` for multi-file extensions)
2. Add the entry point to `pi.extensions` in `package.json`
3. If the extension has npm dependencies, add a `package.json` in its directory and run `npm install`

Skills go in `skills/<name>/SKILL.md`, themes in `themes/*.json`, prompts in `prompts/*.md`.

---
> Source: [tomsej/pi-ext](https://github.com/tomsej/pi-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
