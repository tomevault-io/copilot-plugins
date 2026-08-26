## df-legends-sil-mamgoz

> This project extracts and transforms Dwarf Fortress legends data into readable narratives and story documents.

# CLAUDE.md

This project extracts and transforms Dwarf Fortress legends data into readable narratives and story documents.

## Project Structure

```
df-claude-legends/
├── world/                        # Dwarf Fortress world data
│   ├── SilMamgoz/                # Raw DF save files (do not touch)
│   ├── legends/                  # Legends exports organized by year
│   │   ├── 500/                  # Year 500 exports
│   │   │   ├── *-legends.xml     # Standard legends export from DF
│   │   │   └── *-legends_plus.xml # Extended export from DFHack
│   │   └── 501/                  # Year 501 exports (latest)
│   │       ├── *-legends.xml
│   │       └── *-legends_plus.xml
│   └── parsed/                   # Split XML files (see below)
├── stories/                      # Generated narratives and character studies
│   └── {character}/              # Per-character story folders
├── scripts/                      # Utility scripts
│   ├── parse_legends.py          # XML splitter script
│   ├── lookup_figure.py          # Find figures by ID, name, or race
│   ├── lookup_site.py            # Find sites by ID, name, or type
│   ├── lookup_entity.py          # Find entities/civilizations
│   ├── lookup_events.py          # Find events by figure, site, type, year
│   ├── lookup_creature.py        # Translate creature IDs to names
│   └── figure_history.py         # Generate full timeline for a figure
├── .claude/                      # Claude Code configuration
│   └── settings.json             # Permissions for auto-approved scripts
└── README.md
```

## Folder Guidelines

### `world/SilMamgoz/`
Raw Dwarf Fortress save data. **Leave this alone.** Contains compressed game data.

### `world/legends/{year}/`
Legends exports organized by in-game year. Each folder contains:
- `*-legends.xml` - Standard legends export from DF
- `*-legends_plus.xml` - Extended export from DFHack with additional data

The latest year folder contains the most complete history. Keep these as source of truth but **prefer using parsed/ files** for lookups.

### `world/parsed/` (Preferred for lookups)
Pre-split XML files organized by category. **Use these instead of the monolithic XMLs** to avoid loading unnecessary data.

```
parsed/
├── world_info.xml              # World name: "Sil Måmgoz" (The Plane of Dragons)
├── geography/                  # Regions, rivers, mountains, underground
├── entities/                   # Civilizations, sites, populations
├── figures/                    # Historical figures
│   ├── historical_figures_all.xml      # Complete (2.2 MB)
│   ├── historical_figures_plus.xml     # Extended data from DFHack
│   └── by_race/                        # Split by race for targeted lookups
│       ├── dwarf.xml (434 figures, 1 MB)
│       ├── human.xml (264 figures, 620 KB)
│       ├── kobold.xml (28 figures)
│       ├── elf.xml (19 figures)
│       └── ... (100+ race files)
├── events/                     # Historical events (17,224 total)
│   ├── by_type/                        # Split by event type
│   │   ├── hf_died.xml (1,006 deaths)
│   │   ├── hf_simple_battle_event.xml (5,573 battles)
│   │   ├── creature_devoured.xml (4,810 events)
│   │   └── ... (60+ event types)
│   ├── by_year/                        # Split by 50-year ranges
│   │   ├── years_0000-0049.xml
│   │   ├── years_0050-0099.xml
│   │   └── ...
│   ├── event_collections.xml           # War/battle groupings
│   └── event_relationships.xml         # Figure relationship events
├── culture/                    # Artifacts, writings, music, poetry, dance
└── reference/                  # Creature definitions, historical eras
```

See `world/parsed/INDEX.md` for complete file listing with sizes.

### `stories/`
Contains narrative documents crafted from the legends data. Organized by subject (character, civilization, event, etc.).

## Working with This Project

### Lookup Scripts (Preferred Method)
Use these scripts for fast, formatted lookups. They read from `world/parsed/` and resolve IDs to names automatically.

