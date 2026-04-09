## mq-bridge-mcp

> This document describes the capabilities of the `mq-bridge-mcp` server for AI assistants.

# mq-bridge-mcp Server Context

This document describes the capabilities of the `mq-bridge-mcp` server for AI assistants.

## Purpose
This server bridges the Model Context Protocol (MCP) to the `mq-bridge` library, allowing you (the AI) to act as a **Publisher** (sending messages) or a **Consumer** (receiving messages) to various infrastructure components like Kafka, SQS, databases, and files.

## Tooling Structure

### Publishers (Tools)
Defined in the `publishers` section of the configuration.
- **Action**: You can send data to these endpoints.
- **Naming**: Tools are exposed as `publish_to_<name>`.
- **Usage**: Use these tools when the user asks to "send", "post", "write", or "notify".

### Consumers (Resources & Tools)
Defined in the `consumers` section of the configuration.
- **Action**: You can read data from these endpoints.
- **Naming**: 
    - Resources are exposed as `mq://<name>` (for reading metadata/lists).
    - Tools are exposed as `consume_from_<name>` (for actively reading message batches).
- **Usage**: Use these tools when the user asks to "read", "check", "monitor", or "analyze logs".

## Configuration Samples & Use Cases

Here are example configurations (`mcp-config.yml`) demonstrating good use cases.

### 1. Incident Reporting (HTTP Webhook)
Enable the AI to post alerts to a communication platform like Slack or Teams.

```yaml
mcp:
  enabled: true
  transport: "stdio"

publishers:
  slack_incident:
    description: "Post a high-priority incident to the Slack #ops channel."
    http:
      url: "https://hooks.slack.com/services/XXX/YYY/ZZZ"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcomq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcomq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
