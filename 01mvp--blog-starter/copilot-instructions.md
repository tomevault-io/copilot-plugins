## blog-starter

> Opinionated full-stack TypeScript monorepo: TanStack Start + Drizzle + Better Auth, powered by Vite Plus and pnpm workspaces.

# blog-starter

Opinionated full-stack TypeScript monorepo: TanStack Start + Drizzle + Better Auth, powered by Vite Plus and pnpm workspaces.

## How To Use These Docs

- Start with the most specific `.agents/*.md` file for the task.
- If a task spans multiple topics, follow the most specific doc and use linked docs for adjacent context.
- Keep guidance isolated to its owning file. Link to other docs instead of repeating their rules.

## Rule Layering

- This root file is the global router for the repository.
- Nested `AGENTS.md` files live at workspace or high-risk folder boundaries. They add local constraints and point back to the owning `.agents/*.md` docs.
- Do not duplicate detailed rules across nested `AGENTS.md` files. If a rule belongs to a topic doc, update the topic doc and keep the nested file as a pointer.
- If guidance appears to conflict, the more specific folder rule applies to that folder, then update the shared topic doc so future agents do not have to guess.

## Communication And Public Copy

- Keep replies, docs, and UI copy direct. Avoid formulaic AI openings, overly reassuring catchphrases, and stock contrastive filler patterns.
- Reader-facing pages and docs must not expose internal reasoning, author notes, or agent-only implementation instructions.

## Essentials

- Stack: TypeScript + React (TanStack Start) in a pnpm + Vite+ monorepo, with Drizzle ORM, shadcn/ui, and Better Auth.
- Product: Cloudflare-native Personal Blog CMS + AI Init Skill for `github.com/01mvp/blog-starter`.
- Prefer shared `@repo/ui` components; add primitives via shadcn CLI (`pnpm -F @repo/web ui add <component>`).
- Use `lucide-react` for UI icons (use `Icon` suffix, e.g. `import { Loader2Icon } from "lucide-react"`); for brand icons use `@icons-pack/react-simple-icons` (e.g. `SiGithub`).
- Use shared pnpm catalog versions (`pnpm-workspace.yaml`) via `catalog:`.
- Don't build after every little change. If `pnpm lint` passes; assume changes work.

## Always Apply

- [Toolchain](.agents/toolchain.md) for Vite Plus, `vp`/`vpx`, and workspace command syntax.
- [Workflow](.agents/workflow.md) for validation timing, direct-commit policy, focused checks, and migration workflow.
- [Testing](.agents/testing.md) before creating, modifying, or running tests.

## Topic Index

### App And Frameworks

- [TanStack patterns](.agents/tanstack-patterns.md) for route structure, `beforeLoad`, layouts, and route-level preloading.
- [Auth patterns](.agents/auth.md) for Better Auth architecture, auth query behavior, and protected/guest route rules.
- [SEO patterns](.agents/seo.md) for route `head()` usage and sitemap/feed generation.

### Shared Packages And Platform

- [UI guidelines](.agents/ui.md) for app-level UI composition, shadcn usage, icons, images, and extraction decisions.
- [TypeScript conventions](.agents/typescript.md) for schema placement, import boundaries, and `lib/` vs `utils/`.
- [Toolchain](.agents/toolchain.md) for Vite Plus, `vp`/`vpx`, and repo command equivalents.
- [Workflow](.agents/workflow.md) for validation timing and database schema workflows.
- [Testing](.agents/testing.md) for the repo's no-tests-unless-requested policy and test layout.

## Task Entry Points

- UI fix: Start with [UI guidelines](.agents/ui.md). Add [TanStack patterns](.agents/tanstack-patterns.md) when the fix touches routes, loaders, or page composition.
- Bugfix: Start with the owning domain doc from the index above, then use [Workflow](.agents/workflow.md) for narrow validation. Add [Testing](.agents/testing.md) only when the task explicitly calls for tests.
- Test work: Start with [Testing](.agents/testing.md), then load the owning domain doc so the tests match the real feature boundaries.
- End-to-end feature: Start with [TanStack patterns](.agents/tanstack-patterns.md), then load [Auth patterns](.agents/auth.md) or [UI guidelines](.agents/ui.md) as needed.

<!-- intent-skills:start -->

## Skill Loading

Before substantial work:

- Skill check: run `vpx @tanstack/intent@latest list`, or use skills already listed in context.
- Skill guidance: if one local skill clearly matches the task, run `vpx @tanstack/intent@latest load <package>#<skill>` and follow the returned `SKILL.md`.
- Monorepos: when working across packages, run the skill check from the workspace root and prefer the local skill for the package being changed.
- Multiple matches: prefer the most specific local skill for the package or concern you are changing; load additional skills only when the task spans multiple packages or concerns.

<!-- intent-skills:end -->

## Maintenance Rule

If a rule already belongs to another `.agents/*.md` file, link to that file instead of restating it here or copying it into another topic doc.

---
> Source: [01MVP/blog-starter](https://github.com/01MVP/blog-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
