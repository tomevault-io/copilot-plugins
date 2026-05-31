## personal-site-ai-template

> Build a reusable, production-quality modular personal website template with an AI-powered personal knowledge assistant.

# AGENTS.md

## Mission
Build a reusable, production-quality modular personal website template with an AI-powered personal knowledge assistant.

## Working style
- Work in large coherent batches.
- Do not stop after every minor step.
- Stop only at meaningful batch boundaries.
- Make reasonable assumptions for non-critical UI/content choices.
- Ask only when blocked by an irreversible architecture or product decision.

## Project priorities
1. Working product over overengineering
2. Clean architecture over quick hacks
3. Reusable template over one-off personal customization
4. Source-grounded AI behavior over flashy but unreliable chat
5. Good UX over excessive animation

## Stack
- Next.js
- TypeScript
- Tailwind CSS
- File-based content
- Lightweight AI API layer

## Build / run / check
- Install dependencies
- Run dev server
- Run typecheck
- Run lint
- Run build before ending a batch

## Engineering rules
- Keep components small and composable.
- Prefer typed data models.
- Separate content from presentation.
- Separate retrieval/orchestration/provider logic clearly.
- Avoid hardcoding one user’s profile into core template code.
- Avoid tightly coupling AI UI to one provider.
- Prefer graceful fallbacks.

## AI assistant rules
- Never fabricate facts not present in content/config.
- Always support uncertainty gracefully.
- Treat the assistant as representing public materials, not private commitments.
- Include source references or source labels when available.
- Route visitors to real pages/actions when appropriate.

## Definition of done for a batch
A batch is done only if:
- app runs
- changed pages/components render
- no major broken navigation
- no obvious type or lint failures in changed paths
- relevant docs/config/examples are updated
- summary includes changed files, known risks, and next step

## Do-not rules
- Do not add unnecessary heavy dependencies without strong reason.
- Do not leave dead code or duplicate component systems.
- Do not build fake AI without retrieval/provider structure.
- Do not optimize prematurely.
- Do not pause merely to ask aesthetic micro-questions.

---
> Source: [GuoChencheng/personal-site-ai-template](https://github.com/GuoChencheng/personal-site-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
