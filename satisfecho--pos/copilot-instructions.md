## agent-response-language

> One language per assistant reply; match the user's message language; no mixed-language agent output


# Assistant reply language

- **One language per reply:** Use a single language for the entire message. Do not mix languages in the same answer (no stray words, labels, or sentences in another language).
- **Follow the user's message:** If the user writes in **English**, reply fully in **English**. If they write in **Spanish** or another language, reply fully in that language unless they explicitly ask for a different language.
- **Technical noise:** Log excerpts, code identifiers, URLs, and quoted UI strings may stay as they appear in the product; wrap or explain them in the same language as the rest of the reply.
- **Repository text:** Updates to **`agents/tasks/`**, **`.cursor/rules/`**, **`AGENTS.md`**, **`docs/`** aimed at contributors and agents should stay in **English** unless the task explicitly requires another language. End-user UI copy and **`front/public/i18n/*.json`** follow the translation rules in **`.cursor/rules/angular-ngx-translate.mdc`**.

---
> Source: [satisfecho/pos](https://github.com/satisfecho/pos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
