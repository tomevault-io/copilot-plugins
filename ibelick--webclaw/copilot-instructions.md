## webclaw

> WebClaw app lives in `apps/webclaw` and is built with React + TanStack Router + Tailwind CSS v4.

# AGENTS

## Overview

WebClaw app lives in `apps/webclaw` and is built with React + TanStack Router + Tailwind CSS v4.

## Commands

- `pnpm -C apps/webclaw dev` — Start development server
- `pnpm -C apps/webclaw build` — Build for production
- `pnpm -C apps/webclaw preview` — Preview production build
- `pnpm -C apps/webclaw test` — Run tests
- `pnpm -C apps/webclaw lint` — Run ESLint
- `pnpm -C apps/webclaw format` — Run Prettier
- `pnpm -C apps/webclaw check` — Format and lint fix

## Conventions

### Code Style

- **Functions**: Always use the `function` keyword. Avoid `const` for function definitions.
- **Types**: Always use `type T = { ... }`. Do not use `interface`.
- **File Naming**: Use `kebab-case` for all files (e.g., `chat-screen.tsx`, `use-session.ts`).
- NEVER use useEffect for anything that can be expressed as render logic
- MUST use cn utility (clsx + tailwind-merge) for class logic

### Routing & Structure

- Routes live in `apps/webclaw/src/routes` using TanStack file routing.
- Global styles and CSS variables live in `apps/webclaw/src/styles.css`.
- Local environment values go in `apps/webclaw/.env.local`.

### UI & Styling

- **Typography**: Never use font weights bolder than `font-medium`. Apply small negative tracking (`tracking-tight` or similar) on main titles.
- **Colors**:
  - Use the custom Tailwind palette (e.g., `bg-primary-50`, `text-primary-900`).
  - Never use arbitrary color values.
  - Avoid `bg-white`, `bg-black`, `text-white`, `text-black`, and `outline-black`; use primary palette tokens instead.
- **Markdown Titles**: Avoid top margin on markdown headings.
- MUST use text-balance for headings and text-pretty for body/paragraphs
- MUST use tabular-nums for data
- SHOULD use truncate or line-clamp for dense UI
- NEVER modify letter-spacing (tracking-\*) unless explicitly requested
- MUST use a fixed z-index scale (no arbitrary z-\*)
- SHOULD use size-_ for square elements instead of w-_ + h-\*
- **Icons**:
  - All icons should use `size={20}` and `strokeWidth={1.5}` consistently
- **React 19 Refs**: Use regular `function` components with direct ref passing instead of `React.forwardRef` (React 19 supports refs as regular props)

### Performance

- Avoid chat-wide rerenders while streaming: memoize large UI blocks and pass stable callbacks.
- Prefer passing derived data (maps/ids) instead of whole arrays when only lookups are needed.
- Keep prompt input state local to the composer when possible to avoid chat-wide rerenders on keystrokes.
- Memoize message rows with content-based equality and avoid passing freshly created objects that bust memoization.
- When scroll containers host frequently-updating content, memoize the scroll shell and portal the changing content to reduce root rerenders.
- Keep scroll position state inside scroll controls; avoid context state that forces scroll shells to rerender.

### Optimistic Updates

- For chat messages, write optimistic items directly into the history cache and reconcile when server history arrives (clientId/near-timestamp matching).
- For session rename/delete, optimistically update the sessions cache in mutation `onMutate`, rollback on error, then invalidate on success.

---
> Source: [ibelick/webclaw](https://github.com/ibelick/webclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
