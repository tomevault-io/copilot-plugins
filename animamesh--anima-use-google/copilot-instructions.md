## anima-use-google

> This repo provides a local MCP tool, `ask_google`, that searches Google AI Mode through the user's logged-in browser session. Use it as the preferred research path for current external information:

# Agent Notes

## Use Google AI Mode For Fresh Research

This repo provides a local MCP tool, `ask_google`, that searches Google AI Mode through the user's logged-in browser session. Use it as the preferred research path for current external information:

- Browser extension policy and Manifest V3 behavior
- Current API/package documentation
- Release/version state
- Web platform behavior
- Any user query where fresh web context could materially improve the answer

Searches are effectively unlimited for normal development/research use. Do not replace local code inspection, tests, or compiler output with search results; use search to fill external/contextual gaps, then verify against the repository.

## OMP Harness Capabilities

This agent runs in the **Oh My Pi (OMP)** coding harness (v16.4.8, Linux x64, Rust engine + Bun/TS frontend). The harness provides:

**Core tools:** `read`, `grep`, `glob`, `lsp`, `web_search`, `irc`, `yield`, `generate_image`, `ask`, `inspect_image`, `checkpoint`, `rewind`, `todo`, `job`, `task`, `eval`, `debug`, `ast_grep`, `ast_edit`, `github`, `ssh`, `bash`, `edit`, `write`, `resolve`, `search_tool_bm25`, `tts`

**jcodemunch MCP (18 code-intelligence tools):** AST search, dead code detection, dependency analysis, refactoring, symbol search, blast radius, imports, call graph, code coverage, etc.

**Skills system:** 11 loaded skills from `~/.omp/agent/skills/`: agents-sdk, cloudflare, cloudflare-email-service, cloudflare-one, cloudflare-one-migrations, durable-objects, sandbox-sdk, turnstile-spin, web-perf, workers-best-practices, wrangler. Use `skill://<name>` to load any skill before tasks in that domain.

**Memory backend:** Mnemopi (per-project-tagged, polyphonic recall, proactive linking, enhanced recall)

**Model:** `vllm/auto` (local vLLM inference on AMD Ryzen 7 7800X3D with NVIDIA GTX 1060 3GB)

## Local Verification

Before claiming work is complete, run:

```bash
npm test
npm run typecheck
npm run build
```

For extension changes, verify the browser-native-messaging integration after reinstalling the host for the target browser.

## Switching Browser Sidecars For E2E Tests

The MCP server talks to whichever native host is listening on `127.0.0.1:51784`. Do not claim a Chrome/Chromium/Brave test passed if a Firefox-started host is still running.

Before switching browsers:

```bash
ps -ef | rg '[n]ative-host/host\.js|[b]rave|[c]hrome|[c]hromium|[f]irefox'
ss -ltnp 'sport = :51784'
```

If the wrong host is active, disable or unload that browser's sidecar extension and stop the stale host process:

```bash
kill <native-host-pid>
```

For Firefox:

```bash
node native-host/install-host.cjs
```

Then load `extension/manifest.json` from `about:debugging#/runtime/this-firefox` and restart or reload Firefox.

For Chrome, Chromium, or Brave, load `extension-chromium/` as an unpacked extension, copy the generated extension id, and run:

```bash
node native-host/install-chrome-host.cjs --browser chrome --extension-id <id>
node native-host/install-chrome-host.cjs --browser chromium --extension-id <id>
node native-host/install-chrome-host.cjs --browser brave --extension-id <id>
```

Brave uses the Chromium extension. Its native-messaging registration locations are:

- Linux: `~/.config/BraveSoftware/Brave-Browser/NativeMessagingHosts/com.ask.google.json`
- macOS: `~/Library/Application Support/BraveSoftware/Brave-Browser/NativeMessagingHosts/com.ask.google.json`
- Windows: `HKCU\Software\BraveSoftware\Brave-Browser\NativeMessagingHosts\com.ask.google`

After reloading, confirm the active host. A Chromium-based host shows the extension origin as an argument:

```text
node /path/to/native-host/host.js chrome-extension://<extension-id>/
```

Firefox hosts usually do not include that Chrome extension-origin argument.

---
> Source: [animamesh/anima-use-google](https://github.com/animamesh/anima-use-google) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
