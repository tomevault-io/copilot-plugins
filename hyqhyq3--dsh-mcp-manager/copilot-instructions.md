## dsh-mcp-manager

> DSH plugin: MCP server manager for DeepSeek Harness (web profile). Settings → MCP page for HTTP (OAuth PKCE + RFC 7591 dynamic client registration, or static Bearer token) and local stdio MCP servers, whose tools are registered as `mcp__<name>__<rawName>` native tools.

# AGENTS.md

DSH plugin: MCP server manager for DeepSeek Harness (web profile). Settings → MCP page for HTTP (OAuth PKCE + RFC 7591 dynamic client registration, or static Bearer token) and local stdio MCP servers, whose tools are registered as `mcp__<name>__<rawName>` native tools.

## Layout

The entire plugin is two files — keep it that way unless a refactor is explicitly requested:

- `lib/index.js` — **host half** (Node.js): HTTP API on the DSH GUI webserver under `/mcp-manager/api/*`, OAuth flow (redirect receiver at `/mcp-manager/callback/:id`), both MCP transports, tool registration into `ctx.tools`, state persistence.
- `lib/client.js` — **client half** (browser): a `window.__ModuleLoader__.load(...)` factory using `react.createElement` (no JSX, no bundler). Registers the Settings → MCP tab via the `settings.section` slot. UI strings are Simplified Chinese.
- `cordis.patch.yml` — bundle patch that activates the plugin row (`dsh.bundle.patch` in package.json).
- `README.md` / `README.zh-CN.md` — keep both in sync on behavior changes.

## Architecture rules

- Two halves talk only through the same-origin JSON API `/mcp-manager/api/*`; the client never touches Node APIs and the host never renders UI.
- OAuth redirect origin is derived per-request from the `Host` header — never hardcode a host/port. Client registration is bound to the exact redirect URI and re-registers if the origin changes.
- Token state (`~/.dsh/mcp-manager.json`) contains secrets — never log tokens; treat the file as sensitive.
- Tool name convention `mcp__<server>__<raw>` with `[^A-Za-z0-9_-]` → `_` normalization and a 64-char cap (sha256 suffix on overflow) — must match the built-in `@deepseek-ai/dsh-mcp-client`.
- Tool schemas must pass through `sanitizeValue`/`convParams` (registry accepts only a raw JSON-Schema subset; unsupported vocabulary degrades to unconstrained).
- stdio transport: `child_process.spawn`, newline-delimited JSON-RPC over stdin/stdout. Reconnect must reap the old child first; `ctx.effect` teardown kills all children on unload. `args` are quote-aware tokenized with **no shell expansion**.
- HTTP transport: streamable HTTP (JSON-RPC POST, `Mcp-Session-Id` header, SSE-or-JSON response fallback in `parseRpc`). A 401 triggers one `refresh_token` retry, then reconnect.
- Disable/enable is global per profile: disable unregisters tools + drops the connection but persists config and tokens; enable reconnects without re-auth.

## Commands

No build/test/lint scripts — package.json has none. It's plain ESM with zero dependencies (Node built-ins only; Node `^22.19 || >=24`). Verify changes by installing into a live DSH web profile:

```sh
npx -p @deepseek-ai/dsh dsh plugin --profile web add <path-or-repo>
```

then restart `dsh --profile web` and reload the page. The API liveness probe is `GET /mcp-manager/api/ping`.

## Conventions

- No TypeScript, no bundler, no framework — plain modern JavaScript in both files.
- Host half: `ctx.logger` (`info`/`warn`/`error`) with `mcp-manager:` prefix; never `console.log`.
- Client half: `react` obtained via the factory's `require("react")`; styles in the injected `<style>` string using `--dsw-alias-*` CSS variables with hardcoded fallbacks.
- Bump `version` in package.json on user-visible changes (recent history: 0.1.0 OAuth, 0.2.0 stdio, 0.3.0 enable/disable, 0.4.0 Windows stdio + edit + Codex-style HTTP config).

## Release

Every user-visible change ships as a versioned release: bump the version, sync the docs, commit, then **tag** so consumers can pin `dsh plugin add <repo>#<ref>`. Steps:

1. Bump `version` in `package.json` (semver).
2. Update `README.md` and `README.zh-CN.md` — keep both in sync with the behavior change.
3. Commit on `main`, then create an **annotated** tag on that exact commit and push both:

   ```sh
   git tag -a v0.5.0 -m "dsh-mcp-manager v0.5.0: <one-line summary>"
   git push origin main v0.5.0
   ```

- Tags are **annotated**, not lightweight — match the existing `v0.2.0` style (a `tagger` plus a one-line message).
- The tag must point at the commit that actually carries the release (on `main`).
- Every `vX.Y.Z` gets a tag — never ship a version without one.

## Gotchas

- Changing the client registration/redirect logic requires re-testing a full OAuth round trip — the provider must allow loopback redirects (`http://127.0.0.1:<port>/mcp-manager/callback/<id>`).
- Only `tools` capability is bridged; `resources`/`prompts` are intentionally not.
- Client half is hand-written inside a module factory — no JSX transform available; edit carefully or regenerate deliberately.

---
> Source: [hyqhyq3/dsh-mcp-manager](https://github.com/hyqhyq3/dsh-mcp-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
