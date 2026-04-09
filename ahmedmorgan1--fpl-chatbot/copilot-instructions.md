## fpl-chatbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an FPL (Fantasy Premier League) ChatBot built using a Graph-RAG architecture with Neo4j knowledge graph. The system processes natural language queries about FPL players, teams, fixtures, and statistics, then retrieves relevant information from a Neo4j graph database.

## Common Commands

### Environment Setup
```bash
# Activate virtual environment (Windows)
.venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install spaCy language model (required for entity extraction)
python -m spacy download en_core_web_sm
```

### Running the Application
```bash
# Run main application
python main.py

# Run Stage 1 tests (input processing)
python tests/test_stage1.py

# Run Stage 2 tests (baseline retrieval)
python tests/test_stage2_baseline.py
```

## Architecture

### Two-Stage Pipeline

**Stage 1: Input Processing** ([Input_Processing/](Input_Processing/))
- **Preprocessor** ([preprocessor.py](Input_Processing/preprocessor.py)): Orchestrates the entire input processing pipeline
- **Intent Classifier** ([intent_classifier.py](Input_Processing/intent_classifier.py)): Classifies queries into intents: `recommend_player`, `compare_players`, `ask_stat`, `fixture_query`, or `unknown`
- **Entity Extractor** ([entity_extractor.py](Input_Processing/entity_extractor.py)): Extracts structured entities (players, teams, positions, gameweeks, seasons) using fuzzy matching with Neo4j data and spaCy NER
- **Embedding Handler** ([embedding_handler.py](Input_Processing/embedding_handler.py)): Generates text embeddings using sentence-transformers (all-MiniLM-L6-v2)

**Stage 2: Baseline Retrieval** ([Stage2/](Stage2/))
- **BaselineRetriever** ([baseline_retrieval.py](Stage2/baseline_retrieval.py)): Executes Cypher queries against Neo4j based on intent and extracted entities
  - `recommend_players()`: Returns top players by season points with filters for team, position
  - `ask_stat()`: Aggregates season statistics for a specific player
  - `compare_players()`: Compares statistics for 2 players side-by-side
  - `fixture_query()`: Retrieves fixtures for team/gameweek/season

### Neo4j Knowledge Graph Schema

**Nodes:**
- `Player` (properties: `player_name`, `player_element`)
- `Team` (properties: `name`)
- `Season` (properties: `season_name`)
- `Gameweek` (properties: `GW_number`)
- `Fixture` (properties: `fixture_number`, `kickoff_time`, `season`, `GW_number`)
- `Position` (properties: `name` - GK, DEF, MID, FWD)

**Relationships:**
- `(Player)-[:PLAYS_AS]->(Position)`
- `(Player)-[:PLAYED_IN]->(Fixture)` with properties:
  - **Core stats**: `minutes`, `total_points`, `goals_scored`, `assists`, `bonus`, `bps`, `form`, `value`, 
  - **Defensive**: `clean_sheets`, `goals_conceded`, `own_goals`, `saves`, `penalties_saved`
  - **Discipline**: `yellow_cards`, `red_cards`, `penalties_missed`
  - **ICT Index**: `influence`, `creativity`, `threat`, `ict_index`
- `(Season)-[:HAS_GW]->(Gameweek)`
- `(Gameweek)-[:HAS_FIXTURE]->(Fixture)`
- `(Fixture)-[:HAS_HOME_TEAM]->(Team)`
- `(Fixture)-[:HAS_AWAY_TEAM]->(Team)`

### Utilities

- **Neo4jConnection** ([Utils/neo4j_connection.py](Utils/neo4j_connection.py)): Manages connection to hosted Neo4j instance using neo4j+ssc protocol

## Key Implementation Details

### Entity Extraction Strategy
The entity extractor uses a multi-layered approach:
1. **Lazy loading**: Players, teams, and seasons are loaded from Neo4j on first use
2. **Fuzzy matching**: Uses `rapidfuzz` library with adaptive thresholds (78-88% depending on context)
3. **spaCy NER**: Prioritizes PERSON entities for player name detection
4. **Compare-aware extraction**: Special logic for "A vs B" patterns splits query into chunks and matches each side independently
5. **Season normalization**: Supports patterns like "2022-23", "22-23", "this season", "last season"

### Intent Classification Rules
- Returns `unknown` for price/value/budget queries (not in KG schema)
- Detects comparison through keywords: "compare", "vs", "versus", "or" (with 2+ proper nouns), "better than"
- Fixture intent only when explicit keywords present: "gw", "gameweek", "fixture", "match"
- Stats intent for: "goals", "assists", "points", "scored", "how many"

### Connection Details
- Neo4j instance is hosted remotely (neo4j+ssc://fc16e009.databases.neo4j.io)
- Credentials are currently hardcoded in [Utils/neo4j_connection.py](Utils/neo4j_connection.py)
- Connection uses secure SSL connection (ssc protocol)

## Development Notes

- **Python version**: Requires Python 3.7+
- **Embedding model**: Uses `all-MiniLM-L6-v2` from sentence-transformers (384-dimensional embeddings)
- **spaCy model**: Requires `en_core_web_sm` for NER and POS tagging
- **Season format**: Always use "YYYY-YY" format (e.g., "2022-23")
- **Position codes**: Use GK, DEF, MID, FWD (not full names)
- **Test coverage**: Comprehensive test suites exist in [tests/](tests/) directory for both stages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AhmedMorgan1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AhmedMorgan1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
