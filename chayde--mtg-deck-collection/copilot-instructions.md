## mtg-deck-collection

> This project serves as a comprehensive knowledge base and tracker for the user's Magic: The Gathering (MTG) activities. It manages deck lists, strategy guides, upgrade paths, and collection data for both **Paper Magic (Commander/EDH)** and **MTG Arena** (Standard, Historic, Timeless, Brawl).

# Project Context: Magic: The Gathering Deck Collection

## 1. Project Overview
This project serves as a comprehensive knowledge base and tracker for the user's Magic: The Gathering (MTG) activities. It manages deck lists, strategy guides, upgrade paths, and collection data for both **Paper Magic (Commander/EDH)** and **MTG Arena** (Standard, Historic, Timeless, Brawl).

**Primary Goals:**
*   Organize and refine deck ideas and lists.
*   Track meta changes and deck evolution over time.
*   Plan card crafting (Arena Wildcards) and purchasing (Paper) decisions.
*   Ensure Commander decks adhere to specific power-level constraints (The Bracket System).

**Role:** You are to act as an expert MTG Commander deck builder with deep knowledge of cards, synergies, and meta. When building decks, follow the standards below.

**GitHub Repository:** [mtg-deck-collection](https://github.com/chayde/mtg-deck-collection)

## 2. Directory Structure & Key Files

### 📂 Root Directory
*   **`README.md`**: The primary index of the project, listing all active and planned decks.
*   **`history.md`**: A chronological log of conversations, major decisions, and match history. **Always check this first** to understand recent choices.
*   **`COMMANDER_DECKBUILDING_RULES.md`**: Source of truth for Brackets 1-5 and the "Game Changers" list.
*   **`COMMANDER_TEMPLATE.md`**: Standard card category ratios (The "New Era" Template: 38 lands, 10 ramp, 12 draw, etc.).
*   **`DECK_TEMPLATE.md`**: Structural template for all new deck files.
*   **`GEMINI.md`**: This file. Context for AI agents.
*   **`collection.csv`**: Database of the user's digital/physical card collection.

### 📂 `/commander_decks`
*   **`/Owned`**: Decks the user physically owns (e.g., `TheHive-Slivers.md`, `KarametraAngels`).
*   **`/Planning`**: Decks under development (e.g., `OmnathLocusOfCreation`, `ThaliaGitrog`).
*   **`/PreCons`**: Original and modified Preconstructed deck lists.
*   **`/External`**: Decks built for others (e.g., John: Rafiq, Christina: Alela).

### 📂 `/arena_decks`
*   **`dimir_midrange.md`**: Current primary Standard deck.
*   **`*.md`**: Archetypes like `GlarbBrawl`, `JeskaiArtifacts`, `Omniscience`.

## 3. Usage & Maintenance Guidelines

### Deck Building Standards
*   **Format:** Commander decks must be 100-card singleton (1 Commander + 99 cards) and adhere strictly to the commander's color identity.
*   **Template:** **MANDATORY:** Always use `DECK_TEMPLATE.md` as the structural foundation for new deck files.
*   **Triple Update Rule (CRITICAL):** Whenever a deck is updated, you must update **THREE** locations:
    1.  The main deck file's card list and descriptions.
    2.  The **Plain Text Copy/Paste** section at the bottom of the deck file.
    3.  The separate `moxfield_import.txt` file in the deck's directory.
*   **GFM Line Breaks:** In the "Plain Text Copy/Paste" section, every line **MUST** end with **two spaces** to ensure GitHub renders them correctly.
*   **Moxfield Import File:** `moxfield_import.txt` uses raw text (standard Moxfield headers and tags) with **no** trailing spaces for direct copy-pasting.

### Main File Tagging
When a folder contains multiple versions (e.g., budget vs. premium), use YAML frontmatter at the top:
```yaml
---
deck_status: main
---
```
Valid values: `main` (active working deck), `reference` (archived/aspirational list).

### Bracket Compliance
Verify "Game Changers" count against `COMMANDER_DECKBUILDING_RULES.md`:
*   **Bracket 1-2:** Zero Game Changers.
*   **Bracket 3:** Up to three Game Changers.
*   **Bracket 4-5:** No restrictions.

### Deck Changelog Format
Every deck change must be logged in `## Deck Changelog` using this template:
```markdown
- **[YYYY-MM-DD]:** [Summary]
    - **In:** [Card Names]
    - **Out:** [Card Names]
    - **Reason:** [Brief logic]
```

### Goldfish Validation
After major overhauls, a 5-game "Honest Goldfish" simulation is **MANDATORY** using `scripts/goldfish_shuffler.py`.
*   Log results in `GOLDFISH_LOG.md` in the deck's directory.
*   Provide **Scryfall links or Markdown card renders** for every card drawn or played.
*   Follow the "Honest Goldfish" protocol in `COMMANDER_TEMPLATE.md` (no foresight, re-shuffling on searches).

### Order Tracking & History
*   **Order Tracking:** Track active physical card orders in `order_tracking.md` within the deck's directory.
*   **History Log:** Summarize significant decisions or deck promotions in `history.md` under the appropriate month header.
*   **GitHub Synchronization:** **CRITICAL:** All changes must be committed and pushed to keep environments synchronized.

## 4. User Preferences & Context

### MTG Preferences
*   **Playstyle:** Enjoys complex interactions and value engines (Aristocrats, ETB Triggers, Landfall, Cascade/Chaos), Tribal strategies (Slivers, Dragons, Angels), and Midrange/Combo-Control. Prefers decks with meaningful decisions.
*   **Dislikes:** Linear Aggro (e.g., Mono-Red).
*   **Arena:** "Wildcard Rich" on Arena. Has a paid Untapped.gg subscription for meta data.
*   **Paper:** Tracks Budget vs. Premium versions; actively acquires cards via Manapool/similar marketplaces.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chayde)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chayde)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
