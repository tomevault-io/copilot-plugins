## wanderland

> **Project:** Wanderland\

# AGENTS.md — Guidance for Codex Agents

**Project:** Wanderland\
**Purpose:** A radically simplified land finder that overlays and filters **pre-made GeoJSONs** to help users find sovereign land in Canada. No ETL. No data science. Just maps.

---

## Project Overview

Wanderland is a **GeoJSON-first** mapping tool:

- **All datasets are stored as GeoJSON**
- The app **loads, overlays, and filters** these datasets in real time
- The user experience is **an interactive map** with filter controls

---

## Agent Goals

Codex Agents should:

- Prioritize **clarity over cleverness**
- Write **modular filters** (one function = one filter)
- Always assume **input and output are GeoJSON**
- Never introduce unnecessary data processing pipelines
- Keep functions **pure and auditable**
- Make sure to document changes and update them if needed in the readme and in accompanying /docs folder.

---

## Code Conventions

**Languages:**

- JavaScript/TypeScript (frontend)
- Python (optional for scoring or batch jobs)

**Standards:**

- JS/TS: `prettier`, `eslint`
- Python: `black`, `flake8`
- Filenames: `snake_case.py`, `camelCase.js`

**Style:**

- Use **named imports/exports**
- Comment all public functions and complex logic
- Keep files small; one feature = one file

---

## Filters: Key Rules

- Filters operate on **GeoJSON FeatureCollections**
- Filters can be **attribute-based** (e.g., property thresholds) or **geometry-based** (e.g., distance to water)
- Filters should be **composable**:

```js
const nearWater = (feature) => feature.properties.water_distance < 1000;
const lowTax = (feature) => feature.properties.tax_rate < 0.05;
```

- The UI applies one or more filters, producing a **filtered GeoJSON layer**

---

## PR Guidelines

**Title:**

```
[Filter/Feature/Fix] Short description
```

**Body:**

- What changed and why
- How to test it
- Screenshots (if applicable)

---

## Minimum Working Product (MVP)

Must support:

- Loading multiple GeoJSON layers
- Applying composable filters
- Displaying filtered map overlays
- Optional: exporting filtered results

---

## Do's & Don'ts

✅ DO:

- Use GeoJSON everywhere
- Build tiny, reusable filter functions
- Keep the interface dead simple

❌ DON'T:

- Add complex data processing or server-side transformations
- Use non-spatial formats unless absolutely required
- Build monolithic code blocks

---

## Final Note

Wanderland is for **visual decision-making**, not for perfect data science. The priority is **speed, clarity, and sovereignty-first thinking**.

Build tools that help people see, not tools that bury them in complexity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twonthebuilder)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/twonthebuilder)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
