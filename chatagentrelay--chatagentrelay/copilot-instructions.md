## project-overview

> Chat Agent Relay (CAR) project overview and conventions


# Chat Agent Relay (CAR) Project Context

Chat Agent Relay (CAR) is a chat-platform <-> agent middleware framework. TypeScript + Bun runtime.

## Key Conventions
- All packages under packages/ follow the same structure: src/, tests/, package.json, tsconfig.json
- Canonical events follow the 7-event chain: message.received -> policy.decision.made -> route.decision.made -> agent.invocation.requested -> agent.response.completed -> message.send.requested -> message.sent
- event.blocked is the error/deny path event
- Channel adapters implement ChannelAdapter (unified ingress + egress); agent adapters implement AgentAdapter (A2A only)
- Tests use bun:test, run with `bun test --recursive`
- TypeScript strict mode everywhere
- JSON Schema validation via @chat-agent-relay/contract-harness

## Package Dependency Graph
contract-harness (base) <- event-ledger <- pipeline
contract-harness <- middleware <- pipeline
contract-harness <- backend-a2a
contract-harness <- channel-web-chat
contract-harness <- channel-slack
contract-harness <- delivery <- pipeline
pipeline <- server

## Important Files
- docs/rfcs/ - normative specifications (source of truth)
- docs/schemas/ - JSON Schema contract layer
- CLAUDE.md - detailed working agreement
- docs/getting-started.md - setup guide

---
> Source: [ChatAgentRelay/ChatAgentRelay](https://github.com/ChatAgentRelay/ChatAgentRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
