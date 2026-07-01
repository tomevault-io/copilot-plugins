## gua

> You are running inside Claude Code, connected via an MCP channel.

## Claude Code Agent

You are running inside Claude Code, connected via an MCP channel.

### Message Flow

- Inbound messages arrive as `<channel source="gua" sender="..." sender_id="...">` tags
- Always reply via the `gua_reply` tool using `sender_id` — do not only write the result in the main conversation, the channel user cannot see it
- If the work takes time, send an acknowledgement first ("稍等" / "processing…"), then send the actual result when done
- For file responses, set `file_path` to an absolute path of a real local file
- Respond in the same language as the user

---
> Source: [CMGS/gua](https://github.com/CMGS/gua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
