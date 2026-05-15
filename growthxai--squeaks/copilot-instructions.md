## squeaks

> A lo-fi prototyping tool. Users describe ideas (plain text, ASCII wireframes, screenshots) and you turn them into navigable wireframe pages. Think napkin sketches in the browser — not production UI.

# Squeaks — Claude Code Guide

## What this project is

A lo-fi prototyping tool. Users describe ideas (plain text, ASCII wireframes, screenshots) and you turn them into navigable wireframe pages. Think napkin sketches in the browser — not production UI.

**The goal is speed and clarity, not polish.**

## Stack

Vite + React 19 + TypeScript + React Router + Tailwind CSS v4 + Shadcn UI

No backend. No database. No API calls. Everything is frontend-only with static placeholder data.

## Core rules

- **Monochrome only** — black, white, and grays. No colors unless the user explicitly asks
- **Comic Neue font** — don't change it. It signals "this is a mockup"
- **Use Shadcn UI components** from `@/components/ui/` — don't install new UI libraries
- **Don't modify `src/components/ui/`** — these are Shadcn defaults
- **Placeholder data only** — use realistic fake data (names, numbers, dates), not "Lorem ipsum"
- **No real functionality** — no API calls, no auth, no databases, no localStorage persistence
- **No animations** — transitions and animations are disabled globally in CSS
- **Keep it simple** — flat components, minimal abstraction, no over-engineering

## File conventions

- **kebab-case** for all filenames: `project-dashboard.tsx`, not `ProjectDashboard.tsx`
- **Pages** go in `src/pages/` (or `src/pages/v1/`, `src/pages/v2/` for versions)
- **Routes** are defined in `src/main.tsx`
- **Shared components** (non-Shadcn) go in `src/components/`
- **Layout and nav** live in `src/layout.tsx`
- Use `@/` import alias for all project imports

## Routing

Use React Router (`react-router`). All routing is client-side.

**When adding a new page, always:**

1. Create the page component in `src/pages/`
2. Add the route in `src/main.tsx`
3. Add a nav link in `src/layout.tsx` if it's a top-level page

**Version pattern** — users often want to compare approaches:

```
/v1/dashboard    ← version 1
/v2/dashboard    ← version 2 with different layout
```

Organize versioned pages in matching folders:

```
src/pages/
├── v1/
│   └── dashboard.tsx
└── v2/
    └── dashboard.tsx
```

## When the user shares a wireframe or idea

1. Create the page component in `src/pages/`
2. Add the route to `src/main.tsx`
3. Build the layout using Shadcn components (Card, Table, Button, Input, Tabs, etc.)
4. Use realistic placeholder text and data — real-sounding names, plausible numbers
5. Prioritize layout structure and information hierarchy over detail
6. Make interactive elements look clickable but they don't need to do anything
7. If the wireframe implies navigation between pages, create all the pages and link them

## What NOT to do

- Don't add colors, gradients, brand colors, or themes
- Don't install new npm packages unless absolutely necessary
- Don't build real functionality (forms that submit, data that persists, APIs)
- Don't create abstractions for one-off pages — copy-paste is fine for prototypes
- Don't add loading states, error boundaries, or edge case handling
- Don't add comments explaining obvious layout code
- Don't refactor or "clean up" prototype pages unless asked

## Available Shadcn components

All standard Shadcn UI components are installed and available at `@/components/ui/`:

Accordion, Alert, AlertDialog, AspectRatio, Avatar, Badge, Breadcrumb, Button, Calendar, Card, Carousel, Chart, Checkbox, Collapsible, Command, ContextMenu, Dialog, Drawer, DropdownMenu, Form, HoverCard, Input, InputOTP, Label, Menubar, NavigationMenu, Pagination, Popover, Progress, RadioGroup, Resizable, ScrollArea, Select, Separator, Sheet, Sidebar, Skeleton, Slider, Sonner, Switch, Table, Tabs, Textarea, Toast, Toggle, ToggleGroup, Tooltip

## Package manager

Use `npm`.

## Dev server

```bash
npm run dev     # Start at http://localhost:5173
npm run build   # Production build
```

---
> Source: [growthxai/squeaks](https://github.com/growthxai/squeaks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
