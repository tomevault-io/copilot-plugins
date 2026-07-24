## dvc-org

> Documentation site for DVC (Data Version Control), built with Gatsby.

# DVC.org

Documentation site for DVC (Data Version Control), built with Gatsby.

## Stack

React 19, Gatsby, CSS Modules, Tailwind, Webpack. Theme in
`packages/gatsby-theme/`, docs content in `content/docs/`, components and
plugins in `src/` and `plugins/`. `server/` is an Express production server that
adds redirects, Plausible analytics proxy, Helmet security headers, and
cache-control rules on top of the Gatsby static build.

Docs served at https://doc.dvc.org (Heroku + CloudFront CDN).

- `src/components/Link/index.tsx` has a static DVC `VERSION` that needs periodic
  updates.
- `redirects-list.json` — URL redirect rules, used by the Express server.
- `content/docs/sidebar.json` — left sidebar navigation structure.
- `content/linked-terms.js` — defines which backtick terms auto-link (e.g.
  `dvc.yaml` → project structure page, `dvc push` → command reference).
- `content/docs/user-guide/basic-concepts/` — glossary source files for `<abbr>`
  tooltip content (DVC project, workspace, etc.).

## Commands

**!!Important: Use `yarn` not `npm`**.

- `yarn` — install dependencies
- `yarn develop` — dev server with hot reload (localhost:8000)
- `yarn clean` — clean Gatsby cache and build artifacts (used before starting
  server if Gatsby plugins were updated before using `yarn develop`)
- `yarn build` — production build to `public/`
- `yarn start` — production server over `public/`
- `yarn test` — run tests using vitest
- `yarn format <FILE>` — format with Prettier
- `yarn format-all` / `yarn format-staged` / `yarn format-check-all`
- `yarn lint` / `yarn lint-fix` / `yarn lint-ts` / `yarn lint-css`
- `yarn fix-all` — run all fixers at once

## Writing docs

See `content/docs/contributing/docs.md` for the full style guide.

### Voice and tone

- Clear, human, approachable — not authoritative or overly formal.
- Familiar language over jargon ("commit" not "revision", "version" not
  "reference") when accurate enough.
- Tutorials/getting-started: direct address, contractions, light encouragement.
- Command/API reference: systematic, scan-oriented, but still explanatory when
  examples need it.
- Lead with the essence, then layer in clarifications and edge cases.
- Details/admonitions are a standard editorial pattern, not an exceptional
  device.
- Bullet lists max 5-7 items. Bullet text max ~3 sentences. Full-sentence
  bullets start capitalized and end with a period; fragments can be lowercase
  with no ending punctuation.
- Emojis sparse and purposeful.

### Markup conventions

- One `.md` file per page in `content/docs/`.
- Images in `static/img/`. Navigation in `content/docs/sidebar.json`.
- `dvc <command>` and `dvc.api.<method>()` in backticks auto-link.
- `dvc.yaml`, `.dvc`, `dvc.lock` in backticks also auto-link.
- `<abbr>` tags for glossary terms — powers tooltip popups.
- `<admon type="info">` clarifications, `type="tip"` best practices,
  `type="warn"` data safety (sparingly).
- `<details>` for setup steps, deep dives, optional complexity.
- Code fences: `usage` for `dvc --help` output, `dvc` for terminal examples,
  `yaml` for DVC/YAML files, `dvctable` for colored table cells, `diff` for git
  diff output.
- **Bold** for emphasis, _italics_ for special terms.
- Emojis: 📖 related docs, ⚠️ warnings, 💡 tips.
- 80-char line width. Prettier enforced via pre-commit hook.

## Design

Audience: ML engineers, data engineers, and developers who need fast lookup and
comfortable long-session reading.

- Pragmatic, precise, trustworthy. Not marketing, not dashboard.
- Documentation-first editorial UI. Light mode is the baseline; dark mode is a
  calm extension, not a re-skin.
- Surfaces separate clearly without looking glossy, heavy, or over-cardified.
- Accent color (cyan) concentrated in links and active states, not spread as
  decoration.

### Typography

- Reading comfort over stylistic expression.
- Hierarchy through spacing, weight, and contrast — not size jumps.
- On reference pages, typography does the heavy lifting: monospace rhythm,
  weight, indentation, and alignment establish structure before borders or
  fills.

### Reference pages

- Command and API docs are dense lookup tools. Optimize for scan speed and
  stable structure.
- Quieter than tutorials — no decorative surfaces or oversized gestures.
- Emphasis from typography and grouping first; box treatments only when they
  materially improve comprehension.
- Tutorials may allow more surface treatment and guidance cues; reference pages
  stay tighter, flatter, and more systematic.

### Principles

1. **Content first.** Reading flow and scanability outrank decoration.
2. **Quiet hierarchy.** Spacing, contrast, and weight before stronger effects.
3. **Accent with restraint.** Cyan guides interaction, doesn't dominate.
4. **Supportive chrome.** Sidebars and cards frame content, don't compete.
5. **Calm dark mode.** Durable for long reading sessions.
6. **Accessible by default.** Clear contrast, visible focus, restrained motion,
   differentiation that doesn't rely on color alone.
7. **Minimal interactive states.** Prefer underline, color shift, or icon
   emphasis before background fills or shadows that create extra visual layers.

### Heuristics

- Prefer incremental refinement over redesign. Match the existing docs system
  unless there is a clear product or usability reason to diverge.
- No glossy panels, heavy shadows, hover lifts, or decorative gradients.
- Keep interaction language consistent. Links read as links through color and
  underline, not component-specific hover inventions.
- Keep complexity progressive. Core guidance in the main flow; use
  details/collapsible sections for deeper notes when that preserves scanability.
- Tutorials allow more guidance and surface treatment; reference pages stay
  denser, flatter, and more typographic.
- Verify UI changes in both light and dark mode, on at least one tutorial-style
  and one reference-style page.

---
> Source: [treeverse/dvc.org](https://github.com/treeverse/dvc.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
