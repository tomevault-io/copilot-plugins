## frappe-agent

> Frappe and ERPNext repository guidance


- Inspect bench, site, and app context before suggesting mutations.
- Prefer the narrowest valid Frappe customization layer before framework code changes.
- Prefer Frappe-native ORM and Query Builder patterns before raw SQL.
- When reviewing raw SQL, check `tab{DocType}` naming, permissions, and transaction boundaries.
- Distinguish desk-native, `www`, Vue/`frappe-ui`, React, and external SPA frontend paths.
- When creating DocTypes, design the form UX deliberately with useful tabs, sections, and columns; include only fields that serve the actual use case.
- Treat custom-remoted or forked apps as custom-derived apps and avoid invasive upstream patching when a custom app can own the change.

---
> Source: [Dkm0315/frappe-agent](https://github.com/Dkm0315/frappe-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
