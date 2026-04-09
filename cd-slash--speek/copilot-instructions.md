## speek

> **REQUIRED SEQUENCE** for CLI projects after any changes:

# CLAUDE.md - Speek Project Instructions

## CLI Build & Release Rule

**REQUIRED SEQUENCE** for CLI projects after any changes:

```bash
# 1. Build
npm run build

# 2. Version bump
npm version patch  # or minor/major as appropriate

# 3. Pack
npm pack

# 4. Install globally (test)
npm install -g ./package-name-VERSION.tgz

# 5. Test global functionality
speek --version
speek help

# 6. Commit release
git commit -m "release: vX.X.X"
```

**Auto-trigger**: Any code changes → Run full sequence
**Purpose**: Ensure CLI works globally before release
**Verification**: Global command must function correctly

## Project Context

- **Type**: CLI tool for text-to-speech
- **Command**: `speek`
- **Key Features**: TTS via Gemini API, configurable voices
- **Build**: TypeScript → JavaScript (dist/)
- **Global**: Must work as `speek` command system-wide

## Commands Available

- `speek [text]` - Convert text to speech
- `speek generate-script` - Convert app output to TTS script  
- `speek setup` - Configure speech settings
- `speek config` - Show configuration
- `speek help` - Show help menu

---
*Auto-generated CLI project rules*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cd-slash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cd-slash)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
