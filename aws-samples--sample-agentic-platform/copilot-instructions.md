## sample-agentic-platform

> Context for AI coding assistants (Claude Code, Cursor, Codex, Kiro, Copilot, etc.) editing files in `src/agentic_platform/agent/coding_agent/`. Pair this with the repo-root [`AGENTS.md`](../../../../AGENTS.md) for cross-cutting rules.

# AGENTS.md — Coding Agent

Context for AI coding assistants (Claude Code, Cursor, Codex, Kiro, Copilot, etc.) editing files in `src/agentic_platform/agent/coding_agent/`. Pair this with the repo-root [`AGENTS.md`](../../../../AGENTS.md) for cross-cutting rules.

> **The operator-facing setup/deploy guide is in [`README.md`](README.md)** — it has the end-to-end CDK deploy walkthrough, environment-variable reference, secret/key handling, teardown procedure, and troubleshooting. If a human ran into a problem standing the agent up, send them there. This file is for *contributors changing the code*.

## What this agent is

A **FastAPI shim around the `claude` CLI**. The Python code in this folder is *not* the coding intelligence — Claude Code is. The server's only job is:

1. accept a JSON payload over HTTP,
2. clone a target repo (optional),
3. shell out to `claude -p --dangerously-skip-permissions --output-format stream-json …`,
4. log the CLI's events to CloudWatch and capture the final `result` envelope.

The HTTP response is an immediate `202 Accepted` with a `task_id`; the actual `claude` run continues in the background inside the AgentCore microVM. When you change something in this folder, hold that mental model. The "brain" lives in the CLI subprocess; your job is to keep the harness around it small, predictable, and safe. If you find yourself reaching for the Python SDK or a multi-step state machine, push back and ask whether the CLI already does it.

## Where things live

### In this folder (`src/agentic_platform/agent/coding_agent/`)

| File | Purpose | Edit when… |
|------|---------|------------|
| `Dockerfile` | Builds the runtime image. Python 3.12 + Node 24 + Claude Code CLI + git + gh. | The runtime contract changes (Python version, CLI version, system deps). |
| `server.py` | FastAPI app, payload parsing, repo cloning, `claude` subprocess driver. | The invocation contract changes or the CLI flags change. |
| `requirements.txt` | Python deps (fastapi, uvicorn, boto3, pydantic). No SDK — we call the CLI directly. | Adding/upgrading a Python library. |
| `__init__.py` | Empty marker. | Don't. |
| `README.md` | **Operator** docs (deploy, env vars, request shape, troubleshooting). | Behavior visible to callers / operators changes. |
| `AGENTS.md` | This file. **Contributor** docs. | Conventions for code changes change. |

### Outside this folder (deployment + infra)

The deployment stack is **CDK**, not Terraform. Older revisions referenced `infrastructure/stacks/agentcore-runtime/coding_agent.tfvars`; that path no longer exists.

