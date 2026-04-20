## eastmoney

> 1. **Always read this file** before starting a task to understand which skills or rules to load from `.agents/`.

1. **Always read this file** before starting a task to understand which skills or rules to load from `.agents/`.
2. **Skills**:
   - Load a skill only if its trigger condition matches the task. For example, code-review tasks must load `skills/code-review/SKILL.md`.
   - Once loaded, follow the process and output format defined in the skill file so that the final response remains consistent.
3. **Rules**:
   - Rules are long-lived constraints (API guidelines, React component practices, etc.). Whenever a task touches those domains, read the corresponding file under `.agents/rules/`.
   - Treat these rules as required context: preload them before drafting any response and ensure all recommendations comply with them.
4. **Response contract**:
   - Explicitly state which skills and rules are in effect.
   - Provide findings, recommendations, or code while enforcing all loaded constraints. If any conflicts arise, ask for clarification before diverging.

---
> Source: [Austin-Patrician/eastmoney](https://github.com/Austin-Patrician/eastmoney) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
