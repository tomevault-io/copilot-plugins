## portfolio-pranavmishra-paranoid

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

The entire app lives in `portfolio/` (Create React App). The repo root only holds README/LICENSE/preview assets — all commands run from inside `portfolio/`.

```
portfolio/
  src/
    App.js                    # Router only: "/" → MainPortfolio, "/resume" → ResumeViewer
    MainPortfolio.js          # The single-page layout — wires every section together
    data/                     # ALL editable content lives here (see below)
    components/               # All UI pieces
    styles/About.css          # About-section styles (one-off)
  public/
    assets/images/{ai_ml,game_design,misc,companies,achievements,icons,default}/
    resumes/{ai,game}/        # Drop PDFs here; resumeUtils.js probes common filenames
```

## Commands

Run everything from `portfolio/`:

- `npm start` — dev server on http://localhost:3000 (prestart auto-runs `resume-manifest`)
- `npm run build` — production build (uses `CI=false` so warnings don't fail the build; bash-only — on Windows CMD use `set CI=false && npx react-scripts build`)
- `npm run resume-manifest` — scan `public/resumes/{ai,game}/` for PDFs and write `manifest.json`; auto-runs via `prestart` / `prebuild`
- `npm test` — Jest via react-scripts
- `npm run check` — `npm install && npm run build` (CI sanity check used in RUN_LOCAL.md)

Stack: React 19, react-router-dom 7, framer-motion 12, react-scripts 5, plain CSS (no Tailwind, no TS).

## Where each piece of content lives — the only files you usually edit

All site content is data-driven from three files in [portfolio/src/data/](portfolio/src/data/). You almost never need to touch components to add content.

### Projects — [portfolio/src/data/projects.js](portfolio/src/data/projects.js)

One `projects` object with three arrays, rendered in this order on the page: `aiMl`, then research (publications), then `gameDesign`, then `misc`. Each project shape:

```js
{
  id: "unique-kebab-id",
  title, category, description,
  mainImage: "assets/images/<bucket>/foo.png",   // path RELATIVE to public/, no leading slash
  gallery: ["assets/images/<bucket>/...", ...],  // [] is fine
  techStack: ["..."],
  githubLink, demoLink, websiteLink              // empty string "" hides the button
}
```

To **add a project**: drop the image into `public/assets/images/{ai_ml|game_design|misc}/`, append an object to the right array. To **remove**: delete the entry. `getImageWithFallback` at the bottom of the file handles missing/empty image paths by substituting a category default from `public/assets/images/default/`.

Also in this file: `contactInfo` (name, title, bio, emails, social URLs, resume paths) and `socialIcons` (icon image refs).

### Experience — [portfolio/src/data/experience.js](portfolio/src/data/experience.js)

Flat `experiences` array. Rendered in [ExperienceModal.js](portfolio/src/components/ExperienceModal.js), opened by the "Experience" tab in [CollapsibleSectionTabs.js](portfolio/src/components/CollapsibleSectionTabs.js) (see `onExperienceToggle` wiring in [MainPortfolio.js:291](portfolio/src/MainPortfolio.js#L291)). Each entry has `company`, `title`, `duration`, `location`, `companyLogo` (path under `public/assets/images/companies/`), `description` (bullet array), `techStack`, `links`.

### Publications / Research — [portfolio/src/data/publications.js](portfolio/src/data/publications.js)

`publications` array + a derived `publicationStats` object. Rendered by [PublicationsSection.js](portfolio/src/components/PublicationsSection.js) under section id `research`. Each entry has full abstract, `status` ("ACCEPTED" | "Under Review" | "Under Preparation"), `doi`, `pdfLink`, `codeLink`, `citationCount`, `techStack`, `tags`, `keywords`. Stats (total publications, total citations) auto-recompute from the array — no manual counter to update.

## Page architecture — what renders what

[MainPortfolio.js](portfolio/src/MainPortfolio.js) is the whole page. It owns three pieces of state worth knowing:

- `activeSection` — driven by a scroll listener that registers each `<section>`'s offset via `sectionsRef`; controls nav highlight.
- `connectSlateOpen` — toggled by the **ConnectButton** floating next to the profile image and by every "Connect" CTA. Renders [ConnectSlate.js](portfolio/src/components/ConnectSlate.js) inline beneath the profile pic. The hardcoded social URLs (GitHub/LinkedIn/Resume/YouTube) live in the `BUTTONS` array at the top of that file — NOT in `contactInfo`.
- `experienceModalOpen` — opened from the collapsible Experience tab, renders [ExperienceModal.js](portfolio/src/components/ExperienceModal.js).

Section order in [MainPortfolio.js](portfolio/src/MainPortfolio.js): About → CollapsibleSectionTabs (Experience/Publications launchers) → ProjectTabs (sticky nav) → AI/ML → Research (publications) → Game Design → Misc → MinimalContactBelt.

## The visual gimmicks — where they live

- **Particle background**: [ParticleBackground.js](portfolio/src/components/ParticleBackground.js). Pure canvas, mounted only inside the About section. Particle count, mouse explosion radius, colors, and mobile downscaling (40 vs 120 particles) are all constants in the `useEffect` body — edit there.
- **Connect button + slate**: [ConnectSlate.js](portfolio/src/components/ConnectSlate.js) + [ConnectSlate.css](portfolio/src/components/ConnectSlate.css) export both `ConnectSlate` (the panel) and `ConnectButton` (the floating toggle). Positioned via the `.profile-img-wrapper` `position:relative` in [MainPortfolio.js:181](portfolio/src/MainPortfolio.js#L181). To add/remove a social link, edit the `BUTTONS` array.
- **Trophy pop-ups / achievements**: [TrophyButton.js](portfolio/src/components/TrophyButton.js). Achievements are a hardcoded `achievements` array inside the component (not in `data/`) — each entry has `icon`, `title`, `description`, `image`, `link`. Hint pulse / auto-hide timing are constants in the component's `useEffect`. Achievement images live in `public/assets/images/achievements/`.
- **Theme switch (dark/light)**: [ThemeSwitch.js](portfolio/src/components/ThemeSwitch.js).
- **Three section themes** (game red / AI blue / misc green): CSS variables in [src/index.css](portfolio/src/index.css) (`--game-primary`, `--ai-primary`, `--misc-primary`, etc.). The `PixelText` component is used only for the Game Design heading.

## Routing and resume

[App.js](portfolio/src/App.js) defines two routes:
- `/` → `MainPortfolio`
- `/resume` → [ResumeViewer.js](portfolio/src/components/ResumeViewer.js)

**Resume source of truth is external:** the live PDF lives at [`PranavMishra17/PranavMishra17`](https://github.com/PranavMishra17/PranavMishra17/blob/main/RESUME%20Pranav_Mishra.pdf) on GitHub. [ResumeViewer.js](portfolio/src/components/ResumeViewer.js) embeds it via `raw.githubusercontent.com` and HEAD-probes it on mount; if unreachable, it auto-switches to a local backup.

**Local backup auto-discovery:** drop any `.pdf` file into `public/resumes/ai/` or `public/resumes/game/` — no specific filename required. [scripts/generate-resume-manifest.js](portfolio/scripts/generate-resume-manifest.js) runs via the `prestart` and `prebuild` npm hooks, picks the most recently-modified `.pdf` in each folder, and writes `public/resumes/manifest.json`. ResumeViewer fetches that manifest at runtime. The old [resumeUtils.js](portfolio/src/utils/resumeUtils.js) (filename probing) is now unused.

## Image paths — the one gotcha

Inside `data/projects.js`, `mainImage` and `gallery` paths are written **without** a leading slash (e.g. `"assets/images/ai_ml/foo.png"`). `getImageWithFallback` in the same file prepends `/` before returning. Other data files (experience, publications, social icons) use leading-slash paths directly. Match whichever style the surrounding entries use.

## Deployment

Configured for Vercel ([vercel.json](portfolio/vercel.json) only silences the GitHub integration). Build output is `portfolio/build/` from `npm run build`. The repo also includes a committed `build/` directory — don't edit it by hand, it's regenerated.

---
> Source: [PranavMishra17/Portfolio-PranavMishra-Paranoid](https://github.com/PranavMishra17/Portfolio-PranavMishra-Paranoid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
