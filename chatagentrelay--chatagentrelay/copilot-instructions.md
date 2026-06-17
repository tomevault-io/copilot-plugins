## testing

> Testing conventions for Chat Agent Relay (CAR)


# Testing Conventions

- Use bun:test (describe, it, expect, beforeAll, afterAll)
- Test files go in tests/ directory alongside src/
- New adapters MUST pass conformance tests from @chat-agent-relay/adapter-conformance
- All canonical events must validate against JSON Schema via ContractHarnessValidators
- Mock external services with Bun.serve() on port 0 for random port assignment
- Never use real API keys in tests
- Test both success and error paths

---
> Source: [ChatAgentRelay/ChatAgentRelay](https://github.com/ChatAgentRelay/ChatAgentRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
