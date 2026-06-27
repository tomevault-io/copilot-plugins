## entra-agentid-samples

> This repository contains Microsoft Entra Agent ID sample agents that demonstrate the sidecar identity pattern — the Microsoft Entra SDK for Agent ID runs as a companion container and handles all token acquisition for your agent code.

# GitHub Copilot instructions for entra-agentid-samples

This repository contains Microsoft Entra Agent ID sample agents that demonstrate the sidecar identity pattern — the Microsoft Entra SDK for Agent ID runs as a companion container and handles all token acquisition for your agent code.

## Repository layout

- `sidecar/dev/` — local-LLM sample (Ollama + LangChain + Flask)
- `sidecar/aws/` — AWS Bedrock sample (Claude model + Azure→AWS OIDC federation)
- `sidecar/weather-api/` — shared downstream API that validates agent tokens
- `deploy/azure/container-apps/` — Azure Container Apps deployment tutorials
- `scripts/` — PowerShell bootstrap for Blueprint + Agent Identity + Client SPA

## Working in this repository

- Always confirm the target Azure account, tenant ID, and subscription ID with the user before running any deployment, provisioning, or cleanup command.
- Never commit `.env`, `deploy-vars.sh`, tenant GUIDs, app/client GUIDs, or any secret values.
- Prefer editing existing files over creating new ones.

## Skills

Extended workflows are packaged as skills under `.claude/skills/<name>/SKILL.md`. When the user's request matches a skill's description, load the relevant `SKILL.md` in full before taking action:

- `entra-agent-id-setup` — provision Blueprint + Agent Identity + Client SPA in Microsoft Entra ID
- `deploy-agent-aca-dev` — deploy the local-LLM sample to Azure Container Apps
- `deploy-agent-aca-aws` — deploy the AWS Bedrock sample to Azure Container Apps

The `CLAUDE.md` at the repository root is the primary entry point for agent-based tools and mirrors this guidance.

---
> Source: [microsoft/entra-agentid-samples](https://github.com/microsoft/entra-agentid-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
