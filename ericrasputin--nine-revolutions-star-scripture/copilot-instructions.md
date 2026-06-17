## nine-revolutions-star-scripture

> This repo contains the raw Chinese chapters (`raws/N.md`), English translations (`translations/N.md`), and the assets that drive the translation pipeline (`assets/`).

# Project: Nine Revolutions Star Scripture — English Fan Translation

This repo contains the raw Chinese chapters (`raws/N.md`), English translations (`translations/N.md`), and the assets that drive the translation pipeline (`assets/`).

## Workflow

Claude Code is the translator and editor for this project. Two documents define the workflow end-to-end:

- [`TRANSLATING.md`](./TRANSLATING.md) — raw Chinese → English draft.
- [`EDITING.md`](./EDITING.md) — draft → publishable chapter.

When the user provides raw Chinese (pastes it in chat, or points at `raws/N.md`, or simply says "translate chapter N"), follow `TRANSLATING.md` end-to-end without asking for confirmation at each step:

1. **Save the raw.** Before doing anything else, write the user-provided Chinese text to `raws/N.md` (where N is the chapter number). If the user pasted the chapter directly into chat without saving it first, persist it to disk yourself — do not translate from in-memory text only. The raw file is the source of truth: every re-translation, fidelity check (`EDITING.md` Pass 1), and future glossary diff depends on it being on disk. Skip this step only if the file already exists and matches what the user provided.
2. **Translate.** Read `assets/translation_prompt.md` and use it as the operating instructions. Read `assets/glossary.json` and apply established terminology. Produce the English draft.
3. **Save the draft.** Write to `translations/N.md` with the navigation header (`<!-- NAV START --> ... <!-- NAV END -->` block linking N-1, TOC, N+1) and the chapter heading (`# Chapter N: Title`) per Step 4 of `TRANSLATING.md`. Do not leave the translation only in chat — it must be on disk so the editing passes can act on it and so it survives the session.
4. **Edit.** Run the three editing passes from `EDITING.md` (fidelity check, language cleanup, voice and rhythm) on the draft. Apply the find-and-fix checklist at the end of `EDITING.md` before declaring the chapter done.
5. **Update the glossary.** Per Step 6 of `TRANSLATING.md`, using the schema in `assets/glossary_prompt.txt`. Append new entries to `assets/glossary.json`.
6. **Commit and push.** Per Step 7 of `TRANSLATING.md`. The commit must include `raws/N.md`, `translations/N.md`, and `assets/glossary.json` so all three artifacts land on GitHub together. Direct pushes to main are pre-authorized for this repo (see `.claude/settings.local.json`) — no need to ask.

Do not skip the editing passes. The raw AI draft is not the final output; nothing lands on main until it's been through `EDITING.md`.

Do not skip the save steps (1 and 3). Both the raw Chinese and the final translation must be on disk under `raws/` and `translations/` respectively before commit. A chapter that exists only in chat is a chapter that's about to be lost.

Do not paste the contents of `assets/translation_prompt.md`, `assets/glossary.json`, or `assets/glossary_prompt.txt` into chat — read them from disk on demand.

## How to present editing-pass results to the user

When you finish the three editing passes on a chapter and are about to apply changes, present the proposed edits to the user as a markdown table with these columns:

| # | Line | Issue | Fix | Reasoning |
|---|---|---|---|---|

- **#** — sequential number (1, 2, 3…) for ease of reference.
- **Line** — the line number in the translation file (e.g., `L29`).
- **Issue** — the specific problem in one short clause (e.g., "merged dialogue + tag," "intensifier inflation," "first-person internal thought used third-person").
- **Fix** — the exact change you will make. Quote the new wording when feasible.
- **Reasoning** — *why* this fix over alternatives, in 1-3 sentences. Cite the source phrase (with the Chinese in parentheses if relevant), the editing-pass principle from `EDITING.md`, and any judgment call you made (e.g., "kept colloquial register because the Great Sage is a Sun Wukong-coded character"). The user reads this column to verify your taste, not just your accuracy.

The Reasoning column is mandatory for every chapter's diff table from chapter 3 onwards. Without it, the user cannot tell whether you're applying the right principle or just pattern-matching. If a fix is genuinely trivial (e.g., a typo correction), say so explicitly: "Reasoning: typo, no judgment call."

After the table, separately list:
- Any new glossary entries introduced in the chapter.
- Any new style decisions you're proposing to add to `EDITING.md`.
- Anything you noticed but are NOT fixing, with a one-line explanation.

Apply the edits only after presenting the table.

## Project conventions

- **Author name:** "My Dog-Skin Plaster (我的狗皮膏药)" — use this exact rendering wherever the author is named (README, LICENSE, anywhere). Do not pinyin-romanize the name as "Wode Goupi Gaoyao."
- **Style sheet:** lives in `EDITING.md` under "Style Decisions." When you make a new judgment call that future chapters will need to follow, add a line there rather than re-deciding each time.
- **Glossary discipline:** all character, sect, place, technique, item, and rank names must resolve to a `glossary.json` entry by the time the chapter is committed. New names introduced in a chapter get appended in Step 4 above.
- **Structural fidelity:** keep the same paragraph count and sentence count as the raw Chinese. This is enforced in `EDITING.md` Pass 1 and is non-negotiable — downstream tooling assumes it.

---
> Source: [EricRasputin/Nine-Revolutions-Star-Scripture](https://github.com/EricRasputin/Nine-Revolutions-Star-Scripture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
