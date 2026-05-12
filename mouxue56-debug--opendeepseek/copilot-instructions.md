## opendeepseek

> This file is the project-level operating guide for Codex and other coding agents working on OpenDeepSeek.

# AGENTS.md

This file is the project-level operating guide for Codex and other coding agents working on OpenDeepSeek.

## Project Identity

OpenDeepSeek must not degrade into "Open WebUI plus a DeepSeek chat box".

The product goal is:

> Open WebUI as the friendly user surface, Hermes Agent as the real computer/file/memory/cron/tool execution layer, and DeepSeek V4 Flash as the cheap fast reasoning engine.

Creator Release framing:

> OpenDeepSeek is a product shell and stable glue layer around Open WebUI, Hermes, and DeepSeek/custom OpenAI-compatible APIs. Keep code changes small and experience polish high.

The next product line is:

> OpenDeepSeek CN: Chinese-first, China-network-ready, one-click installable, real Agent, polished entry experience.

## Iron Rules

1. Ordinary Q&A must stay fast and cheap by default. It should route to DeepSeek V4 Flash lightweight path.
2. Real tasks must route to Hermes Agent: files, `/host`, desktop folders, websites, PPT/reports, reminders, memory, images/OCR, terminal, tools, automation, and long-running jobs.
3. Open WebUI native Memory, Knowledge, and Tools must be preserved. The Bridge should not steal native OpenWebUI tool loops unless the request is truly an Agent task.
4. Uploaded images must not be forwarded as OpenAI `image_url` payloads to the DeepSeek text API. The Bridge must save/OCR/summarize them first.
5. Do not reduce `HERMES_AGENT_MAX_TOKENS`; website/PPT/report tasks can be truncated if this budget is lowered.
6. Do not expose Hermes or the Bridge to the public internet by default.
7. If `BIND_HOST=0.0.0.0` and `WEBUI_AUTH=false`, treat it as unsafe and block or loudly fail unless the user explicitly opts into danger.
8. Do not commit real API keys, tokens, cloud AK/SK, registry credentials, cookies, or `.env`.
9. Any claim that a file was generated must be backed by an actual file existence/size/content check.
10. Every routing change must run:

```bash
python3 scripts/benchmark_routing.py
./setup.sh verify
bash scripts/smoke-test.sh
```

If the local stack is intentionally stopped, explain which checks were skipped and why.
11. Do not fork Open WebUI or Hermes, and do not build a replacement chat UI or Agent runtime.
12. Open WebUI should keep connecting only to Smart Bridge; Portal/setup writes Provider config into `.env`, and Bridge/Hermes consume it.
13. Default Provider is DeepSeek. Advanced users may choose `custom` OpenAI-compatible APIs such as OpenRouter, local Ollama/LM Studio/vLLM, LiteLLM, or domestic compatible platforms.
14. Do not enable Open WebUI passthrough by default.

## China Ready Rules

1. China installation must not depend on GitHub raw as a single point of failure.
2. China users need Gitee/GitCode/OSS/COS source fallback, domestic image registry support, and offline bundle fallback.
3. CN scripts must never contain real cloud credentials.
4. CN Docker Compose should not rely on real, unpublished tags without documenting that they are placeholders.
5. CN docs must answer practical user questions: how to install, where files go, what folders Agent can access, how to stop, how to uninstall, and how to stay safe.

## UI Product Rules

1. Do not deeply rebrand or hard-fork Open WebUI UI. Build OpenDeepSeek Portal/Lite around it instead.
2. Product-facing modes should be understandable:
   - `opendeepseek-auto`
   - `opendeepseek-fast`
   - `opendeepseek-agent`
   - `opendeepseek-deepwork`
3. Generated artifacts should be shown as product cards with preview, local path, container path, and file list.
4. Chinese onboarding should explain permissions before granting Agent access to folders.

## Goal Workflow

For multi-step work, follow `docs/GOALS/OPENDEEPSEEK-CN.md` and `docs/GOALS/IMPLEMENT.md`.

Each milestone must:

1. Stay within its declared scope.
2. Avoid unrelated refactors.
3. Run the milestone validation commands.
4. Update `docs/GOALS/STATUS.md` with:
   - changed files
   - validation commands and results
   - risks
   - next recommended goal

Do not push, merge, tag, publish images, or upload release bundles unless the user explicitly asks for that exact operation.

---
> Source: [mouxue56-debug/opendeepseek](https://github.com/mouxue56-debug/opendeepseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
