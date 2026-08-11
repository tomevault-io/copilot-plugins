## aiip

> You have access to AIIP (Agent Identity and Integrity Protocol) tools via MCP.

# AGENTS.md

You have access to AIIP (Agent Identity and Integrity Protocol) tools via MCP.
Follow these instructions every session.

---

## On Every Session Start

Call `bootstrap_self` with your system prompt before doing anything else:

```
bootstrap_self({
  system_prompt: "<paste your system prompt here or describe your role>",
  requested_attributes: ["data:clearance_public"]
})
```

**`requested_attributes` is where your permissions are defined.**
The administrator of this project sets this list. Do not change it yourself.
Current attributes granted to this agent: `["data:clearance_public"]`

Store the returned `agent_code` and `delegation_token` — you will need them.

---

## Before Any Sensitive Operation

Call `verify_permission` before **every** tool call, every time.

**Do not cache or assume a previous result still applies. Policy can change between calls.**

```
verify_permission({
  agent_code: "<your agent_code from bootstrap>",
  agent_attributes: ["data:clearance_public"],
  tool_name: "<name of the tool you want to use>"
})
```

---

## How to Handle the Result

**If `allowed: true`** → proceed normally.

**If `allowed: false`** → do NOT proceed. Tell the user:
- What you tried to do
- What permission is missing (from the `reason` field)
- What they can do about it (from the `suggestion` field)

Example: *"I don't have permission to delete files. I need the `compute:admin` attribute. Ask your administrator to grant it."*

---

## Rules

1. Never skip `bootstrap_self` at session start.
2. Never skip `verify_permission` before sensitive operations.
3. Never reuse a previous `verify_permission` result — call it fresh every time.
4. Never proceed after a denial without explicit human instruction.
5. If AIIP tools are unavailable, say so and ask the user to check the server.

---
> Source: [MihaiCiprianChezan/AIIP](https://github.com/MihaiCiprianChezan/AIIP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
