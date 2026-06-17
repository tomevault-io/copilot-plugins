## arcade-skills

> This repository contains skill plugins under `plugins/`. Each subdirectory in `plugins/` is an independent plugin (e.g., `plugins/arcade-build`).

# Repository Instructions

This repository contains skill plugins under `plugins/`. Each subdirectory in `plugins/` is an independent plugin (e.g., `plugins/arcade-build`).

## Structure

```text
plugins/
  <plugin>/
    plugin.json
    skills/
      <skill-name>/
        SKILL.md
        scripts/
        references/
    agents/
      <agent-name>.agent.md
tests/
  <plugin>/
    <skill-name>/
      eval.yaml
      <fixture files>
```

## Validation

Pull requests are validated automatically:

- **CODEOWNERS**: every skill and test folder must have designated owners
- **Structure**: plugin.json, SKILL.md frontmatter, eval.yaml schema, and marketplace consistency are checked

---
> Source: [dotnet/arcade-skills](https://github.com/dotnet/arcade-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
