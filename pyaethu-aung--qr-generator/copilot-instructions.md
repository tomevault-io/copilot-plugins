## qr-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev             # start dev server
npm run build           # tsc + vite build
npm run lint            # ESLint (type-aware)
npm run lint:fix        # auto-fix lint errors
npm run format          # Prettier check
npm run format:fix      # Prettier write
npm run test            # Vitest (watch mode)
npm run test:coverage   # coverage report (≥85% required)
npm run docker:build    # build production image
npm run docker:run      # run container at http://localhost:8080
```

Run a single test file: `npx vitest run src/utils/share.test.ts`

Before opening any PR, all four must pass locally: `npm run test && npm run lint && npm run build && npm run test:e2e`

Never push directly to `main`. All changes must go through a pull request. A `pre-push` git hook in `.githooks/` enforces this — activated automatically via the `prepare` npm script on `npm install`.

## Architecture

### State flow

`useQRGenerator` owns all QR config state. Input fields update "pending" state (e.g. `inputFgColor`); clicking **Generate** snapshots them into `config`, which drives the `qrcode.react` preview. Downloads use the headless `qrcode` library against the pending input state — not the DOM.

**Capacity guard.** `src/utils/qrCapacity.ts` holds the version-40 byte-mode capacity per EC level and `getCapacityStatus` (measured in UTF-8 bytes). `useQRGenerator` treats content past that capacity like its length validation: `isBlocked` clears the live preview and disables downloads, so `qrcode.create` never sees an unencodable value. This matters because `getMatrixSize` (`qrShapeRenderer.ts`) runs at render and `qrcode.create` throws on over-capacity input — reachable at Q/H levels under the 2000-char input cap — so it catches and falls back to the version-1 size rather than crashing the generator. The `CapacityCounter` under the Text/URL field (text mode only) is the visible signal: `used / max`, amber near the limit, red over it.

### Context providers (wired in `src/main.tsx`)

- `ThemeProvider` — reads/writes `localStorage`, toggles `.dark` on `<html>`, exposes `useThemeContext()`
- `LocaleProvider` — reads/writes `localStorage`, syncs `document.documentElement.lang`, exposes `useLocaleContext()` with `translate(key)` and a locale-aware `seo` object. Locales are additive: register a new JSON in `src/data/i18n/index.ts` and `SupportedLocale`, the `TranslationKey` union, and the registry all widen from it (no type edits). `LocaleMetadata.switchTo` is a `Partial` record, so a new locale needs no edits to existing locale files; `getCopy()` falls back to the default locale for any missing key. The navbar `LanguageToggle` is a native `<select>` dropdown that lists every locale in `localeCodes` by its `locale.name`, so it picks up new locales automatically.

### Directory conventions

| Path | Purpose |
|---|---|
| `src/components/common/` | Reusable primitives (Button, Input, Textarea, Card, etc.) |
| `src/components/feature/qr/` | QR-specific views |
| `src/hooks/` | Stateful hooks and context providers |
| `src/utils/` | Pure helpers — every file here requires a corresponding test |
| `src/data/` | Static config and i18n JSON (`en.json`, `es.json`) |
| `src/types/` | Shared TypeScript types |

### Styling

Tailwind CSS v4 via `@tailwindcss/vite`. Entry point is `src/index.css`. Use semantic design tokens (CSS custom properties) for all colors — never hard-code hex values in component classes. The `dark` class on `<html>` drives dark-mode variants.

### Views

Three top-level views toggle via a `PillGroup` in `src/App.tsx`: **Generate** (`QRGenerator`), **Batch** (`BatchGenerator`), **Scan** (`QRScanner`). Generate stays mounted (`hidden`); Batch and Scan mount on demand.

### Share / export

`useQRShare` handles the share button: tries `navigator.share` with files → `ClipboardItem` image write → download fallback. `useExportState` + `src/utils/export/` drive the hi-res export modal (PNG / SVG / PDF via jspdf).

Headless rendering (no DOM preview) is shared: `renderQrPngBlob` (`src/utils/export/pngRenderer.ts`) for PNG, `exportSvg` / `exportPdf` for the rest. The single-QR download path and batch generation both go through these.

### Batch generation

`BatchGenerator` + `useBatchGenerator` render a pasted list (one value per line, deduped, capped at `BATCH_MAX_LINES`) to PNG/SVG/PDF, or a **Labels** sheet, and pack them into one ZIP via `fflate` (ZIP formats) or a single PDF (Labels). The list can also be populated via **file upload** (`.txt` or `.csv`), either through the **Import** button or by **dragging a file onto the list area** (both routed through the same `processFile` handler and `.txt`/`.csv` validation; drop handlers live on the textarea's wrapper so a drop still lands while the textarea is read-only). A `.txt` file is passed through as-is and a single-column `.csv` extracts that column — both via `parseBatchFile.ts`, with no header detection. A **multi-column `.csv`** instead opens a **column-mapping** UI: `parseBatchCsv.ts` parses the grid (first row = header, RFC-style quoted fields), and the user picks a **content type** plus which column supplies each of its fields, and optionally which column names each output file. Content types are declared in `csvContentTypes.ts` (`CSV_CONTENT_TYPES`): each entry lists its fields (column-mapped text fields, or fixed enum/toggle fields applied to every row) and a `build` that calls the same `buildXString` helper as the single-QR view (Wi-Fi, vCard, email, SMS, tel, geo, vEvent, crypto; `text` encodes one column verbatim). `buildCsvValues` runs every row through `build` (skipping rows that build to '', e.g. a missing required field), and `autoMapColumns` wires fields to columns whose header matches the field key. The built payloads go into `preparedValues` on the hook, the source of truth for the count and `generate` whenever a mapping is active, so a payload that legitimately contains newlines (vCard, iCalendar) is **never** round-tripped through the textarea, where `parseBatchInput` would split it on its own lines into junk codes. `parseBatchInput` splits on newlines for the plain textarea path; `dedupeAndCap` is the shared no-split half that `preparedValues` uses. The filename column builds a `value → filename` map (`filenameOverrides` on the hook) threaded into `buildBatchZip` and applied by `batchFilename` (override slug, no ordinal prefix; falls back to the default name when a value is unmapped or its cell is blank). Overrides apply only to the ZIP formats — the Labels output is a single PDF, so its picker is hidden. Each content type also defines a `caption(get)` (Wi-Fi → SSID, contact → full name, email/sms/tel → recipient/number, geo → `lat,long`, event → summary, crypto → network); `buildCsvValues` returns a `value → caption` map (`captionOverrides`) passed to `buildLabelSheetPdf` as `captionByValue`, so a **Labels** sheet shows a readable caption instead of the raw payload (a value absent from the map, e.g. `text`, falls back to the payload itself). Field labels reuse the existing single-QR `controls.*` i18n keys; only mapping-chrome and the drop hint are new (`batch.csvMap*`, `batch.dropHint`). While a multi-column-CSV mapping is active the textarea is a **read-only source view** of the uploaded file's data rows, comma-separated (header excluded, RFC-quoted via `csvRowsToCommaText` in `parseBatchCsv.ts`); column/type/filename changes update `preparedValues` only and never rewrite it, and a small preview lists the built payloads. The **Clear** button (or importing a `.txt`/single-column `.csv`) drops the mapping and re-enables the textarea for manual entry. Mapping state is component-local, so a tab switch drops it (the raw rows persist as plain textarea text, and then generate as literal comma-joined codes until re-mapped). The core lives in `src/utils/batch/` (`parseBatchInput`, `parseBatchFile`, `parseBatchCsv`, `csvContentTypes`, `batchFilename`, `buildBatchZip`). Labels output uses `labelSheetLayout.ts` for pure-geometry grid maths (page presets, cell placement in PostScript points) and `buildLabelSheetPdf.ts` for the jsPDF render; three Avery-style presets are available (A4·3×7, A4·2×4, Letter·3×6). Each code inherits the user's current design read from `localStorage`: design/frame via `persistedDesign.ts`, foreground/background/EC via `persistedAppearance.ts`. These loaders are the single source of truth, also consumed by `useQRDesign` / `useQRGenerator`, so a batch code matches the live preview. Because the tab mounts on demand it re-reads that design on each open; the pasted list itself is persisted so a tab switch doesn't lose it.

## Testing

Vitest with jsdom. Setup file: `src/setupTests.ts` (imports `@testing-library/jest-dom`). Mock browser APIs (`navigator.share`, `ClipboardItem`) per test file. Coverage threshold: **85%**.

Playwright e2e tests live in `e2e/`. Run with `npm run test:e2e`. Every user-facing feature or fix must have a corresponding e2e spec that proves the scenario works in a real browser. The suite runs across four projects (desktop/mobile × light/dark) and must pass before opening a PR.

## Commit discipline

One logical change per commit. Each of the following is its own commit boundary — do not bundle them:

| Category | Path |
|---|---|
| New/rewritten component | `src/components/` |
| New/rewritten hook | `src/hooks/` |
| New utility module + its test | `src/utils/` |
| New/updated type definition | `src/types/` |
| e2e spec (one scenario group) | `e2e/` |
| i18n key addition (all locales) | `src/data/i18n/` |
| Doc update | `CLAUDE.md`, `README.md`, `DESIGN.md`, `PRODUCT.md` |

If a task touches more than two categories, stage and commit one at a time.

## Skills

Skills are stored under `.agents/skills/` (source files) with symlinks from `.claude/skills/`. Active skills are tracked in `skills-lock.json` (sourced from `pyaethu-aung/skills` on GitHub).

| Skill | When to use |
|---|---|
| `/commit-message` | Creating or amending any git commit |
| `/create-pr` | Opening a GitHub pull request |
| `/update-readme` | After any user-facing change worth documenting |
| `/develop-web-feature` | Building a new feature end-to-end (shape → build → audit → PR) |

Two `PreToolUse` hooks in `.claude/settings.json` enforce that `git commit` and `gh pr create` go through the relevant skills. Do not bypass them with `--no-verify`.

### Permissions (hands-off / autonomous mode)

Auto-approve grants are **personal, not shared**: they live in the gitignored
`.claude/settings.local.json` (where Claude Code also writes "always allow"
approvals), so each developer opts in by running the setup script in their own
checkout. The committed `.claude/settings.json` holds only the enforcement
hooks — project policy everyone shares, never per-developer grants.

> **Workspace trust required.** If you see `Ignoring N permissions.allow entries from .claude/settings.local.json: this workspace has not been trusted`, the allow list is silently inactive. Fix it one of two ways:
> - Run `claude` interactively in this directory once and accept the trust dialog that appears.
> - Or add the entry directly to your personal Claude config (`~/.claude.json`):
>   ```json
>   {
>     "projects": {
>       "/path/to/qr-generator": { "hasTrustDialogAccepted": true }
>     }
>   }
>   ```

`/develop-web-feature` self-configures its required allow entries (into `.claude/settings.local.json`) via a setup script. The only entry you need to add manually (once) is the bootstrap entry for the setup script itself — add it to `.claude/settings.local.json`:

```json
"Bash(node .claude/skills/develop-web-feature/scripts/setup.mjs*)"
```

Then run it from the project root. It **defaults to a dry run** (it prints the grants it would add and writes nothing); re-run with `--write` to apply:

```bash
node .claude/skills/develop-web-feature/scripts/setup.mjs           # preview the delta
node .claude/skills/develop-web-feature/scripts/setup.mjs --write   # apply it
```

This adds the gate runner, the dev-server helper, the skill's other helper scripts, the test/lint/type grants it derives from `package.json`, and the skill-invocation tokens `Skill(commit-message)` / `Skill(create-pr)` (singular; the plural `Skills(...)` never matches) if those skills are installed. It is idempotent, safe to re-run any time. For an unattended run that should not stop on per-edit permission prompts, add `--grant-edits` to also auto-approve `Edit` / `Write` / `MultiEdit`, scoped to the project's source and test directories (config, `package.json`, `.github/`, `.claude/`, and docs still prompt).

The `/impeccable` skill has one separate entry that must be added manually (also to `.claude/settings.local.json`):

| Permission | Why |
|---|---|
| `Bash(node .claude/skills/impeccable/scripts/critique-storage.mjs*)` | `/impeccable` persists critique snapshots via this script; variable expansion (`$SLUG`) triggers Claude Code's obfuscation heuristic without the allow entry. |

## Deployment

- **GitHub Pages**: triggered on push to `main` via `.github/workflows/deploy.yml`
- **Docker image**: published to GHCR on version tags (e.g. `git tag v1.0.0`); Trivy blocks high/critical CVEs
- **Dependabot**: runs daily for npm; no auto-merge

If the hosting URL changes from `pyaethu-aung.github.io`, update the `url` property in `src/components/common/SEOHead.tsx` to keep JSON-LD structured data valid.

---
> Source: [pyaethu-aung/qr-generator](https://github.com/pyaethu-aung/qr-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
