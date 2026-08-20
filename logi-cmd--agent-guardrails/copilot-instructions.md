## agent-guardrails-enforce

> <!-- agent-guardrails-enforce:start -->

<!-- agent-guardrails-enforce:start -->
# agent-guardrails: ALWAYS run before finishing

Before telling the user a task is done, you MUST:

1. Run: `agent-guardrails check --base-ref HEAD~1`
2. If issues found → fix them first, do NOT say "done"
3. If clean → include the check result in your summary
4. If command not found → tell user to run `npx agent-guardrails setup`

This is not optional. Skipping this check means the task is incomplete.
<!-- agent-guardrails-enforce:end -->

---
> Source: [logi-cmd/agent-guardrails](https://github.com/logi-cmd/agent-guardrails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
