## kanban-starter

> - Corpus is ~1,136 words - fits in a single context window. You may not need a graph.

# Graph Report - .  (2026-06-15)

## Corpus Check
- Corpus is ~1,136 words - fits in a single context window. You may not need a graph.

## Summary
- 32 nodes · 31 edges · 7 communities (5 shown, 2 thin omitted)
- Extraction: 90% EXTRACTED · 10% INFERRED · 0% AMBIGUOUS · INFERRED: 3 edges (avg confidence: 0.92)
- Token cost: 2,500 input · 800 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Card CRUD Operations|Card CRUD Operations]]
- [[_COMMUNITY_Project Configuration|Project Configuration]]
- [[_COMMUNITY_App Entry and Tooling|App Entry and Tooling]]
- [[_COMMUNITY_Card Movement Lab Task|Card Movement Lab Task]]
- [[_COMMUNITY_Session 1 Deliverables|Session 1 Deliverables]]
- [[_COMMUNITY_Setup Instructions|Setup Instructions]]

## God Nodes (most connected - your core abstractions)
1. `cards` - 5 edges
2. `load() Frontend Function` - 5 edges
3. `PATCH /api/cards/:id/move Route Handler` - 4 edges
4. `scripts` - 3 edges
5. `app` - 3 edges
6. `GET /api/cards Route Handler` - 3 edges
7. `POST /api/cards Route Handler` - 3 edges
8. `move() Frontend Function` - 3 edges
9. `DELETE /api/cards/:id Route Handler` - 2 edges
10. `Kanban Cards Test Suite` - 2 edges

## Surprising Connections (you probably didn't know these)
- `addCard() Frontend Function` --semantically_similar_to--> `POST /api/cards Route Handler`  [INFERRED] [semantically similar]
  public/index.html → server.js
- `load() Frontend Function` --calls--> `GET /api/cards Route Handler`  [EXTRACTED]
  public/index.html → server.js
- `del() Frontend Function` --calls--> `DELETE /api/cards/:id Route Handler`  [EXTRACTED]
  public/index.html → server.js
- `move() Frontend Function` --calls--> `PATCH /api/cards/:id/move Route Handler`  [EXTRACTED]
  public/index.html → server.js
- `Session 2 Workshop Deliverables` --references--> `PATCH /api/cards/:id/move Route Handler`  [EXTRACTED]
  README.md → server.js

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Card Creation Flow** — public_index_addcard, server_post_api_cards, test_cards_test_post_cards [INFERRED 0.95]
- **Card Movement TDD Task** — server_patch_api_cards_move, test_cards_test_patch_cards_move, public_index_move, readme_session_2_deliverables [INFERRED 0.95]

## Communities (7 total, 2 thin omitted)

### Community 0 - "Card CRUD Operations"
Cohesion: 0.24
Nodes (10): addCard() Frontend Function, del() Frontend Function, load() Frontend Function, render() Frontend Function, cards, DELETE /api/cards/:id Route Handler, GET /api/cards Route Handler, POST /api/cards Route Handler (+2 more)

### Community 1 - "Project Configuration"
Cohesion: 0.22
Nodes (8): dependencies, express, name, scripts, dev, test, type, version

### Community 2 - "App Entry and Tooling"
Cohesion: 0.40
Nodes (4): Dev Server npm Script, Test Runner npm Script, app, Kanban Cards Test Suite

### Community 3 - "Card Movement Lab Task"
Cohesion: 0.50
Nodes (5): move() Frontend Function, showToast() Frontend Function, Session 2 Workshop Deliverables, PATCH /api/cards/:id/move Route Handler, PATCH Card Move Test Case

## Knowledge Gaps
- **14 isolated node(s):** `name`, `version`, `type`, `dev`, `test` (+9 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **2 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `cards` connect `Card CRUD Operations` to `App Entry and Tooling`, `Card Movement Lab Task`?**
  _High betweenness centrality (0.216) - this node is a cross-community bridge._
- **Why does `PATCH /api/cards/:id/move Route Handler` connect `Card Movement Lab Task` to `Card CRUD Operations`?**
  _High betweenness centrality (0.105) - this node is a cross-community bridge._
- **What connects `name`, `version`, `type` to the rest of the system?**
  _14 weakly-connected nodes found - possible documentation gaps or missing edges._

---
> Source: [CHIMHEDO/kanban-starter](https://github.com/CHIMHEDO/kanban-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
