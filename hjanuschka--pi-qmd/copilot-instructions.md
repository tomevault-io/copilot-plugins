## pi-qmd

> This is an npm package for pi coding agent - QMD extension.

# Agent Configuration

This is an npm package for pi coding agent - QMD extension.

## Package Structure

- `package.json` contains the `"pi"` field that declares extensions
- Extensions are in `./extensions/*.ts`
- Skills are in `./skills/*/`

## Installation

```bash
npm install -g pi-qmd
```

Then add to `~/.pi/agent/settings.json`:

```json
{
  "extensions": ["pi-qmd"]
}
```

## Skills

### QMD Knowledge

Knowledge management skill for searching and retrieving documents from your QMD-indexed knowledge base.

**Slug:** `qmd-knowledge`

---
> Source: [hjanuschka/pi-qmd](https://github.com/hjanuschka/pi-qmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
