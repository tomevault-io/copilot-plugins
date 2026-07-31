## 08-troubleshooting

> Svelte 5 reminder


# 08-troubleshooting

## Troubleshooting - How to fix potential issues that may arise during development.

SVELTE 5 (WITH RUNES) HELP:
- In Svelte 5, we can't export and reassign state variables. 
Instead, we need to either:
  1. Export a function that returns the state value, or
  2. Only mutate properties of the state object without reassigning it.

JavaScript property descriptor rules, you cannot have both a getter/setter and a value/writable attribute in the same descriptor. 

---
> Source: [fivelity/usmp](https://github.com/fivelity/usmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
