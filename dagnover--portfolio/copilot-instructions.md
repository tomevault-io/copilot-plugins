## portfolio

> Static personal portfolio website — pure vanilla HTML/CSS/JS. No build step, no package manager, no Node, no TypeScript. Content is in Spanish.

# AGENTS.md

## What this repo is

Static personal portfolio website — pure vanilla HTML/CSS/JS. No build step, no package manager, no Node, no TypeScript. Content is in Spanish.

## Running locally

```bash
python -m http.server 8080
# open http://localhost:8080
```

VS Code Live Server extension also works. There is no `npm install`, no `npm run dev`, nothing to build.

## Architecture

```
index.html          single-page shell; all sections are in-page anchors
css/styles.css      all styling (969 lines); uses CSS custom properties
js/main.js          entry point — runs on DOMContentLoaded, calls all render*() functions
js/contact.js       EmailJS modal logic
js/data/            data layer: projects.js, experience.js, education.js, skills.js
assets/img/         profile.jpg
```

- Data files expose globals on `window` (`window.projects`, `window.experience`, etc.).
- `main.js` reads those globals and injects HTML strings into DOM containers.
- No module system — script load order in `index.html` matters (data files before `main.js`).

## EmailJS setup (not yet configured)

`js/contact.js` contains three placeholder strings that must be replaced before the contact form works:

```js
YOUR_PUBLIC_KEY
YOUR_SERVICE_ID
YOUR_TEMPLATE_ID
```

See `EMAILJS_SETUP.md` for instructions. Do **not** commit real keys.

## Style conventions

- CSS custom properties are defined at `:root` (`--bg`, `--accent: #1e88ff`, `--card`, `--border`, etc.).
- Default theme is dark; light mode overrides via `prefers-color-scheme: light`.
- Mobile-first; breakpoints at 840px, 768px, 640px, 480px.
- Accessibility: `.skip-link`, `aria-*` attributes, `prefers-reduced-motion` support — keep these intact.

## Deployment

Documented options: GitHub Pages or AWS S3 + CloudFront + ACM. No CI/CD is configured.

## No tests, no linting

There are no test suites, no ESLint, no Prettier, no pre-commit hooks. Manual browser testing is the only verification step.

---
> Source: [Dagnover/portfolio](https://github.com/Dagnover/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
