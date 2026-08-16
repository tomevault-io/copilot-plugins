## seektty

> This repository ships one out-of-tree DeepSeek Harness Bundle. Harness remains the only owner of Agent, Session, model, settings, permissions, Profile, plugin, and persistence state.

# Repository instructions

This repository ships one out-of-tree DeepSeek Harness Bundle. Harness remains the only owner of Agent, Session, model, settings, permissions, Profile, plugin, and persistence state.

- The consumer-facing package must not contain `workspace:` dependencies.
- Test install, boot, remove, and reinstall against an unmodified official dsh release under an isolated `DSH_HOME`.
- Keep the GitHub repository private until the user explicitly authorizes publication.
- Never commit credentials, Session data, `.env`, AppleDouble files, local Profile directories, or generated package-manager caches.
- Preserve the `dsh.bundle.patch` manifest and native `dsh plugin` reconciliation semantics.
- Compatibility adapters belong inside this package and must name the exact tested dsh range.

---
> Source: [Hilbert-beinghappy/seektty](https://github.com/Hilbert-beinghappy/seektty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