```bash
# Find a figure by ID or name
python3 scripts/lookup_figure.py 123
python3 scripts/lookup_figure.py "sodel"
python3 scripts/lookup_figure.py --race dwarf --brief

# Find a site by ID or name
python3 scripts/lookup_site.py 8
python3 scripts/lookup_site.py "longpaints"
python3 scripts/lookup_site.py --type fortress

# Find entities/civilizations (includes population counts)
python3 scripts/lookup_entity.py 26
python3 scripts/lookup_entity.py --list
# Output: ENTITY #30: The Unswerving Fells
#         Race: Elf
#         Type: Civilization
#         Population: 20 elf

# Find events (combinable filters)
python3 scripts/lookup_events.py --figure 123
python3 scripts/lookup_events.py --site 8 --year 50-100
python3 scripts/lookup_events.py --type hf_died --limit 50
python3 scripts/lookup_events.py --list-types

# Translate creature IDs to names
python3 scripts/lookup_creature.py NIGHT_CREATURE_3
python3 scripts/lookup_creature.py --list-special

# Get complete timeline for a figure
python3 scripts/figure_history.py 107
python3 scripts/figure_history.py "fimshel"
python3 scripts/figure_history.py 45 --brief
```

### Manual Lookups (parsed/ files)
For direct XML access when scripts don't cover a use case:
1. **Find a figure by race**: Check `parsed/figures/by_race/{race}.xml`
2. **Find death events**: Check `parsed/events/by_type/hf_died.xml`
3. **Find events in a time period**: Check `parsed/events/by_year/years_XXXX-XXXX.xml`
4. **Get civilization info**: Check `parsed/entities/entities_plus.xml`

### Regenerating Parsed Files
Parse the latest year's legends data into smaller files:
```bash
python3 scripts/parse_legends.py          # Parses latest year (auto-detected)
python3 scripts/parse_legends.py 501      # Parse specific year
```

### Creating Narratives
1. Identify subject (figure, site, event, etc.)
2. Load relevant parsed XML files
3. Cross-reference IDs between files (e.g., figure's entity_link → entities file)
4. Transform historical records into readable narrative
5. Store output in `stories/{subject}/`

### Story Output Structure
Each story folder should contain two files:
- `outline.md` - Structural reference with facts, timelines, family trees, skill stats, and source citations
- `story.md` - Creative narrative with embellished prose and character dialog

### Lessons Learned

#### Creature Name Lookups
Generated creatures (night creatures, forgotten beasts, titans) have IDs like `NIGHT_CREATURE_3`. Use the lookup script:
```bash
python3 scripts/lookup_creature.py NIGHT_CREATURE_3
# Output: NIGHT_CREATURE_3: monster of shadow

python3 scripts/lookup_creature.py --list-special
# Lists all night creatures, forgotten beasts, and titans with their names
```

#### Transformation Events
For vampire/werewolf/curse stories, `parsed/events/by_type/changed_creature_type.xml` is essential. It shows:
- Who transformed whom (`changer_hfid` → `changee_hfid`)
- Original and new race
- Exact year and time

#### Family Relationships
The `<hf_link>` tags in figure files define relationships:
- `<link_type>mother</link_type>` / `father` / `child`
- `<link_type>spouse</link_type>` / `former spouse`
- `<link_type>deity</link_type>` (worship relationships)

**Caution:** Trace relationships carefully. A figure's `former spouse` may have children with them listed separately.

#### Skills Inform Character Voice
Use skill values to shape dialog and personality:
- High `PERSUASION`/`NEGOTIATION` → diplomatic speech patterns
- High craft skills (`FORGE_ARMOR`, `PAPERMAKING`) → devotion to craft in dialog
- High combat skills → confident, predatory demeanor
- High `LYING`/`JUDGING_INTENT` → manipulative or perceptive character

#### Timeline Verification
**Always verify ages and dates before writing.** Common calculation:
- Figure's age at event = `event_year - birth_year`
- Years since transformation = `current_year - transformation_year`

Write the outline with verified dates first, then reference it while writing the narrative.

#### Narrative Immersion
When writing stories, translate raw IDs to narrative-friendly text:
- "Entity 32" → "the human settlement" or use site name
- "Deity 44" → "his god" or "the god of craft" (check deity's spheres)
- "Site 69" → use the site's actual name from sites.xml

#### Lair and Site Lookups
Figures often have `<site_link><link_type>lair</link_type>` pointing to their home. Cross-reference with `parsed/entities/sites.xml` to get the lair's name and type.

#### Historical Figures vs Population
**Critical distinction:**
- `parsed/figures/by_race/*.xml` = **Historical Figures** (notable named individuals)
- `parsed/entities/entity_populations_plus.xml` = **General Population** (unnamed members)

A civilization can have all historical figures dead but still have living population. Example:
- The Unswerving Fells has 19 dead historical figures (elf.xml)
- But still has 20 living elves (entity_populations_plus.xml: `<race>elf:20</race>`)

**Always check population before declaring a civilization extinct.**

---
> Source: [Tpikl/df-legends-sil-mamgoz](https://github.com/Tpikl/df-legends-sil-mamgoz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
