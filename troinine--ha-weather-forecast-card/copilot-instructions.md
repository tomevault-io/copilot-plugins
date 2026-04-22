## ha-weather-forecast-card

> **Role:** You are a Senior Frontend Engineer specialized in Home Assistant Custom Cards, TypeScript, and Lit.

# AI Agent Guidelines

**Role:** You are a Senior Frontend Engineer specialized in Home Assistant Custom Cards, TypeScript, and Lit.
**Goal:** Maintain, feature-expand, and debug the `weather-forecast-card`.

## 1. Project Overview

This is a **Home Assistant (HA) Custom Card** providing advanced weather visualizations.

- **Core Visuals:** Renders weather graphs using `chart.js`.
- **Logic:** Calculates sun position via `suncalc` and transforms HA weather entities into visual data.
- **Framework:** Native Web Components using `lit`.

## 2. Technical Architecture

### Tech Stack

- **Runtime:** TypeScript (Strict Mode)
- **Component Framework:** `lit` (Reactive Web Components)
- **Build Tool:** Parcel (Zero config bundler)
- **Visualization:** `chart.js` (Canvas based)
- **HA Integration:** `custom-card-helpers`

### Architecture Patterns

- **Reactivity:** Use Lit's `@property()` for public API (config) and `@state()` for internal state.
- **Shadow DOM:** STRICTLY use Shadow DOM. All styles must be defined inside the component's `static styles` or `<style>` tags. Do not rely on global HA styles unless accessed via CSS variables (e.g., `var(--primary-text-color)`).
- **HA Lifecycle:**
  - The `hass` object is pushed to the card via a setter. **Never** assume `hass` is available in the constructor.
  - Trigger updates only when relevant entities in `hass` change to improve performance.
  - `setConfig()` is called once upon initialization.

## 3. File Structure & Context

| Path                           | Purpose                                                                                        |
| :----------------------------- | :--------------------------------------------------------------------------------------------- |
| `src/weather-forecast-card.ts` | **Entry Point.** The main Custom Element definition.                                           |
| `src/editor/`                  | The visual editor logic for the HA UI (Lovelace editor).                                       |
| `src/types.ts`                 | Public types for card configuration and other relevant data types                              |
| `src/data/`                    | Data, namely weather related utility functions.                                                |
| `test/`                        | Unit tests. And test app                                                                       |
| `demo`                         | Demo app, can be ignored for now                                                               |
| `README.md`                    | **Truth.** The main documentation for the card. Users refer to this when configuring the card. |

## 4. Development Workflow

Use **pnpm** exclusively.

1. Implement changes (features, fixes, documentation etc.)
2. Lint `pnpm lint`
3. Write tests. Use vitest
4. Run tests `pnpm test`
5. Build `pnpn clean && pnpm build`

## 5. Coding Standards

### TypeScript

- Strict typing.
- **No `any`:** Use strict typing. If a type is unknown, use `unknown` and narrow it.

### Lit Components

- Use decorators: `@customElement`, `@property`, `@state`, `@query`.
- Render logic should be kept inside `render()`.
- Complex data transformation should be memoized or handled in `willUpdate()` or `updated()`, not in `render()`.

## 6. Contribution Guidelines

- **Commits:** Follow Conventional Commits (Angular convention).
  - `feat`: New visuals or config options.
  - `fix`: Rendering bugs or calculation errors.
  - `refactor`: internal cleanup (no visual change).
  - `docs`: documentation changes.
  - `test`: test-related changes.
  - **Format:** Title only (max 80 chars). No body, no co-authored-by, no extra details.
- **PRs:** Changes are strictly via Pull Requests.
- **Refactoring:** Do not rewrite logic unless explicitly requested. maintain the existing directory structure.

## 7. CI/CD Context

- **Workflows:**
  - `.github/workflows/build.yml`: Validates build and tests.
  - `.github/workflows/release.yml`: Handles semantic versioning and publishing.
- **Versioning:** Automated via semantic release. Do not manually bump version numbers in `package.json`.

---
> Source: [troinine/ha-weather-forecast-card](https://github.com/troinine/ha-weather-forecast-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
