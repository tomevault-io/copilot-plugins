## gracew-portf

> Grace Williams' personal creative marketing portfolio. The source content is a

# Portfolio Website — Claude Code Handoff

## Project Overview

Grace Williams' personal creative marketing portfolio. The source content is a
designed slide deck (9 images in `assets/Creative Portfolio/`, 1920x1080 each,
exported from Canva) covering a cover, contents/nav, bio, 4 project case studies,
and a resume. The site rebuilds this deck as a real, navigable website:
- The cover + contents slides become a real HTML/CSS homepage (not images)
- The bio, project, and resume slides are kept as Grace's designed images,
  untouched — but with invisible clickable "hotspots" added on top so the
  text she already designed (HOME, CONTENTS, EMAIL, LINKEDIN, etc.) actually
  navigates the site
- Be deployed to Netlify; custom domain to be added later (Grace doesn't have
  one yet)

---

## Current Status

- [x] Claude Code installed
- [x] Project folder created
- [x] Site scaffolded (homepage + about + 4 project pages + resume)
- [x] Images added (`images/`, copied from `assets/Creative Portfolio/`)
- [x] Hotspot navigation added to image-based pages (HOME, CONTENTS, EMAIL, LINKEDIN)
- [ ] Bio written (currently using slide image as-is)
- [x] Calibrate hotspot positions precisely (HOME/CONTENTS were sitting above the
      drawn ovals on about, resume, and all 4 project pages — moved down to
      overlap the actual text; EMAIL/LINKEDIN on about were already correct)
- [ ] Decide what project tags (Copywriting, Art Direction, etc.) should link to, if anything
- [x] Push to GitHub (`gracew752-afk/gracew-portf`, authenticated via `gh auth login`)
- [x] Deployed to Netlify (project "gracew.co", connected to GitHub for auto-deploy on push)
- [x] Custom domain connected (`gracew.co` / `www.gracew.co`, SSL enabled via Let's Encrypt)

---

## File Map

```
gracew-portf/
├── index.html              # Homepage — real HTML/CSS recreation of slides 1+2
│                            #   (cover hero + Contents nav list)
├── about.html               # Slide 3 image + hotspots (HOME, CONTENTS, EMAIL, LINKEDIN)
├── resume.html               # Slide 8 image + hotspots (HOME, CONTENTS)
├── projects/
│   ├── sprite.html           # Slide 4 — Sprite "Who's Mid?!" campaign
│   ├── glazed-craze.html      # Slide 5 — Glazed Craze donut shop concept
│   ├── pinkcore.html          # Slide 6 — PinkCore Pilates brand
│   └── tony-rocks.html        # Slide 7 — Tony Rocks Hair redesign
├── booklab/index.html         # The Book Lab — self-hosted demo app for the Gumroad
│                              #   product (type a book → generate a 10-week course).
│                              #   Demo shelf: Atomic Habits, The Psychology of Money,
│                              #   Educated (full courses); any other title runs the
│                              #   "seminar edition" template. ?book=X deep-links,
│                              #   ?g=slow|fast paces the generate animation for filming.
├── css/style.css
├── js/main.js                # ?debug=1 hotspot calibration mode
├── images/                    # Renamed copies of the slide JPGs used on each page
└── assets/Creative Portfolio/ # Original 9 slide exports — source of truth, don't edit
```

Slide 9 (Bella Matthews graphic) is unused — appears unrelated to this portfolio.

---

## House Style / Rules

- **Don't alter Grace's designed slide images** (about, project, resume pages).
  Add interactivity via transparent `.hotspot` overlay links instead — see
  WORKFLOW GUIDE.md.
- Homepage is the one page built from scratch in HTML/CSS (dark bg `#121212`,
  pink accent `#f3c9dd`), since it merges two slides into one page.
- Plain HTML, CSS, JS — no framework, no build step.

---

## Deployment (after site is ready)

1. Go to [netlify.com](https://netlify.com) and create a free account
2. Push this repo to GitHub, connect it to Netlify for auto-deploy
3. Your site goes live instantly at a `.netlify.app` URL
4. When Grace buys a domain (~$10–15/year, e.g. Namecheap): Netlify → Domain
   Settings → add custom domain → update nameservers → Netlify handles SSL
   automatically

---

## Context from Planning Session

- Mason is teaching Grace this workflow on her computer; Grace will continue solo
- Grace doesn't have a domain yet — site can go fully live on `.netlify.app`
  first, domain added whenever she's ready
- Local preview runs at `http://127.0.0.1:8080` (see WORKFLOW GUIDE.md)

---
> Source: [gracew752-afk/gracew-portf](https://github.com/gracew752-afk/gracew-portf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
