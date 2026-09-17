## qahera-ui-kit

> Welcome, AI Agent or Developer. This document is the supreme behavioral and architectural contract for working within the **Qahera UI Kit** codebase.

# AGENTS.md — Qahera UI Kit Governance & Agent Instructions

Welcome, AI Agent or Developer. This document is the supreme behavioral and architectural contract for working within the **Qahera UI Kit** codebase.

Every AI coding agent (Antigravity, Claude Code, Cursor, Copilot, etc.) operating in this repository **MUST** adhere to the instructions and invariants defined herein without exception.

---

## 1. Identity & Sovereign Ownership
 
* **Owner:** **Alwkala** (Egyptian / Regional Technology Studio).
* **Project Family:** Alwkala UI Kits (`Qahera UI Kit`, `Alex Admin Kit`, `Qena UI Kit`).
* **Ecosystem Relationship:** **Coordinated with & compatible with TidyFactor**, but **NOT** owned by it.
* **Separation of Concerns:**
  * **TidyFactor:** Context, Memory, Governance, Skills, CLI, Agent Workflows.
  * **Qahera:** AI-Native Design System, Component Contracts, Recipes, Multi-Target Renderers, Design Tokens.

### 🏛️ The Core Equation

$$\mathbf{\text{Qahera UI Kit v1.0}} = \mathbf{\text{Design System}} + \mathbf{\text{Registry}} + \mathbf{\text{AI Decision Layer}}$$

> **Rule for Agents:** Qahera is **NOT** just a "Component Library + Compiler". The compiler is a frozen, background build tool. The product is an **AI-Native UI Kit** combining a systematic design language, an authoritative canonical registry, and an explicit AI decision layer.

---

## 2. The 15 Non-Negotiable Architectural Invariants

Agents must verify every change against these 15 invariants before proposing or committing code:

1. **YAML is Authoritative:** The canonical source of truth for all tokens, contracts, and recipes is strictly YAML. JSON is generated for machine interchange only.
2. **Generated Files are Never the Source of Truth:** Never patch files inside `renderers/`, `dist/`, or `generated/` to fix a design bug. Fix the `contract/`, `token/`, or `recipe/` source, then regenerate.
3. **Tokens Precede Styling:** Hardcoded hex values, pixel dimensions, raw transitions, or arbitrary inline styles are strictly prohibited. Every visual attribute must trace back to `--qhr-*` CSS custom properties.
4. **Contracts Precede Implementations:** No component or variant may be rendered or implemented without first having a documented contract definition in `contracts/`.
5. **RTL is Infrastructure, Not a Theme:** RTL/LTR parity is built into the core via logical CSS properties (`margin-inline-start`, `padding-inline-end`, `inset-inline-start`). Never create separate `ButtonLTR` and `ButtonRTL` components.
6. **Arabic Typography Discipline:** The primary canonical fonts are **Alexandria** (headings, display, brand identity) and **Cairo** (body copy, UI labels, form controls) via `--qhr-font-family-primary`, `--qhr-font-heading`, and `--qhr-font-body`. Supported secondary alternatives include **El Messiri** (for heritage display), **Tajawal** (alternative body), and **Plus Jakarta Sans / Inter** (Latin). The font **Amiri** is strictly prohibited for UI components.
7. **Heritage Stays in Templates:** Egyptian, Cairo, and Islamic cultural motifs belong to downstream templates, boilerplates, and themes. The core design system contract remains neutral, modern, and universally portable.
8. **Renderer Parity with Zero Vocabulary Drift:** Renderers may vary in binding syntax (e.g. `data-variant="primary"` in HTML vs `$variant = 'primary'` in PHP vs `variant="primary"` in React), but must NEVER alter the semantic prop names, values, or accessibility contracts.
9. **Source Ownership over Runtime Lock-in:** Consuming projects copy and own their component source code (shadcn-style). Do not enforce unnecessary runtime dependencies or package lock-in.
10. **AI Metadata is a First-Class Requirement:** Every component recipe must expose explicit AI metadata: WHAT (purpose), WHEN (use cases), HOW (slots & props), and WHY NOT (anti-patterns & alternatives).
11. **Icons are Canonical Architecture, Never Emoji (`QAHERA-VISUAL-001`):** Emoji (e.g. 🗑️, ✕, 🚀, ❤️, ☀️, 🌙, 🏛️) and arbitrary Unicode symbols are strictly prohibited anywhere in UI components, headers, showcases, or previews. All iconography MUST reference the authoritative `icons/registry.yaml` semantic catalog (`svg_path` with 24x24 viewBox, stroke-width 2) or clean inline SVGs.
12. **Modular Stylesheet Architecture & RSC 0kb Boundary:** Component styles must never be compiled into monolithic manual files. Every component owns its atomic stylesheet in `renderers/html/native/components/*.css`, while `components.css` remains an `@import` manifest and `dist/qahera.css` is emitted via `cli/build-css.js`. React components preserve a 0kb client footprint via pure Server Components, reserving `'use client'` strictly for terminal interactive leaves.
13. **Dogfooding & Canonical Composition Requirement (`QAHERA-COMP-001`):** All showcases, preview hubs, application templates, and boilerplates MUST be constructed exclusively from registered canonical components (`qhr-*`) and patterns. Inventing ad-hoc CSS classes or unverified interactive controls is strictly prohibited. If a new UI pattern or layout is required, its contract (`contracts/`), recipe (`recipes/`), and schema must be authored, validated, and registered in the canonical registry FIRST before composing it downstream.
14. **Alpine.js Hydration & Template Integrity Protocol (`QAHERA-ALPINE-001`):**
    * **Attribute Collision Banned:** Never embed large data objects or multi-statement functions inside inline HTML attributes (`x-data="{...}"`). State and data models MUST be declared cleanly via `Alpine.data('appName', () => ({...}))` within `<script>` tags.
    * **100% Globally Unique Template Keys:** Every iterated item in `<template x-for="...">` MUST define a globally unique `:key` (e.g. `type-id` or `uniqueKey`). Never use an ambiguous `id` that can collide across different entity types (e.g. Component `Questionnaire` vs Pattern `Questionnaire`).
    * **HTML5 Interactive Descendant Prohibition:** Never place interactive elements (`<button>`, `<select>`, `<input>`, `<a>`) inside an outer `<a>` wrapper within a template. The browser's HTML parser will split the tag and produce multiple root nodes, violating Alpine's single-root requirement for `template x-for` and causing silent hydration failures.
    * **Zero Nested `x-for` Templates:** Never nest `<template x-for>` inside another `<template x-for>` for simple lists (e.g. tags or badges); render them via direct helper functions `x-html="renderTags(item.tags)"` to eliminate hydration stalls.
    * **Initial State Shallow Copies & `$nextTick`:** Array states must be initialized with distinct references (`[...DATA]`), with initial filter synchronizations wrapped in `this.$nextTick()` inside `init()` to guarantee immediate 0ms first-load rendering.
