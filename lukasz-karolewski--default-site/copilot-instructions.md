## default-site

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
 
# Next.js: ALWAYS read docs before coding
 
Before any Next.js work, find and read the relevant doc in `node_modules/next/dist/docs/`. Your training data is outdated — the docs are the source of truth.
 
<!-- END:nextjs-agent-rules -->

# Standards: 
- all components should be thin, view only, all business logic should be in ./src/lib/
- Always use existing components form ./src/components/ when possible, if not available, use shadcn, and if there isn't one only then create a new one in ./src/components/ and use it.
- All DB types must be derived from Drizzle schema using `$inferSelect` / `$inferInsert` and exported from `./src/lib/data/schema.ts`. Never duplicate DB interfaces in component files.
- All DB migrations MUST be created using `drizzle-kit generate`. Never write migration SQL files by hand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukasz-karolewski)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lukasz-karolewski)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
