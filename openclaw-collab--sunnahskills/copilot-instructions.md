## sunnahskills

> Code quality, performance, and security routing (during execution + review)


# Code quality & performance routing

- Keep changes surgical and avoid overengineering: apply karpathy-guidelines.
- If code becomes verbose/boilerplate-y: use ai-slop-cleaner to reduce and tighten.
- If refactoring for clarity without behavior change: use code-simplifier (and keep diffs small).
- For React/UI performance concerns: use vercel-react-best-practices.

# Security routing

- If touching auth, cookies/sessions, payments, webhooks, admin endpoints, secrets, or input validation: invoke security-reviewer before shipping/committing.

# Review routing

- After a meaningful milestone: invoke code-reviewer before finishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw-collab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/openclaw-collab)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
