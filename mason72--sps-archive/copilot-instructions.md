## sps-archive

> AI-powered photo archive for professional photographers. Sister product to SimplePhotoShare (spsv2).

# Pixeltrunk

AI-powered photo archive for professional photographers. Sister product to SimplePhotoShare (spsv2).

## Quick Reference

- **Stack:** Next.js 15, React 19, TypeScript, Tailwind CSS 4, Supabase (pgvector), Cloudflare R2, Modal GPU, Inngest
- **Docs:** `docs/PRD.md` (product), `docs/TECHNICAL.md` (technical), `docs/SESSION-HANDOFF.md` (handoff prompt)
- **Brand:** Elephant pixel-mosaic logo, Libre Baskerville wordmark, Playfair Display headlines, Inter body, stone/white palette with emerald accent

## Project Structure

- `src/app/` — Next.js App Router pages and API routes
- `src/components/` — UI components (button, upload, gallery, search)
- `src/lib/` — Business logic (supabase, r2, ai, upload, sps-integration)
- `modal/` — Python AI pipeline (CLIP, ArcFace, aesthetic scoring)
- `docs/` — PRD and technical documentation

## Key Patterns

- Presigned URL uploads (client -> R2 direct)
- AI processing via Modal serverless GPU
- pgvector for CLIP semantic search
- Smart Stacks group similar images, surface best shot
- Auto Sections from AI scene classification
- SPS integration via shared R2 bucket (zero-copy imports)

## Design System

- **Fonts:** `font-brand` (Libre Baskerville — wordmark only), `font-editorial` (Playfair Display — headlines), `font-sans` (Inter — body)
- **Palette:** Tailwind stone (stone-900 primary, white surfaces, emerald accent)
- **Components:** Button variants (primary, secondary, ghost, danger), BrandButton (animated), lucide-react icons
- **Layout:** CSS columns masonry, `cn()` utility (clsx + tailwind-merge)

## Development

```bash
npm run dev          # Start dev server (port 3000)
npm run build        # Production build
npm run lint         # ESLint
npm run db:gen-types # Regenerate Supabase types
modal deploy modal/ai_pipeline.py  # Deploy AI pipeline
```

## Workflow Orchestration
### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Use agents for exploration and research and have them deliver their findings; this keeps your context window clear
- If something isn't progressing, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity
- Use 'ultrathink' in prompts when maximum reasoning depth is needed
### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution
### 3. Self-Improvement Loop
- After ANY correction from the user: update 'tasks/lessons.md' with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project
### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness
- Consider cross-model QA: review with a different model (e.g., switch to sonnet) to catch blind spots
### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it
- Consider aesthetics always; when adding something new be sure it fits in, make it beautiful
### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Make sure systems have appropriate logging to catch/understand errors when they occur
- Make sure admin users are notified when needed (e.g. system failure/errors, unexpected overspending, etc.)
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management
1. **Plan First**: Write plan to "tasks/todo.md" with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to 'tasks/todo.md'
6. **Capture Lessons**: Update 'tasks/lessons.md' after corrections

## Context & Efficiency
- Keep this file under 150 lines — trim ruthlessly
- Use `/compact` proactively at ~50% context window, don't wait for overflow
- Commit frequently upon task completion — small, atomic commits, but confirm before pushing to main to reduce unneeded deployments
- Break subtasks small enough to complete within 50% context window
- Run background terminal tasks for live log capture during debugging
- Always attach screenshots when reporting visual issues
- When instructions grow, split into `.claude/rules/<topic>.md` — don't bloat this file
- Name sessions with `/rename` for easy `/resume` — enables cross-session continuity
- Pre-approve common permissions: `Bash(pnpm *)`, `Edit(apps/**)`, `Bash(git *)`

## Core Principles
- **Simplicity First**: Make every change as simple as possible. Impact minimal code. Do not break working code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards. These products need to work at scale.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- **Build it beautiful.** Every detail should be scrutinized for polish and quality. Simple, elegant, beautiful. Sweat the small stuff.

---
> Source: [mason72/sps-archive](https://github.com/mason72/sps-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
