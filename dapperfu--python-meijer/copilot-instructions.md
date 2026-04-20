## python-meijer

> - **NEVER** use strings, grep, or plain text tools on mitmproxy logs (*.log files)

# Log Handling Rules

## mitmproxy Log Analysis
- **NEVER** use strings, grep, or plain text tools on mitmproxy logs (*.log files)
- Use `mitmdump` to analyze mitmproxy/mitmweb/mitmweb logs
- These files contain binary data and should not be treated as plain text
- If unsure whether a file is a mitmproxy log, use the proper tooling

## Proper Tools
- `mitmdump` for analyzing mitmproxy logs
- Use the correct tooling that exists for analyzing these files
- Avoid manual text analysis of potentially binary log files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dapperfu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
