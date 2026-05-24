## icv

> Static site that renders an academic/professional CV from a `cv.json` file produced by [cvkit](https://github.com/AlexanderPico/cvkit). Built with Astro SSG, deploys to GitHub Pages. This is the **render layer** in a three-project architecture:

# icv — Interactive CV

## What This Is

Static site that renders an academic/professional CV from a `cv.json` file produced by [cvkit](https://github.com/AlexanderPico/cvkit). Built with Astro SSG, deploys to GitHub Pages. This is the **render layer** in a three-project architecture:

- **cvkit** (data engine) → produces `cv.json`
- **icv** (this repo, render layer) → reads `cv.json`, renders static site
- **cvkit-app** (GUI shell, not yet built) → SvelteKit web app for non-technical users

These three repos stay separate. icv communicates with cvkit through a single file (`cv.json`) and a config pointer (`cvkit.config.json`).

## Owner

Alex Pico (AlexanderPico) — retired biomedical researcher and software engineer. ORCID: 0000-0001-5944-9960.

## Current State (2026-03-22)

**All 8 components are wired to live cv.json data.** The site builds and renders from real data fetched via cvkit (ORCID, Semantic Scholar, GitHub).

### What's Built
- **Data layer**: `src/utils/loadCVData.js` — cached loader with typed slice exports (getBasics, getWork, getEducation, getPublications, getSoftware, getSkills, getCitationMetrics, getPublicationsByYear, getCoauthors)
- **Config**: `src/utils/getConfig.js` — reads `icv.config.yaml` for tab layout, theme, sources
- **8 components**, all consuming live CVData:
  - `ProfileCard` → cv.basics (name, title, summary, location, profile links)
  - `QuickStats` → citationMetrics + derived counts (h-index, citations, pubs, repos, stars)
  - `ExperienceTimeline` → cv.work + cv.education merged into a vertical timeline
  - `PublicationsTable` → cv.publications with DOI links, authors, citation counts
  - `SkillsRadar` → cv.skills as categorized tags with level badges
  - `CitationChart` → publications + citations per year (Chart.js combo chart, loaded client-side via CDN)
  - `GithubHeatmap` → cv.software as a repo card grid (stars, forks, language)
  - `CoauthorNetwork` → derived co-author frequency as horizontal bar chart
- **Dynamic routing**: `src/pages/[tab].astro` generates routes from `icv.config.yaml` tabs
- **Tab navigation**: configurable via YAML (Overview, Experience, Publications, Stats)

### How cv.json Is Loaded (Option A — local file)
`loadCVData.js` resolves the cv.json path in this order:
1. `cvDataPath` field in `cvkit.config.json` at repo root
2. `./cv.json` at repo root (default)

The file is read once per Astro build (Node.js `fs.readFileSync`), cached in-process. All component imports share the cached result.

## Tech Stack
- **Framework**: Astro 3.x (SSG mode)
- **Charts**: Chart.js 4.x (loaded from CDN at runtime)
- **Config**: js-yaml for `icv.config.yaml`
- **Schema types**: `@cvkit/schema` (linked as file dependency from sibling repo during dev)
- **No CSS framework** — component-scoped `<style>` blocks with CSS custom properties

## Key Files
```
icv/
├── cv.json                  # Live data from cvkit fetch (committed)
├── cvkit.config.json        # Points to cv.json location
├── icv.config.yaml          # Tab layout, theme, source IDs
├── astro.config.mjs         # BASE_PATH env var for GitHub Pages
├── src/
│   ├── components/
│   │   ├── ProfileCard.astro
│   │   ├── QuickStats.astro
│   │   ├── ExperienceTimeline.astro
│   │   ├── PublicationsTable.astro
│   │   ├── SkillsRadar.astro
│   │   ├── CitationChart.astro
│   │   ├── GithubHeatmap.astro
│   │   ├── CoauthorNetwork.astro
│   │   └── index.js          # Barrel export (snake_case keys match YAML)
│   ├── pages/
│   │   ├── index.astro        # Redirects to first tab
│   │   └── [tab].astro        # Dynamic route per YAML tab
│   └── utils/
│       ├── loadCVData.js      # cv.json loader + typed slice exports
│       └── getConfig.js       # icv.config.yaml reader
```

## Branches
- `main` — active development
- `origin/m1-dev` — older branch (June–Dec 2025) with LinkedIn PDF parse fixtures, not merged

## Development
```bash
pnpm install
pnpm dev          # http://localhost:4321/
pnpm build        # generates dist/
```

## Refresh Data
```bash
# In the cvkit repo:
cvkit fetch --orcid 0000-0001-5944-9960 --github AlexanderPico
cvkit build --format json --output ../icv/cv.json
# Then rebuild icv
```

## GitHub Pages Deploy
A GitHub Action exists (see commit history for GH Actions setup). Uses `BASE_PATH` env var for path prefix. The action builds Astro and deploys the `dist/` folder.

## What's Next

### Immediate Improvements
- **Styling**: Components have functional CSS but no unified design system. Consider adding CSS custom property theme (light/dark) based on `icv.config.yaml` `theme` field.
- **Responsive**: Basic grid layouts exist but haven't been tested on mobile.
- **GithubHeatmap**: Currently a repo card grid. A real contribution heatmap would need GitHub GraphQL API at runtime or a pre-built data artifact.
- **CoauthorNetwork**: Currently a bar chart. A force-directed graph (d3) is a natural upgrade.
- **SkillsRadar**: Tag layout for now. With >5 skill categories a Chart.js radar chart would work.

### Architecture Decisions Already Made
- icv stays a **static site** — no server, no auth, no data fetching at runtime
- All data comes from `cv.json` at build time
- The GUI for collecting user credentials (ORCID, GitHub, LinkedIn PDF) belongs in **cvkit-app**, not here
- Stack for cvkit-app: SvelteKit + Cloudflare Pages + Workers Unbound + KV + GitHub fork as canonical store

### Dependency Note
`@cvkit/schema` is currently linked as `file:../cvkit/packages/schema` in package.json. This works for local dev when both repos are siblings. Once cvkit is published to npm, switch to the npm version.

## Communication
- Slack workspace: nucleus-ts26071
- Channel: #icv (C0AN1C7T3CJ)
- Related: #cvkit (C0AMY01JBS9)

---
> Source: [AlexanderPico/icv](https://github.com/AlexanderPico/icv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
