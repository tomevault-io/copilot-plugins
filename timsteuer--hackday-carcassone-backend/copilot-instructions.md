## hackday-carcassone-backend

> This is the backend for a carcassone board game.

# What is this about

This is the backend for a carcassone board game.

## Game Description

Carcassonne is a tile-laying game where players dynamically build a medieval landscape, so the UI must render a growing, irregular 2D grid of square tiles. Each tile depicts segments of features—primarily cities, roads, monasteries, and fields—and must be placed adjacent to an existing tile so that all touching edges visually match (e.g., road-to-road, city-to-city). Your viewer will need to support 90-degree tile rotation to facilitate valid placements. After placing a tile, a player may place a meeple (a player pawn) onto a specific feature (like a road segment, a city section, or a field) on that newly placed tile. The UI must therefore clearly display all placed tiles in their correct grid positions and orientations, and visually associate each meeple with its specific player color and its precise location on a feature within a single tile

## Tile Information

- the tile information is stored in `data/tile.confg.json`. The different connection lists tell you wether or not you can connect a tile of that type to the top (T), bottom (B), left (L) or right (R) of the tile. The count tells you how many of these tiles are in the game. 

# Programming Specifics

- use `uv` for package management.
- use python 3.11

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/timsteuer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/timsteuer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
