## macro-tracker

> For detailed documentation about this repository, see [REPO.md](./REPO.md).

# AGENTS.md

For detailed documentation about this repository, see [REPO.md](./REPO.md).

## Development Guidelines

### Code Style
- Run `bun run lint` before committing - this uses Biome for linting
- Run `bun run format` to format code with Biome
- Do NOT add comments to code unless explicitly requested
- Follow existing code patterns and conventions

### Commands
- `bun run dev` - Start development server with Turbopack
- `bun run build` - Build for production
- `bun run lint` - Run Biome linter (check code)
- `bun run format` - Format code with Biome
- `bun run deploy` - Deploy to Cloudflare Workers

### Tech Stack Notes
- Next.js 15 with App Router
- React 19
- TypeScript throughout
- Convex for backend (queries in `convex/` directory)
- Clerk for authentication (use `useConvexAuth()` hook, not Clerk's `useAuth()`)
- shadcn/ui components in `components/shadcn/`
- Radix UI imports use pattern: `import { ComponentName as Primitive } from "radix-ui"`
- Zod for validation

### Project Structure
- `app/` - Next.js pages (App Router)
- `components/` - Reusable React components
- `convex/` - Backend functions and schema
- `actions/` - Server Actions
- `utils/` - Utility functions

### Important
- Never commit secrets or API keys
- Always test changes before committing
- Use `Authenticated`, `Unauthenticated`, `AuthLoading` from Convex, not Clerk
- All Convex functions must check `ctx.auth.getUserIdentity()`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JoshuaKirby88)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JoshuaKirby88)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
