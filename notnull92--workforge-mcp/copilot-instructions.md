## workforge-mcp

> This repository contains the distributable Windows package and a source-based

# WorkForge Guidance

This repository contains the distributable Windows package and a source-based
macOS preview for one general-purpose ChatGPT workstation MCP profile.

- Preserve the default no-startup behavior. A Windows restart or macOS login
  must leave the tunnel stopped until the user starts it manually.
- Never commit runtime credentials, tunnel profiles, generated registries,
  logs, absolute user paths, or release archives.
- Keep the default distribution independent from personal projects and optional
  machine-specific integrations.
- Start read-only when a target is unclear. Mutations require a fresh bootstrap
  context revision and exact-path verification.
- Treat public release, license selection, code signing, and third-party license
  review as explicit gates.

---
> Source: [NotNull92/workforge-mcp](https://github.com/NotNull92/workforge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
