## yandere-masonry

> Userscript for masonry/waterfall browsing on booru-style image sites.

# AGENTS.md - Yande.re Masonry

## Project Summary

Userscript for masonry/waterfall browsing on booru-style image sites.
The app runs as a userscript bundle built by Vite and mounted into the target page.

## Stack

- Vue 2.7 with `@vitejs/plugin-vue2`
- Composition-style SFCs with `<script setup>`
- Vuetify 2
- TypeScript
- pnpm workspace packages under `packages/`

Important: this is not a Vue 3 app. Keep compatibility with Vue 2.7 and the current Vuetify 2 patterns.

## Dev Commands

```bash
pnpm install
pnpm run dev      # Vite dev server, serves /_development.user.js
pnpm run build    # Production userscript build
pnpm run lint     # ESLint for src/**/*.ts and src/**/*.vue
pnpm run release  # Release script
```

## Development Flow

1. Run `pnpm install`
2. Run `pnpm run dev`
3. Open `http://127.0.0.1:3000/_development.user.js`
4. Install/update the userscript in Tampermonkey or Violentmonkey
5. Open a supported target site with `_wf=1` flow enabled, for example `https://yande.re/post?_wf=1`

## Build Output

- Main output: `dist/yandere-masonry.user.js`
- `package.json#main` points to the userscript bundle
- External globals are injected by the userscript Vite plugin: `vue`, `vuetify`, `vue-i18n`, `fast-xml-parser`

## Code Style

- 2 space indent
- no semicolons
- single quotes
- prefer explicit TypeScript types when the shape is not obvious
- use `@/` for imports from `src/`
- keep comments sparse and only for non-obvious behavior

## Architecture

- `src/prepare.ts`
  Initializes the userscript before the Vue app mounts. Also handles early page setup, style injection, locale/tag bootstrap, and entry conditions.
- `src/App.vue`
  Root app shell.
- `src/components/`
  Main UI pieces such as app bar, settings drawer, post list, detail modal, nav drawer, snackbar.
- `src/store/`
  App-wide state via `Vue.observable`.
- `src/store/actions/`
  Loading, site dispatch, and detail-fetch side effects.
- `src/api/`
  Site adapters and support checks.
- `src/utils/`
  Shared helpers, including download, formatting, image, and browser integration helpers.
- `packages/`
  Local workspace packages such as booru client, masonry layouts, userscript plugin, and release helpers.

## Important Files

- `vite.config.ts`
  Vite config, Vue 2 plugin, userscript plugin, alias setup.
- `src/prepare.ts`
  Earliest userscript bootstrap.
- `src/components/PostList.vue`
  Main list rendering and quick actions.
- `src/components/PostDetail.vue`
  Detail modal, single-image actions, preload behavior.
- `src/components/AppBar.vue`
  Batch download UI, export links, page navigation, search.
- `src/components/SettingsDrawer.vue`
  User-configurable behavior and feature toggles.
- `src/api/booru.ts`
  Shared booru support checks and partial-support gating.
- `src/utils/download-source.ts`
  Centralized preferred download-source resolution for sample/jpeg/original fallback.

## Supported Sites

Current adapters exported from `src/api/index.ts` include:

- `moebooru`
- `booruAction` for booru-client-backed sites
- `gelbooru`
- `rule34`
- `r34paheal`
- `zerochan`
- `animepictures`
- `allgirl`
- `hentaibooru`
- `kusowanka`
- `nozomila`
- `sankaku`
- `realbooru`
- `rule34hentai`
- `eshuushuu`
- `anihonetwallpaper`

Do not assume all adapters have the same feature surface.

## Support Tiers

There is an existing distinction between sites with fuller support and `notPartialSupportSite === false` in `src/api/booru.ts`.
That flag currently controls whether some download/list actions are shown at all.

When adding features, check whether they should:

- apply only to full-support sites
- apply to all sites with adapter data available
- or be gated by a more specific capability check

## Download Behavior

Download behavior is layered and not uniform across sites.

- Single-image detail download already supports multiple sources when available: `sampleUrl`, `jpegUrl`, `fileUrl`
- Batch download and quick-download now share centralized source resolution in `src/utils/download-source.ts`
- Preferred download source is stored in `settings.downloadUrlKey`
- Sample-download support is currently gated by `isSampleDownloadSupportedSite()` in `src/api/booru.ts`
- If a site does not show the download UI, the related download-source setting should also stay hidden

When editing download logic, update all three paths together:

- `src/components/PostDetail.vue`
- `src/components/AppBar.vue`
- `src/components/PostList.vue`

Also be careful with detail hydration in `src/store/actions/detail.ts`; `sampleUrl` and `fileUrl` must not be mixed up.

## State And Persistence

- Global settings live in `src/store/settings.ts`
- Runtime store lives in `src/store/index.ts`
- Settings persist through `localStorage` via the watcher in `src/App.vue`

If you add a new setting:

1. add the default in `src/store/settings.ts`
2. expose it in `src/components/SettingsDrawer.vue` if user-facing
3. add locale strings in all shipped locale files when needed
4. verify that unsupported sites do not expose broken controls

## Localization

Locale files live in:

- `src/locales/zh-Hans.json`
- `src/locales/zh-Hant.json`
- `src/locales/ja.json`
- `src/locales/en.json`

When adding visible UI text, update all four unless there is a strong reason not to.

## Verification

Primary verification available in-repo:

```bash
pnpm run lint
```

There is currently no standard automated test suite documented in the repo.
For UI or site-adapter changes, manual verification in the browser/userscript environment is usually required.

## Practical Notes For Agents

- Prefer `rg` for code search
- Do not rewrite large locale files or generated data unnecessarily
- Be careful around host-specific behavior and partial-support gates
- Before changing download UX, inspect both list and detail flows
- Preserve existing userscript assumptions such as Tampermonkey APIs and browser-side execution

---
> Source: [asadahimeka/yandere-masonry](https://github.com/asadahimeka/yandere-masonry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
