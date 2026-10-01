## superqode

> Google models in SuperQode are a **model** route, not a Gemini CLI coding-agent

# Gemini models

Google models in SuperQode are a **model** route, not a Gemini CLI coding-agent
route. SuperQode does not offer Gemini CLI on `:connect`.

## Google's coding agent

Use [Antigravity](antigravity.md) for Google's signed-in agent:

```text
:connect antigravity
```

## Gemini inside SuperQode's harness

Use BYOK when SuperQode should own tools, approvals, and the loop, on a Gemini
model:

```text
:connect byok google <model>
```

Set `GEMINI_API_KEY` or `GOOGLE_API_KEY`. See [Bring Your Own Key](byok.md).

## Related documentation

- [Google Antigravity](antigravity.md)
- [ACP Coding Agents](acp.md)
- [Connection methods](../concepts/modes.md)

---
> Source: [SuperagenticAI/superqode](https://github.com/SuperagenticAI/superqode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
