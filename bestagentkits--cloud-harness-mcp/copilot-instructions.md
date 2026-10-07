## cloud-harness-mcp

> Configuring Google Gemini CLI for remote Streamable HTTP MCP.


# Google Gemini CLI

## Configuration

Set your authorization token in your shell environment:

```bash
export CLOUD_HARNESS_MCP_TOKEN="<YOUR_DASHBOARD_API_KEY>"
```

Add the server to Gemini CLI settings or pass it via command-line flags:

```bash
gemini mcp add cloud-harness \
  --url https://api.harness.zuey.me/mcp \
  --header "Authorization: Bearer $CLOUD_HARNESS_MCP_TOKEN"
```

---
> Source: [bestagentkits/cloud-harness-mcp](https://github.com/bestagentkits/cloud-harness-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
