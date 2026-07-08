## bbl-ai-lab

> This repository is an automation flow hub. Telegram commands enter through the Cloudflare Worker, the Worker dispatches GitHub Actions, and the Actions run scripts/prompts that create issues, reports, or other outputs.

# bbl-ai-lab Agent Guide

This repository is an automation flow hub. Telegram commands enter through the Cloudflare Worker, the Worker dispatches GitHub Actions, and the Actions run scripts/prompts that create issues, reports, or other outputs.

Read this guide before changing repository-level automation, Worker routing, docs, or flow files.

## Source Context

Use these files as the source of truth:

- `CONTEXT.md` for project language.
- `README.md` for the public operating model and setup.
- `docs/plans/README.md` for plan document workflow.
- `worker/src/flows.ts` for automation flow registration.

## Flow Registry Rules

- Start every new automation flow in `worker/src/flows.ts`.
- Treat `worker/src/flows.ts` as the single manifest for Telegram commands, subcommands, `repository_dispatch` event types, and adapter file paths.
- Do not add a workflow, script, prompt, or flow doc without also adding or updating the manifest entry.
- Keep new `eventType` values equal to `<flow>-<action>` unless preserving an explicit legacy event. The only current legacy event allowed by the checker is `idea-submitted`.
- For subcommand flows, use the shape `/command subcommand body`; put action-specific workflow/script/prompt/docs paths inside `subcommands`.
- After flow changes, run:

```bash
cd worker
npm run typecheck
```

This runs TypeScript checking and verifies manifest drift against workflow `repository_dispatch.types` plus workflow/script/prompt/docs file existence.

## Current Flow Layout

- Worker runtime: `worker/src/index.ts`
- Flow manifest and routing helpers: `worker/src/flows.ts`
- Flow consistency checker: `worker/scripts/check-flows.mjs`
- GitHub Actions workflows: `.github/workflows/<flow>-<action>.yml`
- Runtime scripts: `scripts/<flow>-<action>.sh`
- Gemini prompts: `scripts/prompts/<flow>-<action>.md`
- Flow docs: `docs/<flow>-<action>.md`
- Skill context attached by Actions: `skills/*.SKILL.md`
- Operator tooling that is not a Telegram/GitHub Actions flow belongs in the relevant submodule, such as `hermes-workspace/`, not in this repo's top-level `scripts/`.

## Documentation Rules

- Update `README.md` when the repository operating model, setup flow, or user-facing flow list changes.
- Update `CONTEXT.md` when a new project term becomes load-bearing for future plans or code reviews.
- Update `docs/<flow>-<action>.md` when a specific flow's trigger, environment, output, or verification changes.
- Update `knowledge/flows/<flow>-<action>.md` when an automation flow is added or its relationships change.
- Update `knowledge/concepts/` when a load-bearing project concept is added or materially redefined.
- Keep every `knowledge/**/*.md` file in Open Knowledge Format style: YAML frontmatter at the top with `type` required; prefer `title`, `description`, `resource`, `tags`, and `timestamp` when available.
- Update `docs/plans/README.md` only when the plan document workflow changes.

## Safety Rules

- Never commit secrets, OAuth credentials, Telegram bot tokens, GitHub tokens, SSH keys, `.env` files, or remote operator workspace secrets.
- Keep `worker/wrangler.toml` free of secrets; use `wrangler secret put` for sensitive values.
- Do not remove generated or local worktree directories unless the user explicitly asks.
- Existing untracked files may belong to the user. Stage only intentional files when asked to commit.

## Verification Before Finishing

For Worker or flow registry changes:

```bash
cd worker
npm run typecheck
```

For shell script changes:

```bash
bash -n scripts/<changed-script>.sh
```

For docs-only changes, inspect the diff and make sure links/paths still match the manifest.

For OKF knowledge bundle changes:

```bash
rg -n "^type:" knowledge
find knowledge -name "*.md" | sort
```

---
> Source: [BoBeenLee/bbl-ai-lab](https://github.com/BoBeenLee/bbl-ai-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