| Path | Purpose | Edit when… |
|------|---------|------------|
| [`/cdk/bin/coding-agent.ts`](../../../../cdk/bin/coding-agent.ts) | CDK app entrypoint. Registers cdk-nag and instantiates `CodingAgentStack`. | Adding env-level branching, registering more stacks. |
| [`/cdk/lib/stacks/coding-agent-stack.ts`](../../../../cdk/lib/stacks/coding-agent-stack.ts) | Stack glue. Composes `AgentCoreRuntime` + `ApiKeyFrontDoor` and exposes the CFN outputs. | Adding new stack-level outputs or props. |
| [`/cdk/lib/constructs/agentcore-runtime.ts`](../../../../cdk/lib/constructs/agentcore-runtime.ts) | ECR repo (imported), GitHub PAT secret, the AgentCore Runtime, log groups, Bedrock IAM. | The runtime needs new env vars / IAM / log destinations. |
| [`/cdk/lib/constructs/api-key-frontdoor.ts`](../../../../cdk/lib/constructs/api-key-frontdoor.ts) | API Gateway, the Lambda proxy that calls `InvokeAgentRuntime`, API key, usage plan. | The HTTP front door changes (new method, different auth, throttle limits). |
| [`/cdk/lambda/invoke/index.ts`](../../../../cdk/lambda/invoke/index.ts) | Lambda handler — bundled by `NodejsFunction`. Pass-through to AgentCore. | The Lambda needs to do more than pass-through (it shouldn't). |
| [`/deploy/build-container.sh`](../../../../deploy/build-container.sh) | Builds & pushes the multi-arch image to ECR (creates the repo if missing). | Build-time changes (tag strategy, scan settings). |

When you add a new env var to `server.py`, add it to **all three** places:

1. `os.environ.get(...)` in `server.py`
2. `environmentVariables: {...}` in [`/cdk/lib/constructs/agentcore-runtime.ts`](../../../../cdk/lib/constructs/agentcore-runtime.ts)
3. The "Environment variables" section of [`README.md`](README.md)

## Contracts that must not drift

These are the load-bearing contracts other systems depend on. Don't change them without coordinating.

1. **HTTP shape.** `POST /invocations` accepts an arbitrary JSON object. `GET /ping` returns `{"status": "healthy"}` with HTTP 200. Both live on port 8080. AgentCore's container contract requires both.
2. **Recognized payload fields.** `task_description`, `repo_url`, `max_budget_usd`, `cwd`. Everything else passes through as Jira-style context inside the prompt. Adding a new recognized field is fine; renaming an existing one is breaking and needs a corresponding change in the Jira integration.
3. **Response shape.** Today: `{task_id, status, cwd}` returned with `202 Accepted`. Renaming any of these breaks the API key front door and any in-flight Jira automation. New fields the CLI starts emitting flow into background-task logs automatically.
4. **PAT resolution order.** Env var `GITHUB_TOKEN` first, then Secrets Manager (`GITHUB_TOKEN_SECRET_ID`), then empty. The empty fallback must keep working — public repos should clone without any secret configured.
5. **Bedrock as the model provider.** `CLAUDE_CODE_USE_BEDROCK=1` is set at the env layer. Don't switch the agent to direct Anthropic API or LiteLLM without explicit sign-off — the security posture of the rest of the platform assumes prompts/tokens stay in AWS.
6. **`RemovalPolicy.RETAIN` on the ECR repo and the PAT secret.** Both contain state that's expensive to lose (built images / a hand-typed secret). Keeping them out of the destroy path means a `cdk destroy` won't surprise you. The README documents how to remove them manually when actually wanted.

## How to test changes

There are no committed unit tests for this agent yet — exercise it end-to-end:

```bash
# 1. Build
docker build \
  -f src/agentic_platform/agent/coding_agent/Dockerfile \
  -t coding-agent:local .

# 2. Run with AWS creds
eval "$(aws configure export-credentials --format env)"
docker run --rm -p 8080:8080 \
  -e AWS_REGION=us-east-1 \
  -e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -e AWS_SESSION_TOKEN \
  -e CLAUDE_CODE_USE_BEDROCK=1 \
  -e GITHUB_TOKEN="${GITHUB_TOKEN:-}" \
  coding-agent:local

# 3. Smoke
curl -s http://localhost:8080/ping
curl -s -X POST http://localhost:8080/invocations \
  -H 'Content-Type: application/json' \
  -d '{"task_description": "Create answer.txt containing 42."}'
```

If your change touches `repo_url` handling, also test against a real repo (use `octocat/Hello-World` or a private repo with a fine-grained PAT).

If you add unit tests, place them under `tests/agent/coding_agent/` at the repo root and run with `pytest`.

For a **deploy-time** verification (the change requires re-rolling the runtime), follow the [Updating the image](README.md#updating-the-image) section of the README — `build-container.sh` followed by `update-agent-runtime`. Don't run `cdk deploy` for image-only changes; the stack pins `:latest`, AgentCore caches the image at create time, and the rolling-restart command is what picks up the new bits.

## Common mistakes

- **Adding business logic to `server.py`.** This file should stay narrow: parse the payload, resolve secrets, clone the repo, exec the CLI. Anything more elaborate (custom prompt templates, multi-step orchestration, retry policy) belongs in a sibling module so the request handler stays readable.
- **Reaching for the Python SDK.** The whole point of this harness is that the CLI is enough. If you think you need `claude-agent-sdk`, first prove the CLI can't do it (`claude --help`, `--output-format stream-json`, `--input-format stream-json`, `--mcp-config`, etc.). Pulling the SDK back in adds a second protocol surface to maintain.
- **Logging the PAT.** `_resolve_github_pat()` returns a sensitive string. Don't add `logger.info(pat)` "just for debugging," and scrub it from any subprocess error output before re-raising — the existing code does this for `git clone` failures *and* for the `claude` stream drain (`.replace(pat, "<REDACTED>")`); mirror that pattern.
- **Hardcoding the model id.** The model is parameterized via `ANTHROPIC_MODEL` (env var) and `-c anthropicModel=…` (CDK context). Never inline a Bedrock inference profile id in `server.py` or the CDK constructs as a hard-coded string — operators pick the model at deploy time.
- **Removing the budget cap.** Claude Code with `--dangerously-skip-permissions` will happily run for hours if nothing stops it. Always pass `--max-budget-usd` (the default comes from `CODING_AGENT_MAX_BUDGET_USD`). Per-request override goes through the `max_budget_usd` payload field.
- **Running as root.** The Dockerfile creates `appuser` because Claude Code's CLI refuses `bypassPermissions` when invoked as root. Don't add `USER root` for "convenience."
- **Deploying without bumping the runtime.** `:latest` in ECR doesn't auto-roll the AgentCore runtime — the runtime caches the image at create time. Run `update-agent-runtime` (see [Updating the image](README.md#updating-the-image)) or you'll be testing yesterday's bits.
- **Putting the GitHub PAT in CDK context, env vars at synth time, or CFN parameters.** Any of those land it in the synthesized template and the CFN events stream. The PAT goes into Secrets Manager **after** the stack is up, via `aws secretsmanager put-secret-value`. The CDK creates the secret empty for exactly this reason.

## When to escalate

Open a discussion (issue / Slack / PR draft) before:

- Changing the `/invocations` request or response shape.
- Switching the model provider away from Bedrock.
- Adding a new long-lived credential (anything that survives between requests).
- Wiring this agent into AgentCore Gateway or Identity — the platform pattern for tool calls and identity is being defined elsewhere; check before adding a parallel implementation.
- Changing `RemovalPolicy.RETAIN` on the ECR repo or the PAT secret. Those are deliberate; "cleaner destroy" isn't a strong enough reason to drop them.

## Pointers

- Repo-root rules and tool routing: [`/AGENTS.md`](../../../../AGENTS.md)
- Operator-facing setup & deploy guide: [`README.md`](README.md)
- CDK app (the deployment surface): [`/cdk`](../../../../cdk/) and [`/cdk/README.md`](../../../../cdk/README.md)
- Other agents in the platform (different harnesses, different deploy stacks): [`../agentic_chat/`](../agentic_chat/), [`../jira_agent/`](../jira_agent/), [`../langgraph_chat/`](../langgraph_chat/)
- Reference implementation for a much fuller coding-agent system (Cedar policy, hooks, progress writing, PR creation): the sibling repo `sample-autonomous-cloud-coding-agents` — useful when this agent grows into PR-opening territory.
- AgentCore docs: <https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-long-run.html>
- Claude Code CLI: <https://docs.claude.com/en/docs/claude-code/cli-usage>

---
> Source: [aws-samples/sample-agentic-platform](https://github.com/aws-samples/sample-agentic-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
