## hackathon-fullstack-template

> Before editing, read `/AGENTS.md`. It is the repository map and lists the

# SPARK repository instructions

Before editing, read `/AGENTS.md`. It is the repository map and lists the
task-specific source documents.

- Preserve the FastAPI authorization boundary and user ownership filters.
- Keep JWTs in the Next.js BFF HttpOnly cookie; never move them to localStorage.
- Never expose secrets, passwords, tokens, or `hashed_password`.
- Treat user-provided AI input and model output as untrusted data.
- Do not describe the current single Solar completion as an autonomous agent.
- For structural changes read `/DESIGN.md` and `/docs/ARCHITECTURE.md`.
- For AI changes read `/docs/AI_FEATURE_GUIDE.md`,
  `/docs/PROMPT_ENGINEERING.md`, `/docs/AI_EVALUATION.md`, and
  `/docs/AI_SAFETY.md`.
- Run backend tests and frontend lint/build when the touched boundary requires it.
- Do not reformat or modify unrelated user changes.

---
> Source: [KimJinwoo2198/Hackathon-Fullstack-Template](https://github.com/KimJinwoo2198/Hackathon-Fullstack-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
