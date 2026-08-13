## yishiweb

> Pure static vanilla-JS I-Ching divination SPA. Hosted at **https://6yao.cn** via GitHub Pages.

# AGENTS.md — 易师 (yiShiWeb)

Pure static vanilla-JS I-Ching divination SPA. Hosted at **https://6yao.cn** via GitHub Pages.

## Tech stack
- **Zero build tools, zero dependencies.** No npm, no bundler, no transpiler.
- **No tests, no CI, no linter/formatter** — none configured.

## Project structure
- `index.html` — single entrypoint; loads every CSS and JS file via `<script>`/`<link>` tags
- `app.js` — global state, module show/hide, home settings with localStorage, character origin map
- `yizhou-data.js` — 64 hexagrams (bagua, gua data, yaoci, tuanshi) and pinyin map
- `gua-common.js` — shared hexagram rendering utilities
- `najia.js` — 纳甲 (najia) heavenly stems, earthly branches, wuxing, date tools
- `text-render.js` — HTML-safe text rendering with pinyin ruby annotations
- `char-panel.js` + `character-ancient-forms.js` — character origin/oracle bone/bronze script panel
- **9 feature modules** (each is a separate JS+CSS pair):
  `liuyao.js` (六爻), `meihua.js` (梅花易数), `huafu.js` (画符起卦),
  `guaxiang.js` (卦象练习), `practice-memory.js` (卦名记忆 + 爻辞卦辞练习),
  `chaxun.js` (六十四卦查询), `fengshui-compass.js`, `huangdao.js` (黄道吉日)
- `base.css`, `components.css`, `liuyao.css`, `meihua.css`, `huafu.css`, `fengshui-compass.css`, `huangdao.css`

## Critical: script load order
Scripts in `index.html` must load in **exactly this order** (later scripts depend on globals from earlier ones):
1. `yizhou-data.js` — baguaData, liushisiGua, pinyinMap
2. `gua-interpretation-data.js` — hexagram interpretation data
3. `gua-interpretation-overrides.js` — overrides
4. `gua-tuanshi-handwritten.js` — handwritten-style tuanshi
5. `pinyin-data.js` — pinyin map
6. `app.js` — initApp(), module switching, cxCharacterOriginMap
7. `ui-common.js`, `gua-common.js`, `najia.js`, `text-render.js`
8. `character-ancient-forms.js`, `char-panel.js`
9. Module JS files (any order): `guaxiang.js`, `practice-memory.js`, `chaxun.js`, `liuyao.js`, `meihua.js`, `huafu.js`, `fengshui-compass.js`, `huangdao.js`

## Development
- No build step — edit files directly, reload browser
- No local dev server needed (file:// or any static server works)
- `localStorage` key used: `zhouyi-home-module-settings`

## Deployment
- Push to `main` branch — GitHub Pages serves from root automatically
- No build, no CI — just `git push`
- Domain: `6yao.cn` (via `CNAME` file)

## Navigation
- Every module has exactly **two nav buttons** at top: `🏠 首页` (`navGoHome()`) and `← 上一页` (`navGoBack()`)
- `navGoHome()` always returns to home and clears navigation history
- `navGoBack()` returns to the page you came from:
  - From chaxun detail → returns to the source module (liuyao/meihua/huafu/gualibrary) or bagua select
  - From other modules → returns to previous module or home
- Cross-module navigation tracked via `navPrevModule` in `app.js`
- Flags (`fromLiuYaoDetail`, etc.) preserved for state restoration when returning to source module

## Conventions
- All UI text is **Simplified Chinese (zh-CN)**
- AI 助手始终用**中文**与用户交互
- Global functions/pollution throughout (vanilla JS, no modules)
- Module visibility handled by `showModule()`/`showHome()` in `app.js`

---
> Source: [cb252389238/yiShiWeb](https://github.com/cb252389238/yiShiWeb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
