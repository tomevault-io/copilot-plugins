## adobe-enterprise-cms-demo

> Project working rules for Adobe Enterprise CMS


- Root-cause bias
  - Prefer fixing underlying causes over symptom-level workarounds
- Summarization quality
  - Provide thorough, complete summaries; do not omit important details
- Architecture alignment
  - Follow existing patterns and file boundaries (Keystone config, lists, access rules, admin customizations)
- Access control
  - Favor backend enforcement via list `access` (read/create/update/delete & filter rules)
- Operational discipline
  - Use absolute paths for tool calls
  - Parallelize read-only discovery (search/reads) to speed up context gathering
- Coding & communication
  - Keep code edits concise; explanations detailed when summarizing
  - Use clear navigation of files/functions with backticks

@schema.ts
@access.ts
@keystone.ts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eliasisraelancona-dotcom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
