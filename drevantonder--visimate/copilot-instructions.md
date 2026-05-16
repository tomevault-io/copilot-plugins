## important-context

> - Project Plan is in notion in "visimate" page

- Project Plan is in notion in "visimate" page
- Readme.md contains basic app idea
- Use `pnpm` to install node packages.
- We are using Nuxt in v4 mode: https://nuxt.com/docs/getting-started/upgrade#testing-nuxt-4
- Use NuxtUI and TailwindCSS for UI: https://ui.nuxt.com/getting-started
- Use Zod for types and type safety
- Avoid premature optimization and remember KISS (Keep it simple stupid)
- Prefer adding to pre-existing files, rather than creating new ones. Create new ones only if it absolutely makes sense.
- Exported functions, variables in `utils` (in shared, app and server) are autoimported. No need to manually import.
- Our database will be sqlite with drizzle and drizzle-kit for simplicity and typescript
- Rely on convention over configuration to keep code shorter and have less complexity
- Prefer typescript inference
- Run `pnpm run db:push` to push migrations to the databases (don't generate please)
- Use conventional commits
- Don't use `any` or `as`. Use zod to parse and throw an error if the result doesn't match

---
> Source: [drevantonder/visimate](https://github.com/drevantonder/visimate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
