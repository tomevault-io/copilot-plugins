## waba-agent

> Operate and validate the waba-agent CLI and gateway stack. Use when asked to continue builds, run production-readiness checks, wire onboarding, verify safety gating, troubleshoot chat/gateway/resale flows, or configure local-first AI with Ollama fallback.


# WABA Agent Operator

## Use this skill to
- Run regression and smoke checks for `waba-agent`.
- Diagnose CLI command mismatches and runtime errors.
- Patch onboarding for Meta + AI + staging verification.
- Validate safety rules: no shell/eval action flow; approval for high-risk actions.
- Configure or verify local AI defaults with Ollama.

## Standard workflow
1. Confirm repo state before edits.
`git status --porcelain=v1 -b`
2. Run tests using `npm.cmd` on PowerShell systems.
`npm.cmd run test:ai`
`npm.cmd run test:chat`
`npm.cmd run test:gateway`
`npm.cmd run test:resale`
`npm.cmd test`
3. Run command-surface smoke checks.
`node bin/waba.js --no-banner --help`
`node bin/waba.js --no-banner chat --help`
`node bin/waba.js --no-banner gateway --help`
`node bin/waba.js --no-banner resale --help`
4. Verify gateway root headers to avoid stale UI.
Expect:
- `Cache-Control: no-store, no-cache, must-revalidate, proxy-revalidate`
- `Pragma: no-cache`
- `Expires: 0`
5. Verify high-risk gating behavior.
- `allowHighRisk=false` must block `message.send_text`, `template.send`, and schedule send tools.
- `allowHighRisk=true` may proceed but must still fail safely when auth/config is missing.
6. For multi-client checks, prefer explicit `--client` where supported.

## Onboarding requirements
Capture in wizard whenever possible:
- Meta auth: token, phone-id, business-id
- AI setup: provider, key, model, base URL
- Staging verification profile: test recipient, template name/language/params

Persist:
- Global AI config in `~/.waba/config.json`
- Client verification profile in `~/.waba/context/<client>/client.json` under `verification.staging`

## Local AI defaults
If hosted keys are absent, runtime should use local Ollama:
- Base URL: `http://127.0.0.1:11434/v1`
- Primary model: `deepseek-coder-v2:16b`
- Fallback model: `qwen2.5:7b`

Quick local prep:
`ollama pull deepseek-coder-v2:16b`
`ollama pull qwen2.5:7b`
`ollama serve`

## Non-negotiable safety rules
- Keep approval gating for high-risk actions.
- Do not introduce `eval`, shell execution, or dynamic code execution in action flow.
- Do not remove existing user-facing commands unless explicitly requested.

## Output contract for takeover and validation tasks
Return:
1. Exact issues found
2. Code changes made
3. Commands run
4. Demo checklist with pass/fail and explicit blockers

---
> Source: [vishalgojha/waba-agent](https://github.com/vishalgojha/waba-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
