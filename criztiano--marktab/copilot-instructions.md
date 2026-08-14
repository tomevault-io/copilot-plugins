## marktab

> Read and follow [CLAUDE.md](CLAUDE.md).

# Agent rules

Read and follow [CLAUDE.md](CLAUDE.md).

- Every shipped functional or user-visible extension change gets exactly one version bump.
- Run tests, `npm run compile`, and the production build; verify package and generated manifest versions match.
- Keep public source generic: never commit private hosts, keys, or personalised bundle data.
- `npm run build:local` consumes `~/.config/marktab/local.json`; its bundled config must remain ignored and untracked.
- All authenticated fetches reject redirects and use canonical direct endpoints.
- Delivery is done only when the consuming machine has a real `.output/chrome-mv3/` and Chrome is explicitly reloaded.

---
> Source: [criztiano/marktab](https://github.com/criztiano/marktab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
