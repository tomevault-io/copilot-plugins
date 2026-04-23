## openguider

> This file helps both human contributors and agentic coding AIs work safely and consistently in OpenGuider.

# AGENTS.md

This file helps both human contributors and agentic coding AIs work safely and consistently in OpenGuider.

## Project Snapshot

- Stack: Electron + Node.js (CommonJS), vanilla HTML/CSS/JS renderer.
- Entry points: `main.js` (main process), `preload.js` (IPC bridge), `renderer/` (UI).
- Core domains:
  - `src/ai/` provider integrations and streaming logic
  - `src/agent/` planning, orchestration, structured tool flows
  - `src/session/` session state and persistence
  - `src/tts/` text-to-speech providers
  - `src/store.js` + `src/secure-store.js` app settings and secret handling

## Local Development

- Install deps: `npm install`
- Run app: `npm run start`
- Run tests: `npm test`
- Build artifacts:
  - `npm run dist:win`
  - `npm run dist:mac`
  - `npm run dist:linux`

## Contribution Rules

- Keep changes focused; avoid unrelated refactors in the same PR.
- Preserve backward compatibility for existing settings keys whenever possible.
- Do not commit secrets, tokens, personal data, or local machine artifacts.
- If touching provider logic, keep fallback defaults explicit and robust.
- Update docs/UI text when behavior changes (especially settings and provider fields).

## AI Agent Guardrails

- Read nearby code before editing; follow existing naming and style conventions.
- Prefer minimal, surgical changes over broad rewrites.
- Do not remove existing user-facing features unless explicitly requested.
- Do not run destructive git commands (`reset --hard`, force push, history rewrite).
- Respect current branch/worktree state; do not revert unrelated local changes.
- Validate impacted flows with tests or targeted manual verification steps.

## Testing Expectations

- At minimum, run `npm test` after meaningful code changes.
- When modifying provider behavior, ensure AI provider tests still pass.
- If tests fail due to environment/dependency setup rather than your change, state that clearly in PR notes.

## PR Checklist

- Clear title and short "why" summary.
- List user-visible changes (settings, provider behavior, UI, shortcuts, permissions).
- Mention validation performed (`npm test`, manual checks).
- Call out any limitations, follow-ups, or non-blocking risks.

## Security and Data Notes

- Secrets should flow through secure storage (`secure-store`) instead of plain config writes.
- Be careful with logging around prompts, keys, and provider responses.
- Prefer safe defaults for endpoints and model/provider fallbacks.

## When in Doubt

- Choose the least risky implementation.
- Leave concise comments only where logic is non-obvious.
- Ask for clarification before making architectural changes.

---
> Source: [mo-tunn/OpenGuider](https://github.com/mo-tunn/OpenGuider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
