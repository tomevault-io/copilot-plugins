## prevent-duplicate-dev

> This rule prevents running `npm run dev` commands in terminal instances since we assume it's already running in another terminal.

 # Prevent Duplicate Dev Server Commands

This rule prevents running `npm run dev` commands in terminal instances since we assume it's already running in another terminal.

<rule>
name: prevent_duplicate_dev
description: Prevents running duplicate dev server commands
filters:
  # Match terminal command events
  - type: event
    pattern: "terminal_command"
  # Match npm run dev commands
  - type: content
    pattern: "npm run dev"

actions:
  - type: reject
    conditions:
      - pattern: "npm run dev"
        message: "Dev server is assumed to be already running. Please use the existing dev server instance instead of starting a new one."

  - type: suggest
    message: |
      The dev server is assumed to be already running in another terminal.
      
      Instead of starting a new dev server:
      1. Use the existing dev server instance
      2. If the dev server is not running, check your other terminal windows
      3. Only start a new dev server if you've confirmed no other instance is running

examples:
  - input: "npm run dev"
    output: "Command rejected - dev server already running"

metadata:
  priority: medium
  version: 1.0
</rule>

---
> Source: [alexawilcox1120/TestingCursor](https://github.com/alexawilcox1120/TestingCursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
