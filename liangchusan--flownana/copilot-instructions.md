## flownana

> This repository is a 0→1 AI generation product. Read MEMORY.md before changing code.

# AGENTS.md

This repository is a 0→1 AI generation product. Read MEMORY.md before changing code.

Goals:
- ship fast
- keep MVP scope tight
- support paid acquisition validation
- avoid unnecessary complexity

Working rules:
- Treat MEMORY.md as the current source of truth for product scope, tracking, and acceptance.
- Do not redefine requirements on your own.
- Prefer the smallest implementation that satisfies the approved scope.
- Keep image generation flow stable before expanding video/music scope.
- Include required GA4 tracking from MEMORY.md when touching landing, auth, pricing, checkout, generation, or result flows.
- If product logic, funnel logic, or tracking changes, update MEMORY.md before finishing.
- Do not claim tests passed unless they were actually run.
- Do not deploy production without explicit approval.

UI and frontend guardrails (cross-tool baseline):
- Follow `.cursor/rules/frontend-design.mdc` as the source of truth for UI style and frontend architecture.
- Visual tone: warm, trustworthy, minimal. Prefer Stone/Zinc palette; avoid Slate/Gray drift.
- Prefer shadcn primitives in `@/components/ui/`; compose business sections in `@/components/blocks/`.
- Keep `@/app/` page files focused on data + composition; avoid long page-level UI logic.
- No inline `style={{}}`; use Tailwind classes only.
- Mobile-first responsive layouts; avoid hard-coded fixed widths like `w-[800px]`.
- Keep interaction consistency: `transition-all duration-300`, clear hover/active/focus states.
- Use `lucide-react` icons with consistent sizing (`w-4 h-4` / `w-5 h-5`) and muted Stone tones.

Before finishing any task:
1. summarize what changed
2. summarize what to test manually
3. summarize remaining risks
4. update MEMORY.md if key product/flow/tracking decisions changed

---
> Source: [liangchusan/Flownana](https://github.com/liangchusan/Flownana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
