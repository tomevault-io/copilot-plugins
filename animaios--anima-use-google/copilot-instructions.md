## anima-use-google

> This repo provides a local MCP tool, `search_google_ai`, that searches Google AI Mode through the user's logged-in browser session. Future Codex agents should treat it as the preferred research path for current external information:

# Agent Notes

## Use Google AI Mode For Fresh Research

This repo provides a local MCP tool, `search_google_ai`, that searches Google AI Mode through the user's logged-in browser session. Future Codex agents should treat it as the preferred research path for current external information:

- Browser extension policy and Manifest V3 behavior
- Current API/package documentation
- Release/version state
- Web platform behavior
- Any user query where fresh web context could materially improve the answer

Use it extensively. Google AI Mode searches through this tool are effectively unlimited for normal development/research use. Do not replace local code inspection, tests, or compiler output with search results; use search to fill external/contextual gaps, then verify against the repository.

## Codex MCP Workaround

Codex in this environment does not directly expose MCP tools, so call the MCP server with the official SDK over stdio. Run from the repo root after `npm run build`.

```bash
node --input-type=module <<'EOF'
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const query = process.argv.slice(2).join(' ') || 'Chrome Manifest V3 native messaging restrictions answer in English';
const transport = new StdioClientTransport({
  command: 'node',
  args: ['dist/index.js'],
  cwd: process.cwd(),
  stderr: 'pipe',
});
transport.stderr?.on('data', (chunk) => process.stderr.write(`[mcp stderr] ${chunk}`));

const client = new Client({ name: 'codex-google-ai-research', version: '0.0.0' });
try {
  await client.connect(transport);
  const result = await client.callTool({
    name: 'search_google_ai',
    arguments: { query, timeout_ms: 60000 },
  });
  console.log(result.content?.[0]?.text ?? JSON.stringify(result, null, 2));
} finally {
  await client.close().catch(() => {});
}
EOF
```

For multiple searches in one process, keep the same client open and call `client.callTool(...)` repeatedly. Prefer precise, research-style queries and include `answer in English` when the language matters, although the MCP server also appends an English-answer instruction.

## Local Verification Still Wins

Before claiming work is complete, run the relevant local verification:

```bash
npm test
npm run typecheck
npm run build
npm pack --dry-run
```

For extension changes, also inspect or test the browser-specific manifest and native-host installer path. Chrome/Chromium/Brave native messaging requires an extension id in `allowed_origins`, so use `native-host/install-chrome-host.cjs --browser <chrome|chromium|brave> --extension-id <id>` after loading the unpacked Chromium extension and obtaining its id.

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

For Firefox, use:

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

- Linux: `~/.config/BraveSoftware/Brave-Browser/NativeMessagingHosts/com.google.ai.search.json`
- macOS: `~/Library/Application Support/BraveSoftware/Brave-Browser/NativeMessagingHosts/com.google.ai.search.json`
- Windows: `HKCU\Software\BraveSoftware\Brave-Browser\NativeMessagingHosts\com.google.ai.search`

The installer writes those locations for `--browser brave`; then reload the Brave extension from `brave://extensions`.

After reloading, confirm the active host. A Chromium-based host should show the extension origin as an argument, for example:

```text
node /path/to/native-host/host.js chrome-extension://<extension-id>/
```

Firefox hosts usually do not include that Chrome extension-origin argument.

---
> Source: [animaios/anima-use-google](https://github.com/animaios/anima-use-google) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
