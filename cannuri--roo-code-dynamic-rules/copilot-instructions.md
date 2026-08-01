## roo-code-dynamic-rules

> - When a user requests additional rules, add them to the "Additional Rules" section of the `.clinerules` file in the project's root directory.

## Additional Rules

- When a user requests additional rules, add them to the "Additional Rules" section of the `.clinerules` file in the project's root directory.
- Identify new rules by the prefix `RULE:` in user messages.
- For example, if the user writes `RULE: keep the README.md up to date`, append the rule as `- keep README.md up to date`.
- In contrary if a user writes `NORULE:` you remove the according line from the "Additional Rules" section.
- Persist these rules throughout the project lifecycle.
- Ensure that all added rules are clear, specific, and actionable.
- Format each rule as a separate bullet point.

---
> Source: [cannuri/roo-code-dynamic-rules](https://github.com/cannuri/roo-code-dynamic-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
