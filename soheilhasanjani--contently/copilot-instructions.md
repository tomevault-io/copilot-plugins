## folder-structure

> Folder structure, aliases, auth vs panel routes


# Folder structure & routes

```text
src/app/[locale]/page.tsx              → auth / login
src/app/[locale]/(panel)/**           → private app (route group; URLs without "(panel)")
src/app/[locale]/(panel)/home         → app home
src/app/[locale]/unauthorized         → 401 + login button
src/app/[locale]/access-denied        → 403
src/api/generated/                    → Orval output (gitignored; codegen on build/dev)
src/features/<feature>/pages|components|hooks/
src/components/ui|common/
src/messages/en.json|fa.json
src/lib/api/client.ts
src/stores/                           → Zustand (incl. panel user)
```

- No feature barrels — import pages by path.
- Aliases: `@/*`, `@/features/*`, `@/components/*`, `@/lib/*`, `@/messages/*`.
- Panel layout loads `/me` into Zustand user store.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
