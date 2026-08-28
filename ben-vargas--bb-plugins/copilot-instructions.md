## bb-plugins

> - Keep every bb plugin in its own `plugins/<plugin-id>/` directory.

# Repository instructions

- Keep every bb plugin in its own `plugins/<plugin-id>/` directory.
- Add each plugin to `.bb/plugins.json`; do not duplicate plugin manifest data
  in that collection index.
- Run `npm run ci` at the repository root before proposing a release.
- Use plugin-specific release tags: `<plugin-id>/vX.Y.Z`.
- Treat commits, Git pushes, release tags, npm publications, and marketplace
  pull requests as separate publication actions.

---
> Source: [ben-vargas/bb-plugins](https://github.com/ben-vargas/bb-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
