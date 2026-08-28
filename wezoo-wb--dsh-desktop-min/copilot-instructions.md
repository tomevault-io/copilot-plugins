## dsh-desktop-min

> `dsh-desktop-min` is a product-neutral desktop extension host for DeepSeek Harness. Keep Agent, session, workspace, product UI, branding, and business-system behavior outside the core.

# Repository guidance

`dsh-desktop-min` is a product-neutral desktop extension host for DeepSeek Harness. Keep Agent, session, workspace, product UI, branding, and business-system behavior outside the core.

The core may own only Electron bootstrap, the secure main window, Host connection lifecycle, extension discovery, versioned transport, capability routing, command routing, diagnostics, and shutdown. Common desktop features belong in extensions, including embedded pages, tray, updates, custom chrome, notifications, and remote connectors.

Treat native extensions as fully trusted local code. Regular DSH and Client plugins receive only the narrow bridge. Validate every renderer IPC envelope, child-process message, URL, filesystem path, extension manifest, and JSON payload. Keep `contextIsolation`, `sandbox`, `nodeIntegration: false`, and `webSecurity` enabled.

Use the repository-pinned pnpm release through Corepack. Before publishing, run `corepack pnpm run check`; package Windows on Windows and macOS x64/arm64 on macOS. Do not add another package manager or lockfile.

---
> Source: [wezoo-wb/dsh-desktop-min](https://github.com/wezoo-wb/dsh-desktop-min) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
