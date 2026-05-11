## mellow-lines

> Don't run lint and build check on every change, only before making a commit.

# Repository Guidelines

## Personal adjustments

Don't run lint and build check on every change, only before making a commit.

## Core Philosophy

**Teach, don't just tell. Explain, don't just execute.**

When helping with code, architecture, or problem-solving, prioritize education. Every solution should leave me (the user) with deeper understanding.

---

## Response Structure

### 1. Start with the "Why"
Before showing code or solutions:
- What problem are we solving?
- Why does this approach make sense?
- What are the trade-offs?

### 2. Explain APIs & Tools Used

For every API, library, or tool mentioned:

```markdown
**API: [Name]**
- Purpose: What does it do?
- Key Methods/Properties: [list important ones]
- When to use: [context]
- Common pitfalls: [gotchas]
- Documentation: [link to official docs]
```

## Git workflow

When asked to commit, always:
1. Create a new branch from the current branch if currently on main, if not - use current
2. Commit the changes on the branch
3. Push the branch and create a pull request if not already created for current branch

Don't commit or created a new branch / pull request until explicitly asked to do this. 

## Project Structure & Module Organization
This repository is a Next.js 16 App Router project for creating animated code walkthroughs. Routes live in `app/`, with the landing page in `app/page.tsx` and the editor in `app/editor/page.tsx`. Shared animation, audio, typing, and export logic lives under `app/lib/` (`magicMove/`, `video/`, `audio/`, `typing/`). Reusable UI is split between feature components in `components/` and primitive controls in `components/ui/`. Static media and SVG assets are stored in `public/`.

## Build, Test, and Development Commands
Use the scripts in `package.json`:

## Coding Style & Naming Conventions
Write TypeScript with strict typing and use the `@/*` import alias for local modules. Follow the existing style: double quotes, semicolons, and 2-space indentation as enforced by Oxfmt. Use `PascalCase` for React components (`PreviewPanel`), `camelCase` for functions and hooks (`useTypingSound`), and descriptive file names in kebab case for components (`steps-editor.tsx`). Keep route files in Next defaults like `page.tsx` and `layout.tsx`.


## Configuration Tips
Linting is configured in `.oxlintrc.json` and formatting in `.oxfmtrc.json`. Keep large media assets in `public/`, and avoid checking in generated output from `.next/` or temporary export files.

---
> Source: [kostyniuk/mellow-lines](https://github.com/kostyniuk/mellow-lines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
