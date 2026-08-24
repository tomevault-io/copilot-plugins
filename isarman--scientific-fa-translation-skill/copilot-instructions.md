## scientific-fa-translation-skill

> This repository is a single Cursor skill. `SKILL.md` is at the clone

# scientific-fa-translation-skill

This repository is a single Cursor skill. `SKILL.md` is at the clone
root so it is discoverable when cloned into `/home/$USER/.cursor/skills`:

```text
/home/$USER/.cursor/skills/scientific-fa-translation-skill/SKILL.md
```

Cursor only looks one level deep. Do not nest this clone under another
folder inside `/home/$USER/.cursor/skills`, and do not clone it *as*
`/home/$USER/.cursor/skills` itself.

When the user wants to translate a paper, article, book, or technical
document into scientific Persian — or asks for RTL, چاپ, or a PDF, or
asks whether a finished Persian translation follows the rules — read and
follow `SKILL.md` before producing output. Do this even if
`/scientific-fa-translation-skill` is missing from the slash menu.

Do not use that skill for coding, commits, UI copy, or casual chat.

## Working on the skill itself

- The terminology policy has one owner: `references/terminology.md`.
  Lists live in `glossary.md` and `glossary-domains.md`; forbidden
  Persian calques live in `references/term-pairs.tsv`. Do not restate
  the policy in a second file.
- A new rule that a machine could check belongs in `scripts/check-fa.py`
  with a fixture in `tests/fixtures/`, not only in prose. Run
  `bash tests/run.sh` after touching the checker or a fixture. Pass
  `--level journal` when the fixture is a paper, not a sysadmin guide.
- Keep `SKILL.md` short. It is loaded in full whenever the skill
  triggers; detail belongs in `references/`.

---
> Source: [isArman/scientific-fa-translation-skill](https://github.com/isArman/scientific-fa-translation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
