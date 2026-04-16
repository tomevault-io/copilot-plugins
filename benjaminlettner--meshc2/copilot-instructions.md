## meshc2

> Deliver reliable, incremental implementation for MeshC2 with minimal regressions and clear verification at each step.


# AI IDE Ruleset — Prompt Engineering & Implementation Discipline

## Mission
Deliver reliable, incremental implementation for MeshC2 with minimal regressions and clear verification at each step.

## Prompting Rules
1. **State the goal** and the concrete deliverable (file changes, tests, docs).
2. **Specify constraints** (payload limits, rate limits, encryption assumptions).
3. **Define success criteria** (what should work, how to verify).
4. **Request small, reviewable steps** (avoid large refactors).
5. **Require a brief plan** before any code changes.

## Implementation Workflow
1. **Discovery**: locate relevant files, read context, confirm assumptions.
2. **Plan**: break work into milestones with explicit outputs.
3. **Implement**: make minimal changes aligned with plan.
4. **Verify**: run tests or explain how to validate manually.
5. **Review**: summarize changes, risks, and next steps.

## Autonomy
- Implement steps directly when user input is not required.
- Ask only when blocking decisions are needed (secrets, destructive actions, unclear requirements).

## Coding Guidelines
- Prefer minimal upstream fixes over downstream workarounds.
- Keep changes small; avoid touching unrelated files.
- Use consistent formatting and follow existing patterns.
- Add tests for protocol logic and state machines.
- Document any new config or interface changes.

## Reliability Rules (LoRa Constraints)
- Keep payloads <= 200 bytes.
- Respect rate limits; avoid chatter.
- Use acknowledgements and retransmissions conservatively.
- Favor predictability over throughput.

## Safety & Security Rules
- Do not assume public exposure; default to localhost bind.
- Enforce command allowlists and execution timeouts.
- Treat all remote inputs as untrusted.
- Never embed secrets in code or docs.

## Validation Checklist (per change)
- [ ] Transport framing still fits size constraints.
- [ ] Session state machine handles loss/out-of-order.
- [ ] Web UI still connects and reconnects.
- [ ] Command execution respects allowlist/timeouts.

## Docs Quick Reference
- **Architecture**: component roles and data flow.
- **Protocol**: frame format, reliability, flow control.
- **Implementation Plan**: milestones and deliverables.
- **Roadmap**: phased timeline and acceptance criteria.
- **Config**: host/client configuration schema.
- **Deployment**: setup steps for macOS and Pi.
- **Security**: threat model and mitigations.
- **Development**: repo structure, workflow, CI.

## Communication Style
- Be concise and direct.
- Provide file references with line ranges.
- Ask clarifying questions when requirements are ambiguous.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenjaminLettner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
