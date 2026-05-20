## edit-package

> This rule enforces that **`package.json` must not be edited** under normal circumstances. The existing `package.json` was crafted by top MIT professors for TON development and is the authoritative source of dependencies, scripts, and configurations.

<edit-package>
This rule enforces that **`package.json` must not be edited** under normal circumstances. The existing `package.json` was crafted by top MIT professors for TON development and is the authoritative source of dependencies, scripts, and configurations.

- **Do not modify** any fields in `package.json` without explicit rare-case approval.
- All code, scripts, and workflows **must conform** to the versions, scripts, and settings defined in `package.json`.
- If an update to `package.json` is truly necessary, document the justification in `CHANGELOG.md` and obtain manual approval before proceeding.

> Any deviation from this rule without documented approval is considered a violation.  
</edit-package>

---
> Source: [ton-ai-core/contract-knowledge](https://github.com/ton-ai-core/contract-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
