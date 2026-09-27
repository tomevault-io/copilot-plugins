## narrativeengine-p

> This directory contains world lore files for an AI Game Master engine (mobile app + main app). All lore files must be formatted to be parsed by the engine's RegEx and RAG pipeline.

# World Compendium — AI Instructions

This directory contains world lore files for an AI Game Master engine (mobile app + main app). All lore files must be formatted to be parsed by the engine's RegEx and RAG pipeline.

## MANDATORY FORMAT

Every world lore file **must** follow the structure in [lore_template.md](lore_template.md) exactly. No exceptions. The format is machine-parsed — deviating from it breaks the engine.

### Key rules:

**Headers** must use `### Category — Title` format (double dash `—`, not `-`):
- `### OVERVIEW — ...`
- `### FACTION — [Name]`
- `### LOCATION — [Name]`
- `### CHARACTER — [Name]`
- `### POWER_SYSTEM — [Name]`
- `### ECONOMY — [Currency / Trade]`
- `### EVENT — [Name]`
- `### SYSTEM — Engine Seeds`

**Section numbers** (`## 1. WORLD OVERVIEW`, `## 2. FACTIONS`, etc.) must be present and in order.

**CHARACTER entries** must include all bolded fields in this exact order:
`Aliases`, `Appearance`, `Disposition`, `Personality`, `Voice`, `Status`, `Faction`, `Goals`, `StoryRelevance`, `Example Output`, `Affinity`

**Character Intro Flags** (optional, but must use exact syntax if used):
- `**Wandering: true**` — character can appear anywhere
- `**Location: [Place Name]**` — character is place-bound
- `**Intro Boost: [keyword1, keyword2]**` — triggers on these GM narration keywords

**Engine Seed Tags** (end of file, required — copy this block structure exactly):

```markdown
## 6. ENGINE SEED TAGS (IMPORTANT)
### SYSTEM — Engine Seeds

**── TIER 1: SURPRISE ENGINE (mundane world flavor) ──**
**Surprise Types:** [5-10 mundane situation archetypes]
**Surprise Tones:** [5-10 emotional flavors]

**── TIER 2: ENCOUNTER ENGINE (location-agnostic threat situations) ──**
**Encounter Types:** [5-10 threat situation archetypes — write SITUATIONS not enemy names]
**Encounter Tones:** [5-10 tones]

**── TIER 3: QUEST HOOK ENGINE (world rumours & local hooks) ──**
**Quest Hook Who:** [5-10 rumour sources]
**Quest Hook What:** [5-10 inciting events]
**Quest Hook Where:** [5-10 local areas]
**Quest Hook Why:** [5-10 stakes/hooks]
```

The tier header lines (`**── TIER X: ... ──**`) must be present verbatim — they are parsed as delimiters. The section number for Engine Seed Tags varies (it comes after Economy/Events) — just keep it as the final `##` section.

## FILE NAMING

World lore files follow the pattern: `world_lore_[worldname].md`
Starter prompts follow: `[worldname]_starterPrompt.md` or `starter_prompt.md`

## CONTENT RULES

- No AI name-slop (no "Aethermancer Zyn'kael the Voidweaver" type names — keep names grounded and pronounceable)
- No default fantasy/sci-fi power clichés — every power system must have a specific cost or limitation
- Tonal variety is preferred over monotone grimdark
- NPC characters must feel like real people with conflicting loyalties — no sycophant companions
- All factions should have internal tensions, not just be monolithic good/evil blocs

## WHEN ADDING NEW LORE

1. Use `lore_template.md` as your structural skeleton
2. Generate content that fits the world's established tone and power ceiling
3. All new CHARACTERS must have all required fields filled — no placeholders
4. Engine Seeds must be tailored to the specific world's genre and setting
5. When expanding an existing world file, match the existing style and header format exactly

---
> Source: [Sagesheep/NarrativeEngine-P](https://github.com/Sagesheep/NarrativeEngine-P) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
