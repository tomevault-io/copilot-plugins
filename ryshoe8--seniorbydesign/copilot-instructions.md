## git-only-workflow

> No local-dev instructions for the user; changes ship via git push only


# Git-only workflow (seniorbydesign)

- **Do not** tell the user to run local dev commands as something *they* must do: `npm install`, `npm run dev`, `npm run build`, `npm run lint`, starting servers, or “verify locally first.”
- **Assume** the user **only pushes to Git**; deployment/build (e.g. Vercel) happens remotely.
- **Do** make dependency and code changes in the repo (`package.json`, etc.) so remote installs succeed.
- **You may** run terminal commands in the agent environment when available to verify or update the repo, but **never** end with a checklist asking the user to run those same steps locally.

---
> Source: [RyShoe8/seniorbydesign](https://github.com/RyShoe8/seniorbydesign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
