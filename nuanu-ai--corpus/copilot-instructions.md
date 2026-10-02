## corpus

> Corpus is a multi-tenant, provenance-first company knowledge application.

# Repository guidance

Corpus is a multi-tenant, provenance-first company knowledge application.

- Preserve authentication, tenant isolation, connector scopes, and approval gates.
- Never add real customer data, credentials, private URLs, or local absolute paths.
- Use synthetic fixtures under reserved domains such as `example.com`.
- Run `npm run typecheck`, `npm test`, and `npm run build` for application changes.
- Build `@corpus/company-db` when modifying `packages/company-db`.
- Document schema, environment, or security-model changes in the same pull request.

---
> Source: [nuanu-ai/Corpus](https://github.com/nuanu-ai/Corpus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
