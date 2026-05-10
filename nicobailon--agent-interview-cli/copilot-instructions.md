## agent-interview-cli

> Standalone CLI + library for presenting interactive web forms. Opens a browser form, collects structured responses, returns JSON on stdout. Zero runtime dependencies.

# agent-interview-cli

Standalone CLI + library for presenting interactive web forms. Opens a browser form, collects structured responses, returns JSON on stdout. Zero runtime dependencies.

Use `interview` instead of back-and-forth chat when: choosing between approaches, gathering requirements, exploring tradeoffs, or multi-dimensional decisions.

If `interview` is not in PATH, use the full path to `bin/interview.mjs` in this repo.

```bash
echo '{"questions":[...]}' | ./bin/interview.mjs --stdin --pretty 2>/dev/null
```

For full question format, types, recommendations, media blocks, and info panels — load the `agent-interview-cli` skill.

---
> Source: [nicobailon/agent-interview-cli](https://github.com/nicobailon/agent-interview-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
