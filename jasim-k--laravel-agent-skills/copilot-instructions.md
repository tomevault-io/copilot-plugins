## laravel-agent-skills

> This repository contains Claude Code skills for Laravel, Vue 3, and full-stack web development.

# Agent Guide — Claude Code Skills

This repository contains Claude Code skills for Laravel, Vue 3, and full-stack web development.

## Available Skills

| Skill | Use When |
|-------|----------|
| [laravel-inertia-vue](laravel-inertia-vue) | Building Laravel + Inertia.js + Vue 3 applications |

## How Skills Work

Each skill lives in its own directory with:

```
skill-name/
├── SKILL.md          # Overview and quick-reference examples
├── AGENTS.md         # Agent-specific guidance for this skill
├── README.md         # Human-readable documentation
├── metadata.json     # Skill metadata and rule index
└── rules/            # Individual rule files
    ├── _sections.md  # Priority guide
    └── *.md          # One rule per file
```

Claude loads a skill's rules when you copy the skill into `.claude/skills/` in your project.

## Skill: laravel-inertia-vue

**Use this skill when:** building any page, form, or navigation component in a Laravel + Inertia.js + Vue 3 project.

### Key Vue 3 Patterns

| Concept | Pattern |
|---------|---------|
| Component format | `<script setup lang="ts">` + `<template>` |
| Props typing | `defineProps<Props>()` |
| Layout assignment | `defineOptions({ layout: AppLayout })` |
| Form binding | `v-model="form.field"` |
| Shared props | `usePage<PageProps>().props` |
| Reactive state | `ref`, `computed`, `watch` |
| Layout slot | `<slot />` |

### Rule Priorities

- **CRITICAL** — Always follow. Violating these causes broken behavior.
- **HIGH** — Follow unless there is a strong reason not to.
- **MEDIUM** — Best practice; apply when it fits.

### Rule Categories

| Category | Files | What It Covers |
|----------|-------|----------------|
| Page Components | `rules/page-*.md` | Structure, props, `<Head>`, scroll, partial reloads |
| Forms & Validation | `rules/form-*.md` | `useForm`, validation errors, file uploads, dirty tracking |
| Navigation | `rules/nav-*.md` | `<Link>`, `router.*`, external links, history |
| Shared Data | `rules/shared-*.md` | Auth, flash messages, Ziggy routes, app config |
| Layouts | `rules/layout-*.md` | Persistent layouts, nested layouts |

## Adding New Skills

When a new skill is added to this repository:

1. Create a new directory under `skills/` (or at the repo root alongside `laravel-inertia-vue`)
2. Follow the same structure: `SKILL.md`, `AGENTS.md`, `README.md`, `metadata.json`, `rules/`
3. Add a row to the skills table in this file and in `README.md`
4. Keep each rule file focused on a single, testable pattern

---
> Source: [jasim-k/laravel-agent-skills](https://github.com/jasim-k/laravel-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
