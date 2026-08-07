## kiwidown

> - Kiwidown is a static, browser-only WYSIWYG Markdown editor built with TypeScript, Vite, Milkdown/ProseMirror, and remark. It renders vendored Typora themes without a backend.

# AGENTS.md

## Project

- Kiwidown is a static, browser-only WYSIWYG Markdown editor built with TypeScript, Vite, Milkdown/ProseMirror, and remark. It renders vendored Typora themes without a backend.
- Use npm with the committed `package-lock.json`. CI builds with Node.js 20 via `npm ci` and `npm run build`.
- `vite.config.ts` uses `base: './'`; production output must work at a domain root, a subpath, and from disk. Resolve runtime public assets against `document.baseURI`, as `src/theme/index.ts` does, rather than assuming `/`.

## Repository map

- `src/main.ts`: application assembly and startup ordering.
- `src/editor/`: Milkdown setup and Markdown/Typora DOM behavior.
  - `typora-dom.ts`, `typora-nodes.ts`, and `footnote-definition.ts`: the main DOM-contract implementations.
  - `code-highlight/`: Prism tokenization mapped to CodeMirror 5 classes.
  - `math/` and `diagram/`: lazy KaTeX and Mermaid integrations.
- `src/doc/`: document lifecycle, file access, dirty state, recovery, and URL loading. `views.ts` retains per-tab editor state and scroll.
- `src/theme/`: runtime theme loading and the Typora compatibility/bridge CSS layers.
- `src/app/`: application chrome; `src/welcome/`: first-run and contract-check fixture.
  - `source-mode.ts`: the CodeMirror 6 Markdown source view and its extension set. `source-map.ts` pairs caret positions with the rendered document; `source-commands.ts` holds the Markdown-aware editing commands and the fold service.
  - `table-tools/`: the hover toolbar over a table — size, per-column alignment, delete. It is positioned inside the theme's `<content>` scroller but outside `#write`, because the DOM contract makes `#write` the direct parent of block content and themes may make a table a clipping box. Its room is reserved permanently as extra table margin (`--app-table-tools-space`, which the built-in themes add to their own), so appearing on hover never reflows the document.
- `doc/dom-contract.md`: authoritative specification for the DOM expected by unmodified Typora themes.
- `themes.manifest.json`: source of truth for vendored themes. `public/themes/index.json` and theme files are reconciled by `scripts/fetch-themes.mjs`.
- `public/themes/kiwi/`: the six built-in themes — source, not vendored material. `kiwi.css` holds the shared DOM-contract structure and `glassy.css` the Liquid Glass material; `su.css`, `qi.css`, `glassy-su.css` and `glassy-qi.css` are palettes only. `ye.css` starts from `kiwi.css` but deliberately overrides typography and structure for its academic-paper format; `glassy-ye.css` layers the shared glass material over it. Glassy Su/Qi import `kiwi.css`, Glassy Ye imports `ye.css`, and all three then import `glassy.css`, so they inherit the DOM-contract coverage and restate only palette differences — keep it that way rather than forking the structure.
- `fonts.manifest.json` and `public/fonts/`: webfonts our own themes ask for, reconciled by `scripts/fetch-fonts.mjs`. Fonts inside theme packs stay in their pack.
- `public/typora-base/`: committed, vendored Typora base styles. `dist/` and screenshot directories are generated and ignored.
- `scripts/`: browser regression checks, screenshot harnesses, asset fetchers, and diagnostics.

## Important contracts

- Preserve the Typora DOM contract in `doc/dom-contract.md`. Theme compatibility depends on exact elements, direct-child relationships, and classes, not merely semantic equivalence. `npm run check:contract` is the primary regression check.
- Keep the cascade order declared in `index.html`: `typora-base`, `typora-theme`, `typora-bridge`, then `app`. Theme CSS must remain a runtime `@import ... layer(typora-theme)` so relative font/image URLs resolve from the theme file. Keep `bridge.css` small; it intentionally outranks themes only where Milkdown emits DOM Typora never did.
- Do not replace the headless Milkdown assembly with Crepe or Milkdown's code-block component: their markup/CSS conflicts with the required Typora DOM.
- Markdown serialization must round-trip without content loss and become a fixed point after one normalization. In particular, keep remark-gfm `singleTilde: false`: Typora uses `~x~` for subscript and `~~x~~` for strikethrough.
- Code highlighting must emit CodeMirror 5 `cm-*` classes in addition to Prism classes. After changing grammars or `src/editor/code-highlight/token-map.ts`, run `npm run probe:tokens` and the contract check.
- All tabs share one editor. Switching documents swaps `EditorState` so selection, plugin state, and undo history survive; `src/doc/views.ts` separately retains and stabilizes scroll. Keep document/file state in `DocumentStore` and editor-view state in `DocumentViews`.
- Raw inline HTML is deliberately rendered as text because `?src=` can load cross-origin Markdown. Do not enable HTML rendering without sanitization.
- KaTeX and Mermaid are intentionally dynamically imported only when needed; avoid making them eager dependencies.
- Treat `themes.manifest.json` as authoritative. Add/remove/update packs there, then run `npm run themes:fetch`; do not hand-maintain `public/themes/index.json`. Preserve upstream-relative layouts and include only themes whose licenses permit redistribution; update `THIRD-PARTY.md` when licensing changes. Theme fetching uses the network and mutates committed vendored assets, so it is not a routine verification step.
- Packs marked `"local": true` (currently only `kiwi`) are source: never downloaded, never deleted, but still indexed and reference-checked. Editing them still requires a `themes:fetch` run to regenerate `public/themes/index.json`, and that run touches the network zero times. The reference check reads comments too, so an import rule or asset link written inside one is reported as a missing file.
- Webfonts our own themes use go in `fonts.manifest.json` → `public/fonts/`, not in a theme pack. Prefer families published as `unicode-range` subsets: a CJK family is only affordable because a page downloads the ranges it sets and no more. Vendor only the weights actually used, and check the license permits redistribution as a webfont.

## Verification

Install reproducibly with `npm ci`. There is no separate lint or unit-test script.

Choose the smallest relevant checks:

- TypeScript-only change: `npm run typecheck`.
- Source, build configuration, or production asset change: `npm run build` (typecheck plus Vite build).
- Editor DOM, Markdown parsing/serialization, document behavior, theme compatibility, or compat/bridge CSS: `npm run check:contract`.
- Dev-server/base-path/public-asset behavior: `npm run smoke`.
- Theme visual changes: `npm run themes:shoot`, then inspect `test/shots/`.
- Application chrome/layout changes: `npm run ui:shoot`, then inspect `test/shots-ui/`.

The browser checks and screenshot scripts launch the locally installed Chrome through Playwright (`channel: 'chrome'`); they do not rely on a downloaded Playwright browser. Screenshot commands build first and overwrite ignored output.

Before finishing, run `git diff --check` and inspect the complete diff. Do not commit `dist/` or screenshot output.

---
> Source: [ashawkey/kiwidown](https://github.com/ashawkey/kiwidown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
