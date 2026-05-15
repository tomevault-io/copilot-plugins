## case-files-from-rpg-notes

> >-


# Case files from RPG notes (`refs/stories`)

**Invocation:** Globs cover the usual touchpoints; you can still **@mention this rule** or select it in Cursor Rules when you want it explicitly.

## Source material

- Session notes live under `refs/stories/` (e.g. per-adventure folders). Treat them as **raw field notes**, not publish-ready copy.
- **Omit from the website:** XP, leveling, dice rolls, mechanics, equipment purchases, and any other **player/character advancement** content. Keep **in-world investigation narrative**: what happened, what was learned, locations, NPCs, threats, evidence, and Bureau-facing outcomes.

## Investigator identity (aliases only)

- Read `refs/characters/investigator_character_mapping.md` to map players to **Investigator numbers** (07, 13, 23, 42, 51, 66, 89).
- In **all published HTML** (case files, story cards, tags, evidence blurbs): refer to people as **`Investigator NN`** (e.g. `Investigator 66`). **Do not** use the real or table "Character Name" strings from the mapping file in public case copy.
- Use **two-digit** strings in `docs/operations-map.html` mission `investigators` arrays (e.g. `"66"`, `"07"`).

## New case file

1. Copy `docs/templates/story-template.html` into the right category folder under `docs/stories/` (`active-leads/`, `ghost-stories/`, `cryptid-sightings/`, `conspiracy-files/`). The template nav matches **`docs/home.html` / `docs/stories.html`** (OPERATIONS MAP + LORE CODEX + full link set).
2. Replace every `[BRACKET]` placeholder; set `<title>`, case id (`CIB-###`), classification, dates, lead investigator line (`Investigator [NN]`), paths.
3. **Asset paths:** from `docs/stories/<category>/`, CSS/JS use `../../css/` and `../../js/`; links to hub pages use `../../home.html`, etc.
4. **Case numbers:** follow the bands in the template HTML comment (e.g. ghosts `CIB-001`–`099`, cryptids `100`–`199`, conspiracy `200`–`299`, other `300+`). Reuse a number only if intentionally the same case.

## Operations map

- In `docs/operations-map.html`, add a `missions` entry if missing: `id`, `title`, `status` (`active` | `closed` | `resolved`), `date`, `location`, `investigators` (array of **numeric strings**), `lat`, `lng`, `href` (path relative to `docs/`, e.g. `stories/ghost-stories/bloodlines.html`), `summary` (one line, in-universe).

## Lore codex

- If the session confirms or introduces a **recurring entity, protocol, or mystery**, add or update an entry on `docs/lore.html` (card in the right section). Standalone pages: `docs/lore/*.html` from `docs/templates/lore-template.html`. Cross-link from the case file where useful.

## Main hub and index of cases

- **`docs/home.html`:** Update the **marquee** line for active beats if appropriate; add or refresh the top row in **LATEST INVESTIGATIONS** (date `MM/DD/99`, link, location, status styling consistent with existing rows).
- **`docs/stories.html`:** Add or update the **story-card** in the correct category (`#active-leads`, `#ghost-stories`, …): case id, date, threat badge, blurb, tags (use `INVESTIGATOR NN` in tags when listing a lead).

## Optional / consistency pass

- **`docs/js/site-config.js`:** When publishing content for a new in-world “current” date, update `SITE_CONFIG.GAME_DATE` so `last-updated` behavior stays aligned.
- **`docs/investigators.html` / `docs/characters/investigator-NN.html`:** Update only if the case materially changes how an investigator should be summarized or linked.
- **`docs/evidence.html`:** Update only if you are adding a vault item that should appear in the evidence gallery (most session updates are case-file-only).

## Checklist (new case)

- [ ] Case HTML from `story-template.html`; investigator labels are **Investigator NN** only  
- [ ] `docs/stories.html` category card  
- [ ] `docs/home.html` table + marquee if needed  
- [ ] `docs/operations-map.html` mission pin  
- [ ] `docs/lore.html` and/or `docs/lore/*.html` if needed  
- [ ] `docs/js/site-config.js` date if appropriate  
- [ ] `docs/investigators.html` / `docs/characters/investigator-NN.html` only if profiles change  
- [ ] `docs/evidence.html` only if adding a gallery item  

---
> Source: [mplennartz/mplennartz.github.io](https://github.com/mplennartz/mplennartz.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
