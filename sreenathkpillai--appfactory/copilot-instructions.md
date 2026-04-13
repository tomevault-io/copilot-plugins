## appfactory

> App Factory is a **Claude Code orchestration framework** — a feature library + assembler pipeline that Claude Code operates via markdown skills. The intelligence lives in skills, not coded AI modules.

# App Factory — Claude Code Guide

App Factory is a **Claude Code orchestration framework** — a feature library + assembler pipeline that Claude Code operates via markdown skills. The intelligence lives in skills, not coded AI modules.

## Git Rules

- Never mention or attribute Claude, AI, or any AI assistant in commit messages, PR descriptions, branch names, or any git-related content
- No `Co-Authored-By` lines referencing Claude or AI
- Commit messages should read as if a human developer wrote them
- Use conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`

---

## Architecture

### Skills (the brain) — `skills/`
Markdown instructions that Claude Code follows using native capabilities (file ops, web search, Playwright MCP, frontend-design skill). Skills orchestrate the entire pipeline — from meeting transcription to deployed prototype.

### Feature Library (the catalog) — `features/`
Pre-built components organized as features with functional variants and visual skins. Structure spec: `docs/component-standard.md`.

### Assembler (the muscle) — `src/compose/`
TypeScript pipeline: copy template → generate CSS → copy variants → merge Prisma → merge middleware → generate layouts → install deps. ~15s.

### Template (the skeleton) — `template/`
Bundled Next.js 16 + React 19 + Tailwind CSS 4 + NextAuth v5 + Prisma/Neon.

---

## Working on the Factory

### Compose from YAML config

```bash
npx tsx bin/appfactory.ts compose -c <config.yaml> -o <output-dir>
```

### Verify a generated project

```bash
cd <output-dir>/<app-slug>
pnpm run build
```

### Preview components

```bash
pnpm --filter harness dev
```

---

## Working Inside a Generated Project

### Project Structure

```
src/
├── app/
│   ├── layout.tsx              # Root layout (fonts, ThemeProvider)
│   ├── globals.css             # Design tokens via @theme inline
│   ├── (public)/               # No auth required
│   │   ├── layout.tsx          # Nav bar + public shell
│   │   ├── page.tsx            # Landing page
│   │   └── login/page.tsx      # Login form
│   ├── (portal)/               # Auth required
│   │   ├── layout.tsx          # Auth check + sidebar
│   │   ├── dashboard/          # Server + client component pair
│   │   └── <model-slug>/      # CRUD pages
│   └── api/                    # REST endpoints
├── components/ui/              # Button, Card, Input, Badge (CVA)
├── lib/                        # prisma, auth, utils, motion
└── types/                      # next-auth.d.ts
prisma/
├── schema.prisma
└── seed.ts
```

### Key Patterns

**Server/Client Split:** Every `(portal)/` page has `page.tsx` (server, Prisma queries) + `*-client.tsx` (client, UI rendering).

**Auth:** NextAuth v5, Credentials provider, roles: ADMIN/MEMBER/VIEWER. Session includes `user.id` and `user.role`.

**Styling:** Tailwind CSS 4 with `@theme inline`. CSS custom properties for colors (`--color-primary-500` etc). Dark mode via `next-themes`. Glassmorphism utilities.

**Components:** CVA-based Button (default/secondary/outline/ghost/danger), Badge, Card, Input.

### Quality Gates

After any change:
1. `pnpm run build` must pass
2. Dark mode: no hardcoded hex colors
3. If Prisma modified: `prisma validate` → `prisma generate` → rebuild

### Environment

```bash
DATABASE_URL="postgresql://..."      # Neon (required)
NEXTAUTH_SECRET="..."                # openssl rand -base64 32
NEXTAUTH_URL="http://localhost:3000"
```

Default login after seeding: `admin@<app-slug>.com` / `changeme123`

---

## .context/ Directory

| Directory | Purpose |
|-----------|---------|
| `.context/meetings/` | Plaude drops transcripts here |
| `.context/clients/` | Brand briefs per client |
| `.context/compose/` | Generated compose configs |
| `.context/reports/` | R&D cycle reports |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sreenathkpillai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sreenathkpillai)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
