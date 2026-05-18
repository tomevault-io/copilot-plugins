## frontend-developer-workspace

> Repo for standalone frontend projects and experiments.

# Frontend Developer Project

Repo for standalone frontend projects and experiments.

## Structure

- `projects/` — Organized by tech stack (01-vanille, 02-react, 03-nextjs, etc.)
  - Each numbered folder groups projects by framework
  - Projects inside are self-contained (no shared tooling)
- `learning/` — Experiments, tutorials, new technologies
- `evolution/` — Career tracking, goals, certifications
- `resources/prompts.md` — Prompts used to generate projects
- `.opencode/` — OpenCode config (has `@opencode-ai/plugin`), not a project package
  - `.opencode/skills/index.md` — Full skills index

## Available Skills

See `.opencode/skills/index.md` for the complete list. Key custom skills:
- `project-scaffolder` — Scaffold new projects in correct tech folder
- `prompt-archivist` — Document AI prompts in `resources/prompts.md`
- `accessibility-auditor` — WCAG audit on frontend projects
- `case-study-generator` — Generate portfolio case studies
- `client-onboarding` — Briefs, quotes, estimates for freelance clients
- `tech-watch` — Organize `learning/` and track tech evolution

## Workflow

See `docs/workflow-template.md` for the complete freelance frontend workflow (6 phases: Strategy → Scope → Architecture → Wireframes → Visual Design → Testing).

Use with any new project: copy to `projects/{tech}/{client}/WORKFLOW.md` and follow the checklist.

## Notes

- No build system, no tests, no package manager at root — these are static HTML files
- Each project in `projects/` is self-contained; don't assume shared tooling
- `.opencode/package.json` is for the OpenCode plugin only, not project dependencies
- For frontend fundamentals, see `docs/frontend-fundamentals.md`
- For workflow template, see `docs/workflow-template.md`

---
> Source: [tawounfouet/frontend-developer-workspace](https://github.com/tawounfouet/frontend-developer-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
