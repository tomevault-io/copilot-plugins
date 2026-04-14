## reflow-oven

> description: "Enforce hardware interface management and safety guidelines."

---
triggers:
  - file_change
  - hardware_interface

rule_definition:
  description: "Enforce hardware interface management and safety guidelines."

  rules:
    - All hardware interfaces must be initialized with proper error checking.
    - GPIO pins must be documented with their purpose and configuration.
    - Hardware interfaces must be protected against concurrent access.
    - All hardware operations must include timeout mechanisms.
    - Hardware interfaces must be properly closed/released when not in use.
    - Hardware errors must be logged and handled gracefully.
    - Hardware interfaces must be tested for proper operation at startup.
    - Hardware configuration must be validated against physical constraints.
    - Hardware interfaces must implement proper error recovery procedures.
    - Hardware access must be abstracted through service classes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dcronje)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dcronje)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