15. **Thematic Topography & Cultural Coherence (`QAHERA-THEME-001`):** Themes in Qahera UI Kit must NEVER be arbitrary abstract names (e.g. `theme-blue`, `cool-dark`) or cosmetic neighborhood labels. Every theme MUST synthesize a recognized global design movement (e.g. Art Deco, Belle Époque, Neo-Brutalism, Modern Minimalist, Vernacular Claymorphism, Desert Raw Materiality) with an authentic Egyptian cultural, architectural, or urban context (e.g. Heliopolis, Downtown Khedivial Cairo, New Cairo, Maadi, Shubra, Nubia, Sinai Bedouin, Historic Islamic Cairo). Themes operate strictly as semantic token overrides in `tokens/themes/*.yaml` via `[data-theme="..."]`, maintaining 100% component contract purity and 100% logical RTL layout.
16. **Strict Repository Isolation & Downstream Separation (`QAHERA-REPO-001`):** The `Qahera-UI-Kit` repository is strictly reserved for the core AI-native design system, canonical contracts, recipes, multi-target renderers, tokens, and compilation toolchain. Downstream consumer projects — including the official marketing website (`c:\wamp64\www\Qahera`), customer applications, and raw design assets — belong strictly in separate repositories or dedicated workspace roots. AI agents are strictly prohibited from creating marketing website directories (e.g. `site-marketing/`, `style-guide/`, `marketing/`), committing binary design dumps (`*.zip`, `*.psd`, `*.fig`), or mixing consumer application code into the core kit Git tree. Downstream consumers consume the compiled `dist/` artifacts or `npx qahera-ui` packages.
17. **Smart Token Map & IDE CSS Custom Data Autocomplete (`QAHERA-CSS-DATA-001`):** All canonical design tokens MUST be exported to `qahera.css-data.json` conforming to Microsoft VS Code Custom Data v1.1 schema via `cli/generate-css-data.js`. Whenever tokens in `tokens/*.yaml`, `tokens.css`, or `tokens/themes/*.yaml` are modified or added, `qahera build:css-data` must be re-run to maintain 100% synchronization. This guarantees instant IDE autocomplete, rich bilingual Markdown hover cards, and syntax validation across VS Code, Cursor, and Google Antigravity IDE without manual token lookup.

