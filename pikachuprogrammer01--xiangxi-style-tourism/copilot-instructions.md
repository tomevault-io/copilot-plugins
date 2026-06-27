## xiangxi-style-tourism

> **Don't assume. Don't hide confusion. Surface tradeoffs.**

# CLAUDE.md

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Project

湘西风情 (Xiangxi Style) — a tourism & cultural showcase SPA for Xiangxi, China. College students graduation project (前端方向专科毕设). Mandatory requirements: 12 pages, 5 CSS+DIV layout types, 3+ animation effects, responsive (desktop/tablet/mobile), mock data, `scoped` styles with BEM naming.

## Stack

- **Framework**: Vue 3 (Composition API, `<script setup>`), JavaScript (no TypeScript)
- **Build**: Vite 7
- **Routing**: Vue Router 5 (history mode, `@/` = `./src/`)
- **State**: Pinia 3 (stores: user, attraction, food)
- **UI Library**: Element Plus (per spec — not yet in `package.json`)

## Commands

```bash
npm run dev       # Start dev server (HMR)
npm run build     # Production build
npm run preview   # Preview production build
npm run format    # Prettier (semi:false, singleQuote:true, printWidth:100)
```

Node >= 20.19 or >= 22.12 required.

## Directory structure (target per SPEC)

```
src/
├── assets/
│   ├── images/        # logo, attractions/, foods/, carousel/, icons/
│   └── styles/        # main.css, variables.css, animations.css
├── components/
│   ├── common/        # HeaderNav, FooterBar, BreadcrumbNav, BackToTop, LoadingSkeleton, EmptyState
│   ├── home/          # HeroCarousel, HotRecommend, CultureHighlights, FoodPreview
│   ├── attraction/    # AttractionCard, AttractionSidebar, AttractionTabs
│   ├── food/          # FoodCard, FoodSidebar
│   └── user/          # UserProfileCard, UserStats, FavoriteList
├── views/             # Home, Login, Register, Attractions, AttractionDetail, Foods, FoodDetail, Profile, About
├── router/index.js    # Routes + global guard for requiresAuth
├── stores/            # user.js, attraction.js, food.js
├── composables/       # useAuth, useFavorites, useScroll, useResponsive
├── utils/             # request.js, validators.js, formatters.js, storage.js
├── constants/         # index.js, attractions.js, foods.js, mockData.js
├── App.vue            # Root: <RouterView />
└── main.js            # createApp → use(pinia) → use(router) → mount
```

## Routes

| Path              | View             | Auth required | Title suffix |
| ----------------- | ---------------- | ------------- | ------------ |
| `/`               | Home             | No            | 首页         |
| `/login`          | Login            | No\*          | 登录         |
| `/register`       | Register         | No\*          | 注册         |
| `/attractions`    | Attractions      | No            | 景点总览     |
| `/attraction/:id` | AttractionDetail | No            | {{name}}     |
| `/foods`          | Foods            | No            | 美食推荐     |
| `/food/:id`       | FoodDetail       | No            | {{name}}     |
| `/profile`        | Profile          | Yes           | 个人中心     |
| `/about`          | About            | No            | 关于我们     |
| `*`               | 404              | No            | 页面未找到   |

\* Redirect to `/` if already logged in. Guard: `beforeEach` checks `meta.requiresAuth` — unauthenticated users redirect to `/login?redirect=...`.

## Code conventions

- **CSS**: BEM naming (`.block__element--modifier`), `<style scoped>` on every component. Root class matches component kebab-case name (e.g. `AttractionCard.vue` → `.attraction-card`).
- **JS**: `semi: false`, `singleQuote: true`, `printWidth: 100` (Prettier). No TypeScript.
- **Vue**: `<script setup>` with Composition API. Imports use `@/` alias (maps to `src/`).
- **Data flow**: Mock data lives in `src/constants/`. Views import constants, pass to components via props. Cross-page state (user, favorites, history) goes in Pinia, persisted to localStorage.
- **Layouts**: 5 prescribed CSS+DIV layout types — Banner full-width (home hero), Holy Grail/sidebar (attraction pages), CSS Grid (food grid), Flex center (login/register), Dashboard Grid (profile).

## Current state

Project is freshly scaffolded. `App.vue` renders `<RouterView />`. Router has zero active routes (all commented out). Store `counter.js` is a skeleton with commented-out example code. No views, components, styles, or data exist yet. `package.json` includes vue, pinia, vue-router, vite, prettier, and vite-plugin-vue-devtools — but NOT Element Plus (needs `npm install element-plus` when UI work begins).

The PRD.md and SPEC.md documents define the full scope — read them when designing specific features.

---
> Source: [pikachuprogrammer01/xiangxi-style-tourism](https://github.com/pikachuprogrammer01/xiangxi-style-tourism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
