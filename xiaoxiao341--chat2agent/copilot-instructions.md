## chat2agent

> - This repository exists to let ChatGPT's web conversation mode reach the local machine through a public MCP/OAuth bridge and perform coding work as close as practical to a Codex Agent.

# Project Scope

- This repository exists to let ChatGPT's web conversation mode reach the local machine through a public MCP/OAuth bridge and perform coding work as close as practical to a Codex Agent.
- Do not modify or disrupt the local Codex Agent, including its installed capabilities, global configuration, or normal local workflow.
- The local Codex Agent is not a product client or part of the runtime topology. However, DevSpace may discover and expose selected reusable Codex resources, such as applicable `AGENTS.md` instructions, Skills, and compatible Hook behavior, to ChatGPT web conversations without changing the originals or the local Codex runtime.
- Reuse Codex resources through read-only discovery or an explicit compatibility layer. Keep unsupported, secret-bearing, machine-private, or local-runtime-only resources out of the web-facing capability surface.
- Provider-backed Subagents are optional integrations, not part of the default web-chat capability path. They may consume the selected local provider account or API quota and must never be recommended or launched implicitly.
- Product recommendations and implementation should focus on the web ChatGPT connector experience, remote coding capability, reliability, observability, and security.
- Full `C:\` and `D:\` access is an explicit requirement for the owner's deployment. Open-source defaults and examples may use narrower roots, but must not redefine the owner's deployment requirement.

---
> Source: [xiaoxiao341/Chat2Agent](https://github.com/xiaoxiao341/Chat2Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
