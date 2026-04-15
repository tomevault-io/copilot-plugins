## omega-gemini-cli

> Headless script usage for omega-gemini-cli (ask-gemini.mjs)


# Omega Gemini headless script

When the user wants Gemini's analysis or execution, use the **headless script** (no MCP):

```bash
node .claude/skills/omega-gemini-cli/scripts/ask-gemini.mjs "PROMPT" [--model MODEL] [--sandbox] [--json]
```

- **PROMPT** — Required. The request to send to Gemini; include @ file refs or context as needed.
- **--model** / **-m** — Optional. e.g. `gemini-2.5-flash` to reduce quota issues.
- **--sandbox** / **-s** — Optional. Sandbox mode for running code.
- **--json** — Optional. Output a JSON object `{"response":"..."}` (consistent envelope on success and error).

Stdin: `echo "prompt" | node .claude/skills/omega-gemini-cli/scripts/ask-gemini.mjs`

Full reference: `.claude/skills/omega-gemini-cli/references/headless.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oimiragieo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
