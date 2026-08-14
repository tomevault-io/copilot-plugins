## devadvokit

> This repo contains AI agent skills for Developer Relations practitioners. Skills are installed via the `npx skills` CLI and run inside Claude Code or compatible agents.

# DevAdvokit — Agent Guide

This repo contains AI agent skills for Developer Relations practitioners. Skills are installed via the `npx skills` CLI and run inside Claude Code or compatible agents.

---

## What this is

DevAdvokit provides skills for DevRel practitioners:

- `/setup-devadvokit` — one-time setup that writes your professional context to `~/.devadvokit.md`
- `/generate-cfp` — generates a complete conference proposal through a structured Q&A
- `/generate-bio` — generates a speaker or author bio with controls for length, person, and employer focus
- `/build-talk-outline` — turns a CFP submission into a timed outline with demo risk assessments and Patrick Winston principles applied throughout

All skills read `~/.devadvokit.md` at runtime. If it's missing, they stop and redirect to `/setup-devadvokit`.

---

## Repository structure

```
devadvokit/
├── skills/
│   ├── setup-devadvokit/
│   │   ├── SKILL.md
│   │   └── reference/
│   │       └── devadvokit-md-schema.md
│   ├── generate-cfp/
│   │   ├── SKILL.md
│   │   └── reference/
│   │       ├── cfp-evaluation-criteria.md
│   │       └── cfp-anti-patterns.md
│   ├── generate-bio/
│   │   ├── SKILL.md
│   │   └── reference/
│   │       └── bio-contexts.md
│   └── build-talk-outline/
│       ├── SKILL.md
│       └── reference/
│           ├── outline-structure-patterns.md
│           ├── demo-risk-patterns.md
│           └── winston-talk-principles.md
├── shared/
│   └── ai-antipatterns.md
├── AGENTS.md
├── README.md
├── LICENSE
└── .gitignore
```

No build step. The `skills/` directory is the single source of truth and the install target. Edit files directly — what's in the repo is what gets installed.

---

## Adding a new skill

1. Create `skills/{name}/SKILL.md` with the following frontmatter:

```markdown
---
name: your-skill-name
description: One-sentence description of what the skill does.
---
```

Only `name` and `description` in frontmatter. Nothing else.

2. Add a `reference/` subdirectory inside the skill folder if you need supporting documents. Reference files are copied alongside the skill on install.

3. Include the context check block at the top of the skill body (unless the skill is `/setup-devadvokit` itself):

```
Before doing anything else:
1. Check if `~/.devadvokit.md` exists.
2. If it does, read it silently and use it throughout this skill.
3. If it does not, stop and tell the user: "I need your DevRel context before I can run this skill. Please run /setup-devadvokit first."
```

4. Include the anti-patterns instruction at the end of the output section:

```
Before presenting any output, read `../../shared/ai-antipatterns.md` and silently rewrite any flagged patterns. Do not mention this step to the user.
```

5. Reference skill-specific files using relative paths from the skill directory (e.g. `reference/my-reference.md`) and shared files from the repo root (e.g. `../../shared/ai-antipatterns.md`).

---

## The `~/.devadvokit.md` file

Written by `/setup-devadvokit` to the user's home directory. Contains:

- **Identity** — name, current role, current company, company one-liner, years in DevRel, location
- **Background** — current role specialisms, previous roles, career-wide specialisms
- **Audience** — primary audience description
- **Content** — platforms, content pillars, writing voice, words/styles to avoid
- **Personal** — hobbies and interests (used by `/generate-bio`)
- **Content Library** — table of past talks, posts, videos, and other content

The file is never overwritten without user confirmation. On subsequent `/setup-devadvokit` runs, the skill offers to update individual sections or append to the content library.

The full field-level schema — including what a strong vs. thin answer looks like for each field — is in `skills/setup-devadvokit/reference/devadvokit-md-schema.md`.

---

## Shared reference files

`shared/ai-antipatterns.md` is a rewrite checklist applied silently to all output before it's shown to the user. It covers word choice, sentence patterns, and tone to eliminate. Every skill must run this pass before presenting output. Do not mention it to the user.

---

## No build step

Edit `skills/` directly. There is no `dist/`, no `source/`, no compilation. The `npx skills` CLI reads from `skills/` and handles provider detection and placement.

---
> Source: [phazonoverload/devadvokit](https://github.com/phazonoverload/devadvokit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
