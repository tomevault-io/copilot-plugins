## ai-flavor-forensic

> This repository is a **portable prose-forensics protocol** for Chinese and Japanese, written as plain markdown. There is no code to run and nothing to install beyond reading files.

# ai-flavor-forensic

This repository is a **portable prose-forensics protocol** for Chinese and Japanese, written as plain markdown. There is no code to run and nothing to install beyond reading files.

## For an agent picking this up

Read [SKILL.md](SKILL.md) in full — it is the protocol. Follow it exactly; the gates are the point, and skipping the gut-check gate is the documented failure mode.

Consult [references/taxonomy-deep.md](references/taxonomy-deep.md) only when a specific sentence is in doubt. Do not scan the whole table by default — it is an armory, not a checklist.

## Ground rules baked into the protocol

- A taxonomy match is a **candidate**, never a verdict. Every candidate must clear the gut-check gate before it can be reported.
- Default toward "clean." Correctly declining to flag beats flagging a lot.
- **Do not rewrite.** Report findings; leave the fixing to the author.
- Output must state the overall verdict first, then the quantitative readout, then the findings table, then the matched-but-kept table with reasons.
- This is a style ruler, not an authorship detector. Never report a conclusion about who wrote the text.

## Human-facing docs

[English](README.md) · [日本語](README.ja.md) · [中文](README.zh.md)

---
> Source: [vincent-wen789/ai-flavor-forensic](https://github.com/vincent-wen789/ai-flavor-forensic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
