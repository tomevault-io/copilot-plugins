## racore-browser

> Racore Browser has three runtime layers:

# Racore Browser project guide

Racore Browser has three runtime layers:

1. React/Vinext for the hosted application and a static Vite entry for desktop.
2. Rust/Tauri v2 for native commands, windows, security, and sidecar lifecycle.
3. The Go `racored` daemon for providers, vault, authority, IPFS, and mesh behavior.

## Important paths

| Area | Path |
| --- | --- |
| Shared React UI | `app/` |
| Typed desktop adapter | `app/lib/desktop.ts` |
| Daemon client | `app/lib/racore-client.ts` |
| Desktop React entry | `desktop-ui/` |
| Rust application | `src-tauri/` |
| Go daemon and CLI | `god/` |
| Architecture docs | `.llms/` and `docs/` |

## Invariants

- Preserve daemon error bodies as `{ "detail": "message" }`.
- Keep frontend and Go request/response types aligned.
- Desktop daemon traffic always targets `http://127.0.0.1:47831` through the Rust allowlist.
- Remote browser windows must not inherit the trusted main-window capability.
- Do not add frontend filesystem, OS, or unrestricted shell permissions.
- Run Go tests with `-race -count=1` where the platform toolchain supports race detection.
- Keep `src-tauri/Cargo.lock` and `package-lock.json` committed.

## Commands

```powershell
npm run build
npm run desktop:ui
npm run desktop:dev
npm run desktop:package
npm test
npm run lint
cargo test --manifest-path src-tauri/Cargo.toml
```

---
> Source: [racore-xyz/racore-browser](https://github.com/racore-xyz/racore-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
