## magnus-flipper-ai-v1-0-pro-reset

> Before allowing any git push, merge, or Vercel deployment:

# 🧯 Deployment Safety Rule (Non-Negotiable)

Before allowing any git push, merge, or Vercel deployment:

REQUIRED CHECKS:
- Local `pnpm build` completed successfully
- No TypeScript errors
- No ESLint errors
- No duplicate imports
- No concatenated imports
- AuthProvider is imported exactly once and used correctly
- Auth hooks (`useAuth`, `useRouter`, etc.) are valid

If ANY check fails:
- Do NOT provide git commands
- Do NOT approve deployment
- Explain the failure and how to fix it

This rule cannot be bypassed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chiosemen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
