## qiekn-github-io

> Personal resume / portfolio site, deployed at `https://qiekn.github.io`.

# qiekn.github.io

Personal resume / portfolio site, deployed at `https://qiekn.github.io`.

## Stack

- **Vite 8** + **React 19** + **TypeScript 5.9** (strict, `noUnusedLocals`, `noUnusedParameters`)
- **Tailwind CSS v4** via `@tailwindcss/vite` — configured inline in `src/index.css` with `@theme inline`. **No `tailwind.config.js`**.
- **shadcn/ui** — style `base-nova`, base color `neutral`, CSS variables enabled. Components live in `src/components/ui/`.
- **@base-ui/react** primitives, **lucide-react** icons, **Geist Variable** font.
- SCSS is available (`sass` installed) — `src/App.scss` uses it for the current scaffold.
- **pnpm** is the package manager (lockfile is `pnpm-lock.yaml`; CI uses pnpm 10).

## Commands

```bash
pnpm dev        # vite dev server
pnpm build      # tsc -b && vite build → dist/  (TS errors fail the build)
pnpm lint       # eslint .
pnpm preview    # preview dist/
```

## Layout

```
src/
  main.tsx          # entry
  App.tsx           # root component
  App.scss          # component-scoped SCSS (current scaffold)
  index.css         # Tailwind + shadcn theme + CSS vars (light + .dark)
  components/ui/    # shadcn components (added via `pnpm dlx shadcn@latest add <name>`)
  lib/utils.ts      # cn() = clsx + tailwind-merge
  assets/           # images imported from TS/TSX
public/             # served as-is at site root (favicon.svg, icons.svg)
design/design.pen   # Pencil design file — use the pencil MCP tools, not Read/Grep
```

- Path alias: `@/*` → `src/*` (set in both `vite.config.ts` and `tsconfig.app.json`).
- Dark mode: toggle the `.dark` class on a root element. Custom variant registered in `index.css` as `@custom-variant dark (&:is(.dark *))`.

## Styling conventions

- Prefer **Tailwind utilities + shadcn tokens** (`bg-background`, `text-foreground`, `border-border`, etc.) over raw hex values so light/dark stay in sync.
- Use `cn()` from `@/lib/utils` when composing conditional class strings.
- The current scaffold (`App.scss` + `.page-shell` / `.intro-card`) is placeholder — feel free to replace it with Tailwind-based sections when building the portfolio.

## Deployment

- GitHub Actions workflow: `.github/workflows/deploy-pages.yml`. Push to `main` → builds with pnpm/Node 22 → publishes `dist/` to GitHub Pages.
- This is a **user site** (`qiekn.github.io`), so Vite's `base` stays at the default `/`. Do not set a sub-path base.
- Assets under `public/` are served from the site root (e.g. `/favicon.svg`).

## Notes for future work

- Portfolio/resume content is not yet built — `App.tsx` is a minimal intro card.
- When adding shadcn components, respect the aliases in `components.json` (`@/components`, `@/components/ui`, `@/lib`, `@/hooks`).
- Design exploration happens in `design/design.pen` via the pencil MCP server; the file is encrypted and cannot be read with Read/Grep.

---
> Source: [qiekn/qiekn.github.io](https://github.com/qiekn/qiekn.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
