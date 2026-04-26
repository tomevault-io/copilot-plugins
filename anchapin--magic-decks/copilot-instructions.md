## magic-decks

> This project is a collection of Magic: The Gathering deck ideas and lists, organized for easy import into MTG Arena. All deck lists are stored in MTG Arena compatible format for seamless importing.

# Magic: The Gathering Deck Collection

## Project Overview
This project is a collection of Magic: The Gathering deck ideas and lists, organized for easy import into MTG Arena. All deck lists are stored in MTG Arena compatible format for seamless importing.

## Directory Structure
```
magic_decks/
├── standard/          # Standard format decks
├── historic/          # Historic format decks  
├── explorer/          # Explorer format decks
├── timeless/          # Timeless format decks
├── brawl/             # Brawl format decks
├── draft_ideas/       # Draft archetypes and ideas
├── brewing/           # Work-in-progress deck ideas
└── completed/         # Finalized, tested deck lists
```

## MTG Arena Deck Format Standards

### CRITICAL: Arena Import Compatibility
**MTG Arena CANNOT import deck lists with inline comments or section headers.**
- ❌ NO comments like `// === CREATURES ===` or `# Removal package`
- ❌ NO section dividers within the deck list
- ✅ Only card quantities and names in the main deck section
- ✅ Comments ONLY at the top (before "Deck") or bottom (after deck list)

### Basic Format
All deck lists should follow MTG Arena import format:
```
<quantity> <Card Name>
<quantity> <Card Name> (<Set Code>) <Collector Number>

<quantity> <Card Name>
```

### Correct Arena-Compatible Format
```
Deck
4 Lightning Bolt
4 Monastery Swiftspear
2 Embercleave
1 Castle Embereth

Sideboard
2 Roiling Vortex
3 Redcap Melee
```

### WRONG Format (Arena Import Will Fail)
```
Deck
// Creatures
4 Monastery Swiftspear
// Spells  
4 Lightning Bolt
```

### Example Brawl Deck
```
Commander
1 Alesha, Who Smiles at Death

Deck
1 Temple of Silence
1 Godless Shrine
```

## MCP Server Usage

### MTG Server (mcp__mtg-server)
- `upload_deck`: Import deck lists for testing
- `draw_card`: Simulate draws
- `play_card`: Test gameplay
- `view_hand`: Check current hand
- `view_deck_stats`: Analyze deck composition
- `mulligan`: Test opening hands
- `sideboard_swap`: Test sideboard options

### Scryfall Server (mcp__scryfall)
- `search_cards`: Find cards by query
- `get_card_by_name`: Get specific card details
- `get_random_card`: Discover new cards

## Deck Development Workflow

1. **Concept Phase**: Use Scryfall to research cards and themes
2. **Initial Build**: Create draft deck list in brewing/
3. **Testing Phase**: Upload to mtg-server for simulation
4. **Refinement**: Iterate based on testing results
5. **Final Version**: Move completed deck to appropriate format folder

## File Naming Convention
- Use descriptive names: `aggro_red_burn.txt`
- Include format prefix: `standard_mono_white_lifegain.txt`  
- Date stamp for versions: `historic_elves_v2_2025-01-15.txt`

## Tools Integration
- Always test deck lists with mtg-server before finalizing
- Use Scryfall for card research and discovery
- Validate MTG Arena format compatibility before saving

## Notes
- Keep sideboards under 15 cards for competitive formats
- Standard decks must be exactly 60 cards (minimum)
- Brawl decks must be exactly 100 cards with singleton restriction
- Include mana curve analysis in deck comments when possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anchapin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
