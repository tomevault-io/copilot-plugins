## resume-pilot

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Project Notes

- This is a Next.js resume builder and AI tailoring app. Keep changes small, scoped, and consistent with existing feature folders.
- Do not commit secrets. Runtime keys live in `.env`; public setup guidance belongs in `.env.example`.
- Use `npm run lint` and `npx tsc --noEmit` before merging public-facing changes.
- AI provider behavior is configured through `AI_PROVIDER` plus the matching OpenAI or Anthropic environment variables.

---
> Source: [GAaronZhao/Resume-Pilot](https://github.com/GAaronZhao/Resume-Pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
