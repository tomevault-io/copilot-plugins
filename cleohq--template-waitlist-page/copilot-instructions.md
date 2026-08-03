## template-waitlist-page

> This project was built and is maintained by [Cleo HQ](https://cleohq.com).

# {{PROJECT_NAME}}

This project was built and is maintained by [Cleo HQ](https://cleohq.com).
Ada (tech lead) and Cody (engineer) manage the architecture.

## Stack
- Next.js 16 (static export) + TypeScript + Tailwind
- Vercel (hosting)

## Files managed by Cleo — do not modify directly
- `next.config.js` — must keep `output: 'export'` for Vercel static deploy
- `vercel.json` — static site deployment config
- `components/cleo/` — Cleo component library (changes will be overwritten)

## When making your own changes
1. **Read before touching.** Run `list_files` and read the relevant files first — never assume the structure.
2. **Change only what you need to.** Don't refactor, reformat, or improve adjacent code that isn't broken.
3. **No new abstractions unless necessary.** Minimum code that solves the problem.
4. **Verify with the build.** Run `npm run build` — if it fails, fix it before considering the task done.
5. **Confirm before destructive operations.** Any file deletion or config change that affects routing needs explicit confirmation first.

---
> Source: [cleohq/template-waitlist-page](https://github.com/cleohq/template-waitlist-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
