## pure-ds

> ﻿# PDS (Pure Design System) — AI Instructions

﻿# PDS (Pure Design System) — AI Instructions

> **CRITICAL**: This workspace uses **Pure Design System (PDS)**. All code generation MUST follow PDS and vanilla Web Platform patterns. Never use 3rd party framework patterns, non-PDS utility classes, inline styles, or hardcoded CSS values.

> **CRITICAL — Unless in the pure-ds origin project, `/pds/` PATHS ARE READ-ONLY**: Files under `public/pds/**`, `public/assets/pds/**`, and `node_modules/@pure-ds/core/public/**` are immutable package output. To change behavior, edit source (`src/js/pds-core/**`, `pds.config.js`) then rebuild.

> **Note: In the pure-ds (@pure-ds/core origin) project itself**, `pds-*` web components in `pds/components/` ARE their own source — not compiled from elsewhere.

---

## Philosophy

PDS follows the [Pure Web Manifesto](https://pureweb.dev/manifesto): "The browser is the framework."

- **Standards-first**: Web Platform APIs only (no framework dependencies)
- **Configuration-driven**: `pds.config.js` generates all CSS (tokens, scales, semantics, surfaces, states). Zero specificity via `:where()`.
- **Progressive Enhancement**: Semantic HTML first, enhance where needed
- **Layers**: 1 — Styles/tokens → 2 — Enhancements (`data-*`) → 3 — Web Components (`pds-*`) → 4 — LLM Support

> PDS web components are lazy-loaded from `[public-root]/assets/pds/components/` — never bundled by the consuming project. The PDS AutoDefiner registers them when added to the DOM.

---

## Single Sources of Truth

> In consuming projects, prefix paths with `node_modules/@pure-ds/core/`.

| Need | SSoT File |
|------|-----------|
| CSS Tokens | `public/assets/pds/pds.css-data.json` |
| Web Component APIs | `custom-elements.json` |
| HTML tags & attributes | `public/assets/pds/vscode-custom-data.json` |
| Primitives & Utilities | `src/js/pds-core/pds-ontology.js` |
| Enhancement metadata | `src/js/pds-core/pds-enhancers.js` |
| Config types & token relations | `src/js/pds-core/pds-config.js` |

**Prefer reading these files over guessing class names or token names.**

---

## 🔌 MCP Lookup Protocol — MANDATORY

**Before generating any PDS code, call the relevant MCP tool(s) first. Never invent class names, tokens, attributes, or component APIs.**

| Topic | MCP Tool |
|-------|----------|
| CSS tokens (`--color-*`, `--spacing-*`, etc.) | `get_tokens` |
| Buttons, cards, layout, surfaces, utilities | `find_utility_class` |
| Any `pds-*` component | `get_component_api` |
| Any `data-*` enhancement | `get_enhancer_metadata` |
| Forms, localization, DOM building, toasts, treeview | `query_design_system("your question")` |
| Config / token naming rules | `get_config_relations` |
| After generating HTML | `validate_pds_snippet` |

**`query_design_system` auto-injects best-practice guidance with code examples** based on your question. Use it for: pds-form patterns, submit handlers, localization, `parse`/`html` DOM building, `PDS.toast`/`PDS.ask`, treeview lazy loading, Lit import maps, empty states, and more.

**If MCP is unavailable**: fall back to reading SSoT files directly. Never block on MCP.

---

## Intent Scoping

Match implementation to the smallest relevant layer first:

1. **Layer 1**: CSS tokens + utility classes + primitive classes (`.card`, `.btn-*`, `.flex`, `.grid-*`)
2. **Layer 2**: `data-*` enhancements (`data-dropdown`, `data-toggle`, `data-required`)
3. **Layer 3**: Web Components — only when native HTML has no equivalent

Do not inspect or modify `pds-form` unless the request explicitly involves that component.

---

## Critical Anti-Patterns

- ❌ `style="..."` → use `var(--token-name)` CSS custom properties
- ❌ Hardcoded colors / spacing → tokens: `--color-*`, `--spacing-*`, `--radius-*`
- ❌ `alert()` / `confirm()` / `prompt()` → `PDS.ask()` / `PDS.toast()`
- ❌ Manual dropdown / modal / tab → `<nav data-dropdown>`, `PDS.ask()`, `<pds-tabstrip>`
- ❌ Native `submit` event on pds-form → listen to `pw:submit`
- ❌ Accessing lazy component before definition → `await customElements.whenDefined('tag-name')`
- ❌ Missing `btn-working` class during async button operations
- ❌ `<div class="button">` or other non-semantic HTML → `<button class="btn-primary">`
- ❌ `ui:icon` on non-text-like inputs (date-range, textarea, select, omnibox)
- ❌ Importing localization helpers from `#pds/lit` → import from `#pds`

---

## Summary Checklist

Before generating any code:

1. ✅ **Call MCP first** — `query_design_system`, `get_component_api`, `find_utility_class` as needed
2. ✅ **No inline styles** — CSS custom property tokens only
3. ✅ **No browser dialogs** — `PDS.ask()` and `PDS.toast()`
4. ✅ **Semantic HTML** — `<button>`, `<nav>`, `<article>`, `<label>`, `<details>`
5. ✅ **Enhancements via `data-*`** — not manual JavaScript
6. ✅ **Components as last resort** — prefer primitives and enhancements
7. ✅ **Await lazy-loaded components** — `await customElements.whenDefined()`
8. ✅ **Validate generated HTML** — `validate_pds_snippet`

> **For all topic-specific patterns** (pds-form, localization, DOM building, toasts, treeview, Lit import maps), call `query_design_system("your question")` — it returns authoritative guidance with live code examples straight from source.

---
> Source: [Pure-Web-Foundation/pure-ds](https://github.com/Pure-Web-Foundation/pure-ds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
