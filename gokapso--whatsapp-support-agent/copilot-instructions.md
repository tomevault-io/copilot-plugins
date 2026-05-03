## whatsapp-support-agent

> This file is for agents landing in `whatsapp-support-agent` for the first time.

# AGENTS.md

This file is for agents landing in `whatsapp-support-agent` for the first time.

## Repo Purpose

This repo manages one Kapso example:

- one workflow: `whats-app-support-agent-example`
- one private function tool: `whatsapp-support-agent-ask-team-question`
- one public webhook function: `whatsapp-support-agent-slack-events`
- one inbound WhatsApp trigger on `WHATSAPP_PHONE_NUMBER_ID`

The runtime flow is:

1. WhatsApp message starts the workflow.
2. The agent node answers directly when confident.
3. If uncertain, the agent calls `ask_team_question`, tells the user it is checking internally, then calls `enter_waiting`.
4. The support team replies in a Slack thread and sends `done`.
5. `whatsapp-support-agent-slack-events` aggregates the thread replies and resumes the waiting workflow.

## Setup Flow

When a user asks to set up everything, do this:

1. Read `README.md`.
2. Run `bun install`.
3. Copy `.env.example` to `.env.local`.
4. Fill in `KAPSO_API_KEY`, `WHATSAPP_PHONE_NUMBER_ID`, `SLACK_BOT_TOKEN`, `SLACK_SIGNING_SECRET`, `SLACK_CHANNEL_ID`, and optional `PROVIDER_MODEL_NAME`.
5. Run `bun run validate` and `bun test`.
6. Ensure `kapso` is on `PATH`. For the unpublished local checkout, run `export PATH="$PWD/node_modules/.bin:$PATH"`.
7. Run `kapso link`.
8. Run `kapso push`.
9. Run `bun run sync:secrets`.
10. Copy the printed Slack events URL into the Slack app Event Subscriptions page.

The repo is not fully configured until Slack points to the printed webhook URL and the bot is invited to `SLACK_CHANNEL_ID`.

## Source Layout

- `functions/whatsapp-support-agent-ask-team-question/index.js`: self-contained uploaded Worker handler.
- `functions/whatsapp-support-agent-slack-events/index.js`: self-contained uploaded Worker handler.
- `functions/*/function.yaml`: CLI function metadata.
- `workflows/whats-app-support-agent-example/workflow.ts`: source workflow built with `@kapso/workflows`.
- `scripts/sync-secrets.js`: temporary per-function secret sync helper.
- `src/lib/`: local helpers used by the workflow source, validation, and secret sync.

The generated workflow files `workflows/*/workflow.yaml` and `workflows/*/definition.json` are ignored in this scaffold because `workflow.ts` is the source of truth.

The CLI contract is simple: `workflow.ts` must default-export a `Workflow` instance. This scaffold also exports `buildWorkflow()` only so tests can rebuild the workflow after changing env vars.

## Important Rules

- Function entrypoints must stay plain `async function handler(request, env)` files.
- Do not add `export default` or `module.exports` to uploaded function files.
- `whatsapp-support-agent-slack-events` must stay `public_endpoint: true`.
- Both functions must stay `invoke_response_mode: passthrough`.
- Use function slugs in workflow source, not function IDs.
- Do not manually edit generated workflow YAML/JSON unless intentionally moving to CLI-only JSON editing.
- Keep function secrets out of git; use `.env.local` and `bun run sync:secrets`.

## Advanced Repository Sandbox

To mount a GitHub repository into the support agent, set these in `.env.local` before `kapso push`:

- `AGENT_SANDBOX_GITHUB_REPO_URL`
- `AGENT_SANDBOX_GITHUB_REPO_BRANCH`, defaults to `main`
- `AGENT_SANDBOX_GITHUB_PAT`, empty is valid for public repos
- `AGENT_SANDBOX_NETWORK_MODE`, defaults to `allow_list`
- `AGENT_SANDBOX_ALLOWED_OUTBOUND_HOSTS`, comma-separated or newline-separated
- `AGENT_SANDBOX_ENABLED`, optional explicit `true` or `false`

This is scaffold-specific workflow code, not a Kapso CLI feature. When sandbox is enabled, `workflow.ts` adds raw agent config and prompt guidance telling the support agent to inspect the mounted repo before escalating repository-specific or code-specific questions to Slack.

## Commands

- `bun run validate`: local safety check for function source and workflow source.
- `bun test`: unit tests for Slack signature handling, KV state, aggregation, handlers, and workflow source.
- `kapso push --dry-run`: preview CLI changes.
- `kapso push`: build workflow source and push functions/workflow/trigger.
- `bun run sync:secrets`: sync function secrets after the functions exist remotely.

## Recommended Extra Context

If you are using Codex or another skills-aware agent and want stronger Kapso-specific context, install:

`npx skills add gokapso/kapso-agent-skills`

This is not required for normal setup, but it helps when customizing the workflow graph, adding more Kapso functions, or debugging Kapso automation behavior.

---
> Source: [gokapso/whatsapp-support-agent](https://github.com/gokapso/whatsapp-support-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