---

## 3. Strict Controlled Vocabulary

Agents must NEVER invent arbitrary props, variants, or states:

| Contract Property | Canonical Allowed Values | Prohibited Slop |
|---|---|---|
| `variant` | `primary`, `secondary`, `outline`, `ghost`, `link`, `destructive` | `special`, `hero`, `nice`, `blue`, `action` |
| `size` | `xs`, `sm`, `md`, `lg`, `xl` | `tiny`, `huge`, `normal`, `medium` |
| `tone` | `neutral`, `info`, `success`, `warning`, `danger` | `error`, `positive`, `alert-red`, `good` |
| `state` | `default`, `hover`, `focus`, `disabled`, `loading` | `busy`, `blocked`, `inactive` |

---

## 4. Anti-Slop Rules

To keep the codebase pure and enterprise-ready, agents must reject and never introduce:

* ❌ **Hardcoded colors** (e.g. `#0b6bcb`, `#fff`, `rgb(...)`) — use `var(--qhr-color-...)` or semantic tokens.
* ❌ **Hardcoded physical margins/paddings** (e.g. `margin-left: 16px`) — use logical properties (`margin-inline-start: var(--qhr-space-4)`).
* ❌ **Emoji or Unicode symbols as icons** (`QAHERA-VISUAL-001`) — use registered SVGs in `icons/registry.yaml`.
* ❌ **Arbitrary component creation** — Check if existing components can compose the requirement before introducing new ones.
* ❌ **Framework-specific leaks** into contracts or recipes.
* ❌ **Mixed Alpine & React DOM ownership** — Alpine.js is dedicated to server-rendered tracks (HTML, PHP, HTMX); React tracks use pure React state.

---

## 5. Progressive AI Context Protocol (Tokens & Latency Efficiency)

When reasoning about or generating UI components, agents should request only the minimum required layer:

* **Level 0 (Index):** `ai/components.yaml` — To discover available components and categories.
* **Level 1 (Metadata):** Read component metadata (`purpose`, `use_when`, `avoid_when`).
* **Level 2 (Recipe):** Read `recipes/<name>.yaml` — Structure, tokens, and props.
* **Level 3 (Renderer):** Read the specific target renderer (`renderers/<framework>/<name>.*`).
* **Level 4 (Example):** Inspect real usage in `examples/`.

Do NOT read the entire repository when assisting the user with a single component.

---

## 6. Directory Map & Authority

```text
qahera-ui-kit/
├── SPEC.md                  # Canonical Normative Specification
├── AGENTS.md                # Agent Instructions (This file)
├── README.md                # Public documentation & overview
├── docs/                    # Subordinate Specifications (01-12) & Archive
│   ├── specs/               # Detailed technical specs
│   └── archive/             # Historical drafts (read-only reference)
├── contracts/               # Semantic vocabulary & component interfaces
├── tokens/                  # YAML tokens, tokens.css, brand.json, presets
├── icons/                   # Canonical Icon Registry (41 semantic SVGs & aliases)
├── recipes/                 # 42 YAML component recipes
├── patterns/                # 20 Compositional UX patterns
├── behavior/                # 11 Alpine.js interactive modules
├── renderers/               # Framework renderers (HTML Native/Tailwind, PHP, HTMX, React, JS)
├── schemas/                 # YAML schemas for validation
├── ai/                      # AI discovery manifests
├── cli/                     # Compiler & validation tools
└── .tidyfactor/             # Ecosystem integration adapter
```

---

## 7. Definition of Done (DoD) for Any Component

A component is considered complete only when:
- [ ] Contract is defined or verified in `contracts/components/`.
- [ ] Recipe is written in `recipes/<name>.yaml` with full AI metadata.
- [ ] Visual properties reference `--qhr-*` tokens.
- [ ] Logical CSS and RTL parity are verified.
- [ ] Behavior script is authored in `behavior/<name>.js` (if interactive).
- [ ] All 6 renderer targets (HTML Native, HTML Tailwind, PHP, HTMX, React, JS Web Components) are generated or implemented.
- [ ] Standalone interactive preview page exists in `examples/previews/`.
- [ ] Passes `node bin/qahera.js test` with 0 errors and 0 warnings.

---
> Source: [alwkala/Qahera-UI-Kit](https://github.com/alwkala/Qahera-UI-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
