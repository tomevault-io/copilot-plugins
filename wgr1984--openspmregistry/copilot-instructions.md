## openspmregistry

> name: concise_explanations

# Cursor Rules

<rule>
name: concise_explanations
description: Standards for keeping explanations concise

filters:
  - type: all_files
    pattern: "*"

actions:
  - type: suggest
    message: |
      When explaining code or changes:

      1. Keep explanations brief and focused:
         - One line per main point
         - No redundant information
         - Skip obvious details

      2. For code changes:
         - State what changed
         - State why (if not obvious)
         - No need to explain unchanged code

      3. For errors:
         - State the error
         - State the fix
         - No need for apologies

      4. For test results:
         - State pass/fail
         - If failed, state why
         - No need for detailed logs unless relevant

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wgr1984)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wgr1984)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
