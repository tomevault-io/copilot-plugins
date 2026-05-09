## superwisper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Custom modes and automations for [Superwhisper](https://superwhisper.com/), an AI-powered dictation app on macOS.

## Key Files

- **`modes/`** — 9 custom mode JSON files, ready to install
- **`install.sh`** — Copies modes to superwhisper and registers them in settings
- **`.claude/skills/superwhisper/SKILL.md`** — Claude Code skill for creating new custom modes
- **`.claude/skills/superwhisper/reference.md`** — Voice/language model tables, URL scheme reference
- **`.claude/skills/superwhisper/examples.md`** — Ready-to-use mode recipes and templates

## Superwhisper Mode JSON Format

Mode files go in `~/Documents/superwhisper/modes/`. They **must** use Apple-style JSON formatting (`"key" : value` with spaces before colons). Standard `json.dump` format will silently fail.

```json
{
  "activationApps" : [

  ],
  "activationSites" : [

  ],
  "adjustOutputVolume" : false,
  "contextFromActiveApplication" : false,
  "contextFromClipboard" : false,
  "contextFromSelection" : false,
  "contextTemplate" : "Use the copied text as context to complete this task.\n\nCopied text: ",
  "description" : "",
  "diarize" : false,
  "iconName" : "",
  "key" : "unique-key-here",
  "language" : "auto",
  "languageModelEnabled" : true,
  "languageModelID" : "sw-claude-4p5-sonnet",
  "literalPunctuation" : false,
  "name" : "Mode Name",
  "pauseMediaPlayback" : true,
  "prompt" : "",
  "promptExamples" : [

  ],
  "realtimeOutput" : false,
  "script" : "",
  "scriptEnabled" : false,
  "translateToEnglish" : false,
  "type" : "custom",
  "useSystemAudio" : false,
  "version" : 1,
  "voiceModelID" : "sw-ultra-cloud-v1-east"
}
```

Use `to_apple_json()` from the skill to generate this format. All `promptExamples` IDs must be UUID format.

## Registering New Modes

After creating a mode JSON file, you **must** add its `key` to the `modeKeys` array in `~/Documents/superwhisper/settings/settings.json`. Write settings.json in Apple-style JSON too. Then quit and reopen superwhisper.

## URL Schemes

```
superwhisper://mode?key=MODE_KEY   # Switch mode
superwhisper://record              # Start recording
superwhisper://settings            # Open settings
```

## File Paths

| Path | Purpose |
|------|---------|
| `~/Documents/superwhisper/modes/` | Installed mode JSON files |
| `~/Documents/superwhisper/settings/settings.json` | Mode registry (`modeKeys` array) |
| `~/Documents/superwhisper/recordings/` | Dictation history |

## Defaults

- **Language model**: `sw-claude-4p5-sonnet` (Claude 4.5 Sonnet)
- **Voice model**: `sw-ultra-cloud-v1-east` (Ultra Cloud)
- **Prompt style**: XML tags (`<role>`, `<critical>`, `<instructions>`, `<requirements>`, `<output-format>`) with anti-conversational overrides

---
> Source: [Haqbani/superwisper](https://github.com/Haqbani/superwisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
