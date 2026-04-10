## plugin-argocd

> Enables Kestra workflows to interact with ArgoCD, allowing orchestration of ArgoCD-based operations as part of data pipelines and automation workflows.

# Kestra ArgoCD Plugin

## What

description = 'ArgoCD Plugin for Kestra Exposes 2 plugin components (tasks, triggers, and/or conditions).

## Why

Enables Kestra workflows to interact with ArgoCD, allowing orchestration of ArgoCD-based operations as part of data pipelines and automation workflows.

## How

### Architecture

Single-module plugin. Source packages under `io.kestra.plugin`:

- `argocd`

Infrastructure dependencies (Docker Compose services):

- `argocd`

### Key Plugin Classes

- `io.kestra.plugin.argocd.apps.Status`
- `io.kestra.plugin.argocd.apps.Sync`

### Project Structure

```
plugin-argocd/
├── src/main/java/io/kestra/plugin/argocd/apps/
├── src/test/java/io/kestra/plugin/argocd/apps/
├── build.gradle
└── README.md
```

### Important Commands

```bash
# Build the plugin
./gradlew shadowJar

# Run tests
./gradlew test

# Build without tests
./gradlew shadowJar -x test
```

### Configuration

All tasks and triggers accept standard Kestra plugin properties. Credentials should use
`{{ secret('SECRET_NAME') }}` — never hardcode real values.

## Agents

**IMPORTANT:** This is a Kestra plugin repository (prefixed by `plugin-`, `storage-`, or `secret-`). You **MUST** delegate all coding tasks to the `kestra-plugin-developer` agent. Do NOT implement code changes directly — always use this agent.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kestra-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kestra-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
