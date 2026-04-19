## cnc-redalert-port

> This project contains coding standards and best practices for use

# Claude Code Standards

This project contains coding standards and best practices for use
with Claude Code. These files define preferences for code style,
architecture, workflow, and agent behavior.

## Standards Files

@parts/workflow.md
@parts/files.md
@parts/agent.md
@parts/code.md
@parts/style.md
@parts/architecture.md
@parts/error-handling.md
@parts/security.md
@parts/config.md
@parts/concurrency.md
@parts/testing.md
@parts/tooling.md
@parts/unix.md

## Priority Tiers

Standards use these priority levels:
- **MUST**: Violating causes real harm. No exceptions.
- **SHOULD**: Strong preference, but judgment applies.
- **MAY**: Good defaults, flexible per context.

## Key Behaviors

1. **Interrogative vs Imperative**: Default to discussion mode.
   Only generate artifacts after "make it so" or explicit command.

2. **Design First**: Understand and agree on design before
   generating code, config, docs, or shell commands.

3. **tmp/ Isolation**: Any file Claude creates on its own
   initiative goes in `tmp/`, not the project root.

4. **Surface Gaps**: If a task falls outside documented standards,
   say so and propose an approach.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jassoncasey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
