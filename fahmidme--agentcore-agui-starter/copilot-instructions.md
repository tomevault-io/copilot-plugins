## agentcore-agui-starter

> Keep this repository a small, transparent reference implementation for streaming a Strands agent through Amazon Bedrock AgentCore and AG-UI into a Next.js CopilotKit interface.

# Repository guide for coding agents

## Mission

Keep this repository a small, transparent reference implementation for streaming a Strands agent through Amazon Bedrock AgentCore and AG-UI into a Next.js CopilotKit interface.

## Architecture boundaries

- `app/StreamingAssistant/` owns the Python agent, tools, Bedrock model setup, and AG-UI application.
- `agentcore/agentcore.json` is the source of truth for AgentCore resources. The public runtime name is `StreamingAssistant`.
- `web/app/api/copilotkit/[[...slug]]/route.ts` is the server-side bridge. AWS credentials must never be sent to the browser.
- `web/app/page.tsx` owns the minimal demonstration UI.
- The `add_numbers` tool is deliberately simple so AG-UI tool events remain easy to inspect.

## Important invariants

- Preserve streaming. Do not replace AG-UI/SSE with a buffered JSON response.
- Keep account-specific targets, deployment state, credentials, and environment files out of Git.
- Conversation history in this starter is supplied by the active AG-UI thread. Do not describe it as durable AgentCore Memory.
- Do not edit generated files in `agentcore/cdk/` unless the AgentCore CLI workflow explicitly requires it.
- Do not deploy AWS resources, push branches, or open pull requests without explicit user approval.
- Prefer environment variables for model or endpoint configuration; never hard-code secrets.

## Local commands

Agent:

```bash
cd app/StreamingAssistant
uv sync --locked
uv run python -m compileall main.py model
```

AgentCore development server, from the repository root:

```bash
agentcore dev --runtime StreamingAssistant --skip-deploy --logs --no-traces --port 8081
```

Web:

```bash
cd web
npm ci
npm run lint
npm run typecheck
npm run build
```

## Editing guidance

- Python: target 3.12+, type public functions, and keep tools deterministic and documented.
- TypeScript: preserve strict typing and keep AWS/network integration in server-only routes.
- Next.js 16 can contain breaking changes. Consult the installed documentation in `web/node_modules/next/dist/docs/` before relying on older examples.
- Update the root README whenever commands, ports, environment variables, or architecture change.

## Verification before handoff

1. Run Python compilation.
2. Run web lint, typecheck, and production build.
3. Start both local servers and exercise a normal prompt and an `add_numbers` prompt.
4. Confirm streamed AG-UI events and current-thread history.
5. Run a secret scan and inspect `git status --ignored` before staging.

---
> Source: [fahmidme/agentcore-agui-starter](https://github.com/fahmidme/agentcore-agui-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
