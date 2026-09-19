## jev-mac-voice

> Use the installed TypeSafe skill at `.agents/skills/typesafe-ai/SKILL.md` when working on Jev decisions or TypeSafe integration. Read the relevant live TypeSafe documentation before changing API contracts, questions, or confidence policies.

# Project instructions

Use the installed TypeSafe skill at `.agents/skills/typesafe-ai/SKILL.md` when working on Jev decisions or TypeSafe integration. Read the relevant live TypeSafe documentation before changing API contracts, questions, or confidence policies.

Keep OpenAI Realtime responsible for English voice conversation; keep execution, access checks, approvals, cancellation and context freshness in code. Jev returns typed judgments, not permission to execute an action.

Run `npm test` for execution changes, `npm run test:ui` for voice UI changes, and `npm run build:native` for Swift changes. Do not claim live audio or desktop input was verified when only mocks ran. Never commit `.env`, browser profiles, recordings, or native build output.

The desktop application uses Electron with a sandboxed React renderer. Run `npm run build:ui`, `npm run test:react`, and `npm run test:electron` for desktop UI changes. Never return stored API credentials to the renderer or persist them there; save user-entered keys through the narrow Electron IPC using macOS safeStorage. The live smoke test is paid and explicitly opt-in.

---
> Source: [brudarko/jev-mac-voice](https://github.com/brudarko/jev-mac-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
