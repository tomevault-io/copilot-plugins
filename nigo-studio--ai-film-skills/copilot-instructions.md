## ai-film-skills

> Two production skills from the pipeline behind CROW LINE, an AI-generated action film. This folder is a ready-to-use workspace: the skills live in `.claude/skills/` and trigger automatically once this folder is opened in Claude Code.

# AI Film Skills — Starter Kit

Two production skills from the pipeline behind CROW LINE, an AI-generated action film. This folder is a ready-to-use workspace: the skills live in `.claude/skills/` and trigger automatically once this folder is opened in Claude Code.

## The two skills

| Skill | Open it when |
|---|---|
| `video-prompt-director` | The user gives a scene direction, action choreography, or shot idea and wants a full Seedance 2.5 reference-to-video prompt built from it. |
| `character-consistency` | The user wants one character to stay identical across generated shots — expanding a character brief, building the ultra-detailed close-up, or assembling the three-panel character sheet. |

Always read the matching SKILL.md before writing, and follow its steps and self-checks exactly. The two skills compose: build the character sheet first with `character-consistency`, then hand that sheet to `video-prompt-director` as one of the reference images.

## Ground rules

- Generation prompts are always written in English, whatever language the user speaks. Talk to the user in their language.
- These skills write prompts and nothing else. The user runs the generation on their own platform and picks the takes. Never claim to have generated media, and never call a generation API on your own initiative.
- The direction belongs to the user: who appears, where, what happens, which moment matters most. Never override it. The prompt craft belongs to the skill.

## Iterating on a take

When the user comes back with a generated take and points at what looks wrong or different from what they pictured:

1. Find the prompt sentence that caused it — or the sentence that is missing.
2. Fix that part only. Keep the rest of the prompt and the template structure intact.
3. To remove something that appeared in frame, delete the sentence that put it there. Adding "no X" injects the token X and tends to backfire.
4. Deliver the complete revised prompt, ready to paste — not a diff, not a commentary.

## Layout

- `.claude/skills/video-prompt-director/` — scene direction in, submit-ready Seedance 2.5 prompt out
- `.claude/skills/character-consistency/` — character brief in, three-panel identity sheet out
- `README.md` — install and usage notes for humans

---
> Source: [nigo-studio/ai-film-skills](https://github.com/nigo-studio/ai-film-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
