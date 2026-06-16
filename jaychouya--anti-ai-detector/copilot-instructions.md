## anti-ai-detector

> Use when users ask to reduce AI traces, optimize Chinese/English academic writing style, or run anti-template checks for paper paragraphs.


# anti-ai-detector Entry

## Overview

This repository follows a root-entry pattern similar to `colleague-skill`: keep one top-level `SKILL.md` as discoverable entry, while implementation files live under `skill/skills/anti-ai-detector/`.

## Skill Location

- Main skill file: `skill/skills/anti-ai-detector/SKILL.md`
- Chinese phrase profile: `skill/skills/anti-ai-detector/chinese-ai-trace-blacklist.md`
- Static checker: `skill/skills/anti-ai-detector/scripts/check_ai_traces.py`

## How to Use

Invoke by skill name and provide your paragraph plus locked terminology.
For strict Chinese checks, run:

```bash
python skill/skills/anti-ai-detector/scripts/check_ai_traces.py path/to/chinese_draft.txt --zh
```

---
> Source: [jaychouya/anti-ai-detector](https://github.com/jaychouya/anti-ai-detector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
