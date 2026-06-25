## controls

> Guidance for working in the **Shiny Controls** repo (`Shiny.Maui.Controls` + `Shiny.Blazor.Controls` and their add-on packages).

# CLAUDE.md

Guidance for working in the **Shiny Controls** repo (`Shiny.Maui.Controls` + `Shiny.Blazor.Controls` and their add-on packages).

## Repo layout

- `src/` — one project per package per host (e.g. `Shiny.Maui.Controls`, `Shiny.Blazor.Controls`, plus add-ons: `*.Camera`, `*.Barcodes`, `*.Markdown`, `*.MermaidDiagrams`, `*.Desktop`, `*.Kiosk`, camera analyzers `Camera.Barcode/Documents/Face/Motion/Ocr`, themes `Themes.Material/Ocean`).
- `samples/Sample/` — the MAUI + Blazor demo app. Each control has a feature page under `samples/Sample/Features/<Area>/`, wired into `AppShell.xaml` and `MauiProgram.cs`.
- `tests/` — unit tests.
- `SKILLS/shiny-controls/` — the **local skill** (`SKILL.md` + one markdown file per control) that teaches code generation for these controls.
- `README.md` — the package-level overview (top-of-file summary paragraph + per-control sections + NuGet badges).
- `themes/` — M3 theme pack seeds.

## Documentation site

The public docs live in a **separate repo**: `~/Desktop/dev/documentation` (Astro / Starlight).

- Controls docs: `src/content/docs/controls/<control>/`
- Controls release notes: `src/content/docs/controls/release-notes.mdx`
- Main menu (sidebar): `src/sidebar-topics.mjs` (the `Controls` topic, ~line 289). The homepage menu (`HomepageNav.astro`) is **auto-generated from this file** — no separate edit needed.
- Homepage: `src/content/docs/index.mdx` — the **"UI Controls"** `<Card>` (~line 149) lists every control grouped by category (Flagship / Layout & Overlays / Input / Display & Media / Status & Feedback / Desktop).

## Required updates for EVERY fix & feature

With each fix and each new feature, update all of the following so they stay in sync:

1. **README.md** — reflect new/changed behavior; add a NuGet badge + section if it's a new package.
2. **Local skill** (`SKILLS/shiny-controls/`) — update the relevant control's `.md` (or add a new one and reference it in `SKILL.md`) so generated code matches.
3. **Shiny docs** (`~/Desktop/dev/documentation`):
   - **Release notes** — add an entry to `src/content/docs/controls/release-notes.mdx`.
   - **Menu** — for a **new feature**, add the new menu node(s) under the `Controls` topic in `src/sidebar-topics.mjs`. (The homepage menu updates automatically from this.)

### Additionally, if the control itself is NEW

4. Add its docs folder under `src/content/docs/controls/<control>/`.
5. Add it to the **homepage section** — the "UI Controls" `<Card>` in `src/content/docs/index.mdx` (place it in the appropriate category group).
6. Add its top-level node to the **main menu** (`src/sidebar-topics.mjs`) under the `Controls` topic — which also surfaces it in the homepage menu.
7. **Leave a screenshot TODO** — do **not** capture screenshots as part of the feature/release work. Instead, after wiring up a new component, note a TODO (e.g. `TODO: capture screenshots for <control>`) so they can be done later on request. See **Screenshots** below.

## Screenshots (on request)

Capturing control screenshots is a **separate, on-request task** — never run it automatically as part of a fix, feature, or release. When a new control ships, just leave a TODO (see step 7 above). Only capture when the user explicitly asks.

When asked to capture screenshots for a control:

1. **MAUI** — drive the demo app (`samples/Sample/`) to the control's feature page (`samples/Sample/Features/<Area>/`) and capture the screenshot with **mauidevflow**.
2. **Blazor** — drive the Blazor sample to the matching page and capture the screenshot with **Playwright**.
3. **Add them to the docs repo** (`~/Desktop/dev/documentation`):
   - Save images under `public/images/<control>/`, named `s1.png`, `s2.png`, … (one folder per control, sequential names).
   - Reference them from the control's `.mdx` (e.g. `src/content/docs/controls/<control>/index.mdx`) with `<img src="/images/<control>/s1.png" width="220" alt="<Control>" />`.

## Blog posts (only when explicitly requested)

Do **not** write blog posts automatically as part of a fix/feature. Write them **only when the user asks**. When asked to blog a feature, produce **two** posts — first the docs-site version, then adapt it for the personal blog.

### 1. Docs site — `~/Desktop/dev/documentation`

- File: `src/content/docs/blog/YYYY/MM/<slug>.mdx` (current year/month folders; create the month folder if needed).
- Frontmatter:
  ```yaml
  ---
  title: '...'
  description: '...'
  date: YYYY-MM-DD
  authors:
    - allanritchie
  tags:
    - Release        # or Feature, AI, etc.
  ---
  ```
- Body is MDX. Reuse components where relevant, e.g. `import NugetBadge from '/src/components/NugetBadge.astro';` then `<NugetBadge name="Shiny.Maui.Controls.Xxx" />`.
- Voice: product/release-note tone — what shipped, breaking changes, code samples, how to use it. **No hero image** on this site.

### 2. Personal blog — `~/Desktop/dev/blog` (adapt the docs post)

- File: `src/content/blog/YYYY/MM/<slug>.mdx` (note: `content/blog`, not `content/docs/blog`).
- Frontmatter (different schema — see `src/content.config.ts`):
  ```yaml
  ---
  title: '...'
  description: '...'
  pubDate: 'Mon DD YYYY'                          # e.g. 'Jun 15 2026'
  heroImage: '../../../../assets/<slug>-hero.svg'
  tags: ['Shiny', '.NET MAUI', 'Controls']
  ---
  ```
- Voice: rework the docs post into a personal, first-person narrative ("Here's something that shouldn't be hard but is…", "So I built…") — story/motivation up front, not a dry changelog.
- **Hero image is required.** Create `src/assets/<slug>-hero.svg`:
  - SVG, `viewBox="0 0 1200 630"`, `width="1200" height="630"`.
  - Match the house style: dark navy/indigo gradient background (`#0f172a` → `#1e1b4b`), cyan/green/violet accent gradients, subtle glow filters, the feature name as the headline. Crib an existing one (e.g. `datasync-hero.svg`, `documentdb-orleans-hero.svg`) as a starting template.

## Conventions

- Keep MAUI and Blazor at feature parity where the platform allows; note platform-only features explicitly (Desktop is MAUI-only; SheetView/Kiosk are Blazor-specific, etc.).
- Add/update a demo page in `samples/Sample/Features/` for any new control or notable feature.
- Build with `dotnet build Build.slnf`.

---
> Source: [shinyorg/controls](https://github.com/shinyorg/controls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
