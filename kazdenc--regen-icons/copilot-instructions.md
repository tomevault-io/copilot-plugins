## regen-icons

> Read [AGENTS.md](AGENTS.md) and [docs/spec.md](docs/spec.md) before changing an icon.

# Regen Icons

Read [AGENTS.md](AGENTS.md) and [docs/spec.md](docs/spec.md) before changing an icon.

- Canonical drawings live in `src/<name>.icon.json`; `dist/` and `../svg/` are
  generated and must not be edited.
- Search `src/` for an existing icon before adding a new one.
- Validate with `pnpm check src/<name>.icon.json`.
- Compare geometry beside related icons with `pnpm preview <name> <neighbour> --matrix`.
- Run `pnpm test` before completing an icon change.

Use the smallest set of shapes that communicates the object. Preserve existing names
and metadata when refining an icon, and never import another icon library's paths.

---
> Source: [kazdenc/regen-icons](https://github.com/kazdenc/regen-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
