## grocery-delivery

> |


# Full-Stack UI/UX + PHP Skill

This repository contains two major capability sets:

1. UI/UX intelligence driven by searchable design datasets and generation scripts.
2. Native PHP full-stack engineering guidance for production-grade apps.

This file consolidates both into one operating guide.

## When to apply

Use this skill whenever the task changes how the product looks, behaves, or works.

### Must use

- New screens: landing pages, dashboards, admin panels, SaaS views, app interfaces.
- UI component work: buttons, cards, forms, tables, dialogs, navigation, charts.
- Design decisions: style, palette, typography, spacing, interaction, motion.
- Frontend implementation: HTML5, Tailwind CSS v4, vanilla JS patterns.
- Backend implementation in PHP: routing, MVC, services, repositories, APIs.
- Data/auth/security work: PDO queries, transactions, sessions, CSRF, JWT, RBAC.
- Full-stack feature delivery that crosses UI and backend boundaries.

### Recommended

- UI looks inconsistent or low-quality but root cause is unclear.
- Users report usability, accessibility, or interaction friction.
- Auth flows need hardening, cleanup, or modernization.
- You need consistent rules for both design and implementation.

### Skip

- Pure infrastructure or DevOps tasks with no product/UI/backend code changes.
- Tasks unrelated to web app behavior.

## Core operating model

### Track A: UI/UX intelligence

Use data-backed search and design-system generation to pick style direction,
colors, typography, UX rules, and stack-specific guidance.

### Track B: PHP full-stack engineering

Use the references for architecture, secure auth, authorization, and MySQL/PDO
implementation details in a framework-free PHP 8.5 codebase.

### Track C: Integrated execution

For real features, run both tracks in order:

1. Define UX and visual rules.
2. Implement frontend structure and interactions.
3. Build backend routes, services, and persistence.
4. Apply security and access controls.
5. Verify quality with UX + security checklists.

## Rule categories by priority (long-form)

Use this matrix to decide validation order before implementation and before release.

| Priority | Category | Impact | Domain | Key checks (must have) | Anti-patterns (avoid) |
|---|---|---|---|---|---|
| 1 | Accessibility | CRITICAL | ux | Contrast 4.5:1, alt text, keyboard nav, aria labels | Hidden focus, icon-only controls without labels |
| 2 | Touch and interaction | CRITICAL | ux | Min target 44x44, 8px spacing, loading feedback | Hover-only behavior, instant unannounced state jumps |
| 3 | Performance | HIGH | ux | WebP/AVIF, lazy loading, reserve layout space | Layout thrashing, cumulative layout shift |
| 4 | Style selection | HIGH | style, product | Product-fit style, consistency, SVG icons | Random style mixing, emoji-as-icon structure |
| 5 | Layout and responsive | HIGH | ux | Mobile-first, viewport meta, no horizontal scroll | Fixed narrow desktop assumptions on mobile |
| 6 | Typography and color | MEDIUM | typography, color | 16px base, line-height 1.5+, semantic tokens | Tiny body text, gray-on-gray, raw ad-hoc hex values |
| 7 | Animation | MEDIUM | ux | 150-300ms timing, meaningful motion, continuity | Decorative-only motion, width/height animation |
| 8 | Forms and feedback | MEDIUM | ux | Visible labels, inline errors, helper text | Placeholder-only labels, delayed ambiguous errors |
| 9 | Navigation patterns | HIGH | ux | Predictable back, max 5 bottom tabs, deep links | Overloaded nav, broken back stack |
| 10 | Charts and data | LOW | chart | Correct chart type, legend/tooltips, accessible colors | Color-only encoding, unreadable dense charts |

## UI/UX quick reference (long-form)

### 1) Accessibility (CRITICAL)

- color-contrast: minimum 4.5:1 for normal text, 3:1 for large text.
- focus-states: keep visible focus indicators on interactive elements.
- alt-text: meaningful alt text for meaningful images.
- aria-labels: label icon-only controls.
- keyboard-nav: tab order must match visual order.
- form-labels: each input needs a visible label.
- skip-links: support skip to main content.
- heading-hierarchy: keep heading levels sequential.
- color-not-only: never use color as the only signal.
- dynamic-type: support text scaling without clipped UI.
- reduced-motion: honor reduced motion preferences.
- voiceover-sr: ensure logical screen-reader reading order.
- escape-routes: provide cancel/back in dialogs and flows.
- keyboard-shortcuts: preserve system and accessibility shortcuts.

### 2) Touch and interaction (CRITICAL)

- touch-target-size: minimum 44x44pt (iOS) or 48x48dp (Android).
- touch-spacing: minimum 8px spacing between targets.
- hover-vs-tap: primary interactions cannot depend on hover.
- loading-buttons: disable and show pending state during async.
- error-feedback: show local, actionable errors near source.
- cursor-pointer: use pointer cursor for clickable web elements.
- gesture-conflicts: avoid conflicting nested gestures.
- tap-delay: use touch-action where relevant to reduce delay.
- standard-gestures: follow platform-standard gestures.
- system-gestures: do not block OS-level gestures.
- press-feedback: give immediate visual press response.
- haptic-feedback: use haptics sparingly for key confirmations.
- gesture-alternative: provide visible alternatives to hidden gestures.
- safe-area-awareness: keep controls clear of notches/gesture bars.
- no-precision-required: avoid tiny precision-dependent controls.
- swipe-clarity: expose swipe affordances where used.
- drag-threshold: require threshold before drag begins.

### 3) Performance (HIGH)

- image-optimization: use WebP/AVIF and responsive sources.
- image-dimension: reserve width/height or aspect-ratio.
- font-loading: use font-display swap/optional.
- font-preload: preload only critical faces.
- critical-css: prioritize above-the-fold CSS.
- lazy-loading: lazy-load below-the-fold modules/media.
- bundle-splitting: split by route/feature.
- third-party-scripts: async/defer and audit regularly.
- reduce-reflows: batch reads/writes and avoid thrashing.
- content-jumping: reserve async content space.
- lazy-load-below-fold: defer heavy media off initial viewport.
- virtualize-lists: virtualize long lists.
- main-thread-budget: keep frame work near 16ms.
- progressive-loading: prefer skeletons over long spinners.
- input-latency: target sub-100ms interaction response.
- tap-feedback-speed: visible response within 100ms.
- debounce-throttle: guard high-frequency handlers.
- offline-support: provide useful offline state.
- network-fallback: degrade gracefully on slow networks.

### 4) Style selection (HIGH)

- style-match: match style to product and audience.
- consistency: keep one coherent visual language.
- no-emoji-icons: use SVG/vector icon libraries for structure.
- color-palette-from-product: choose palette by domain context.
- effects-match-style: shadow/blur/radius must fit chosen style.
- platform-adaptive: respect iOS/Android/web idioms.
- state-clarity: hover/pressed/disabled must be distinct.
- elevation-consistent: use a consistent elevation scale.
- dark-mode-pairing: design light/dark as a pair.
- icon-style-consistent: keep stroke/corner style consistent.
- system-controls: prefer native controls where possible.
- blur-purpose: blur should communicate layering, not decoration.
- primary-action: one clear primary CTA per screen.

### 5) Layout and responsive (HIGH)

- viewport-meta: width=device-width initial-scale=1.
- mobile-first: design from small to large screens.
- breakpoint-consistency: use systematic breakpoints.
- readable-font-size: body text minimum 16px on mobile.
- line-length-control: keep readable line lengths.
- horizontal-scroll: no horizontal overflow on mobile.
- spacing-scale: use consistent 4/8-based spacing rhythm.
- touch-density: avoid overly dense touch layouts.
- container-width: use consistent max-width on desktop.
- z-index-management: define a layered z-index scale.
- fixed-element-offset: reserve space for fixed nav bars.
- scroll-behavior: avoid problematic nested scroll regions.
- viewport-units: prefer dynamic viewport units on mobile.
- orientation-support: validate portrait and landscape.
- content-priority: prioritize primary content on small screens.
- visual-hierarchy: use size/spacing/contrast for hierarchy.

### 6) Typography and color (MEDIUM)

- line-height: use 1.5 to 1.75 for body text.
- line-length: keep body lines readable.
- font-pairing: align heading/body personality.
- font-scale: use a consistent type scale.
- contrast-readability: ensure readable foreground/background pairs.
- text-styles-system: use platform/system type roles.
- weight-hierarchy: use weight consistently for hierarchy.
- color-semantic: use semantic tokens, not raw literal values.
- color-dark-mode: design true dark variants, not inversion hacks.
- color-accessible-pairs: verify contrast for all key pairs.
- color-not-decorative-only: add icon/text cues beyond color.
- truncation-strategy: prefer wrapping where possible.
- letter-spacing: avoid overly tight tracking for body text.
- number-tabular: use tabular numerals in data-heavy UI.
- whitespace-balance: separate groups with intentional whitespace.

### 7) Animation (MEDIUM)

- duration-timing: micro 150-300ms, complex up to 400ms.
- transform-performance: animate transform/opacity only.
- loading-states: show progress if loading exceeds 300ms.
- excessive-motion: limit simultaneous key motions.
- easing: ease-out for enter, ease-in for exit.
- motion-meaning: animation must communicate state/causality.
- state-transition: avoid abrupt state snaps.
- continuity: preserve spatial continuity between screens.
- parallax-subtle: keep parallax subtle and optional.
- spring-physics: prefer natural spring curves when appropriate.
- exit-faster-than-enter: keep exits faster than enters.
- stagger-sequence: stagger list reveals lightly.
- shared-element-transition: use for continuity across routes.
- interruptible: allow user actions to interrupt motion.
- no-blocking-animation: animation should not block input.
- fade-crossfade: use crossfade for same-container replacement.
- scale-feedback: subtle press-scale feedback is acceptable.
- gesture-feedback: drag/swipe should track finger in real time.
- hierarchy-motion: directional motion should reflect hierarchy.
- motion-consistency: use shared timing/easing tokens.
- opacity-threshold: avoid lingering nearly-invisible elements.
- modal-motion: animate modal entry from logical origin.
- navigation-direction: keep forward/back motion direction consistent.
- layout-shift-avoid: animation must not cause CLS.

### 8) Forms and feedback (MEDIUM)

- input-labels: visible label for every field.
- error-placement: show error near offending field.
- submit-feedback: loading then explicit success/error.
- required-indicators: mark required inputs clearly.
- empty-states: provide clear empty-state guidance.
- toast-dismiss: auto-dismiss non-critical toasts.
- confirmation-dialogs: confirm destructive actions.
- input-helper-text: helper text for complex inputs.
- disabled-states: disabled controls must look and act disabled.
- progressive-disclosure: reveal complexity gradually.
- inline-validation: validate on blur/commit, not every keystroke.
- input-type-keyboard: choose semantic input types.
- password-toggle: allow show/hide passwords.
- autofill-support: enable autofill metadata.
- undo-support: allow undo for destructive actions.
- success-feedback: short positive confirmation feedback.
- error-recovery: errors must include recovery actions.
- multi-step-progress: show progress and allow back.
- form-autosave: autosave long forms/drafts when practical.
- sheet-dismiss-confirm: confirm close when unsaved data exists.
- error-clarity: include cause and fix, not generic failure text.
- field-grouping: group related fields semantically.
- read-only-distinction: read-only and disabled should differ.
- focus-management: focus first invalid field on submit failure.
- error-summary: provide top summary for multi-error forms.
- touch-friendly-input: mobile input height should be touch-safe.
- destructive-emphasis: visually separate destructive actions.
- toast-accessibility: use polite live regions for toasts.
- aria-live-errors: announce form errors for assistive tech.
- contrast-feedback: feedback colors need accessible contrast.
- timeout-feedback: timeouts should include retry path.

### 9) Navigation patterns (HIGH)

- bottom-nav-limit: keep bottom nav to five items or fewer.
- drawer-usage: reserve drawers for secondary navigation.
- back-behavior: keep back behavior predictable and stateful.
- deep-linking: key screens should be deep-linkable.
- tab-bar-ios: use iOS tab bar conventions.
- top-app-bar-android: follow Android app-bar patterns.
- nav-label-icon: include both icon and label.
- nav-state-active: clearly indicate active destination.
- nav-hierarchy: separate primary vs secondary navigation layers.
- modal-escape: provide clear modal close routes.
- search-accessible: keep search easy to reach.
- breadcrumb-web: use breadcrumbs in deep web hierarchies.
- state-preservation: preserve scroll/filter/input on back.
- gesture-nav-support: support native back gestures.
- tab-badge: use badges sparingly and clear them reliably.
- overflow-menu: overflow excess actions cleanly.
- bottom-nav-top-level: use bottom nav only for top-level routes.
- adaptive-navigation: switch to side-nav on larger layouts.
- back-stack-integrity: avoid silent stack resets.
- navigation-consistency: keep nav placement consistent.
- avoid-mixed-patterns: do not mix equivalent nav systems.
- modal-vs-navigation: modals are not primary navigation.
- focus-on-route-change: move focus to main content on route change.
- persistent-nav: keep core navigation reachable.
- destructive-nav-separation: separate destructive nav actions.
- empty-nav-state: explain unavailable destinations.

### 10) Charts and data (LOW)

- chart-type: map chart type to data story.
- color-guidance: avoid red/green-only differentiation.
- data-table: provide tabular alternative where needed.
- pattern-texture: add non-color differentiation cues.
- legend-visible: keep legends visible and close.
- tooltip-on-interact: show exact values on interaction.
- axis-labels: label units and keep scale readable.
- responsive-chart: simplify chart density on small screens.
- empty-data-state: show meaningful empty states.
- loading-chart: show loading placeholders during fetch.
- animation-optional: respect reduced motion.
- large-dataset: aggregate/sample large data by default.
- number-formatting: use locale-aware formatting.
- touch-target-chart: touch-safe interactive marks.
- no-pie-overuse: avoid pie/donut when category count is high.
- contrast-data: keep data marks and labels readable.
- legend-interactive: allow toggling series when useful.
- direct-labeling: label directly for small datasets.
- tooltip-keyboard: keyboard-accessible tooltip behavior.
- sortable-table: table sorting should be accessible.
- axis-readability: avoid cramped axis ticks.
- data-density: split over-dense charts into multiple views.
- trend-emphasis: prioritize trend clarity over decoration.
- gridline-subtle: keep gridlines low-contrast.
- focusable-elements: focusable interactive marks in accessible charts.
- screen-reader-summary: provide chart summary for assistive tech.
- error-state-chart: show recoverable chart load failures.
- export-option: provide export for data-heavy workflows.
- drill-down-consistency: preserve hierarchy and back-path.
- time-scale-clarity: expose and label time granularity.

## Repository assets

## Data sources in data/

The search engine and generator use structured CSV data.

### Active domain datasets

| File | Rows | Purpose |
|---|---:|---|
| data/styles.csv | 84 | Style system candidates, effects, compatibility, complexity |
| data/colors.csv | 161 | Product-type semantic palettes and UI token colors |
| data/charts.csv | 25 | Data type to chart-type decision support |
| data/landing.csv | 34 | Landing section and conversion patterns |
| data/products.csv | 161 | Product-type style and pattern recommendations |
| data/ux-guidelines.csv | 99 | UX do/dont rules with severity |
| data/typography.csv | 73 | Font pairing and usage guidance |
| data/google-fonts.csv | 1923 | Font discovery metadata |
| data/icons.csv | 105 | Icon guidance and library mappings |
| data/react-performance.csv | 44 | React/Next performance and architecture pitfalls |
| data/app-interface.csv | 30 | Web interface quality and implementation notes |
| data/ui-reasoning.csv | 161 | Reasoning rules used by design-system generator |

### Stack-specific datasets (data/stacks/)

16 stack files are available with guideline-level do/dont patterns:

- angular
- astro
- flutter
- html-tailwind
- jetpack-compose
- laravel
- nextjs
- nuxt-ui
- nuxtjs
- react
- react-native
- shadcn
- svelte
- swiftui
- threejs
- vue

### Reference-only large files

- data/design.csv and data/draft.csv are long-form design backups.
- They are not part of the active CLI search index used by scripts/search.py.

## Script toolkit in scripts/

### scripts/core.py

BM25 search engine over CSV domains.

- Auto-domain detection from query keywords.
- Explicit domain search via `--domain`.
- Stack search via `--stack`.

Supported domains:

- style
- color
- chart
- landing
- product
- ux
- typography
- google-fonts
- icons
- react
- web

### scripts/design_system.py

Design system generator that:

- Runs multi-domain searches.
- Applies reasoning from data/ui-reasoning.csv.
- Produces pattern/style/colors/typography recommendations.
- Supports persistence using a Master + Overrides structure:
  - design-system/<project>/MASTER.md
  - design-system/<project>/pages/<page>.md

### scripts/search.py

CLI entry point for search and design-system generation.

Common usage:

```bash
python scripts/search.py "saas dashboard" --domain style
python scripts/search.py "virtualize list" --stack react
python scripts/search.py "fintech app" --design-system -p "FinX"
python scripts/search.py "admin panel" --design-system --persist -p "Ops" --page "dashboard"
```

### scripts/rg.*

Pinned ripgrep wrappers for environments where `rg` is unavailable globally:

- scripts/rg.cjs
- scripts/rg.cmd
- scripts/rg.ps1

## Engineering references in references/

Read only what matches the task, but use all relevant files for multi-layer work.

| File | Use for |
|---|---|
| references/php-core.md | PHP 8.5 language features, strict typing, enums, OOP patterns, errors, CLI |
| references/pdo-mysql.md | PDO config, prepared queries, CRUD, transactions, migrations, indexing |
| references/frontend.md | HTML5 semantics, Tailwind v4.2.2, responsive UI, forms, vanilla JS |
| references/architecture.md | Front controller, router, request lifecycle, sessions, REST endpoints |
| references/security.md | Validation, escaping, SQLi/XSS/CSRF prevention, headers, session hardening |
| references/authentication.md | Session auth, Argon2id, JWT access tokens, refresh token rotation |
| references/authorization.md | RBAC, permissions, policy checks, route guards, JWT scope decisions |
| references/auth-schema.md | Migration-ready auth schema, constraints, index strategy, seed order |
| references/middleware.md | AuthMiddleware, PermissionMiddleware, JwtMiddleware integration |

## Non-negotiable implementation rules

### UI/UX quality

1. Accessibility first: contrast, focus visibility, keyboard and screen-reader support.
2. Touch-safe interactions: minimum target sizing and clear feedback.
3. Responsive behavior without horizontal overflow.
4. Motion used meaningfully; respect reduced-motion.
5. Use semantic color and typography systems, not ad-hoc values.

### PHP/backend quality

1. Every PHP file starts with `declare(strict_types=1);`.
2. Use typed signatures and properties wherever practical.
3. Controllers remain thin; business logic belongs in services.
4. Use prepared statements only for external input.
5. Use transactions for multi-step writes.
6. Hash passwords with Argon2id via `password_hash` and verify with `password_verify`.
7. Regenerate session ID on login and enforce CSRF on state-changing browser requests.
8. Escape output at render time with `htmlspecialchars`.
9. Enforce authorization server-side; UI visibility is never a permission check.
10. Keep secrets in env files, not source control.

## Decision flow

Use this quick routing before implementation:

1. If the task is visual or interaction-heavy:
	- Run `scripts/search.py` for domain and stack guidance.
	- Use generated style/color/typography/pattern decisions.
2. If the task is backend/data/auth-heavy:
	- Read matching files under references/.
	- Implement with strict typing, PDO patterns, and security defaults.
3. If the task is full feature delivery:
	- Start with product + style + UX search.
	- Implement frontend structure from references/frontend.md.
	- Implement backend flow from architecture + PDO + auth refs.
	- Final pass with security.md and UX checklist.

## Default stack assumptions

Unless the repository explicitly overrides them:

- PHP 8.5
- Native MVC-style structure with public web root
- PDO + MySQL with utf8mb4
- Tailwind CSS 4.2.2
- Vanilla JS modules
- Session auth for web and JWT for APIs where needed

## Expected output behavior when this skill is used

Responses and code should be:

- Concrete and implementation-ready.
- Security-aware by default.
- UX-aware by default.
- Consistent across UI and backend layers.
- Minimal in abstraction and rich in practical choices.

## Quick examples

### Example: New SaaS dashboard

1. `python scripts/search.py "saas dashboard" --design-system -p "Project"`
2. `python scripts/search.py "dashboard table interactions" --domain ux`
3. Implement UI with `references/frontend.md`.
4. Add routes/controllers/services using `references/architecture.md`.
5. Add secure data flows using `references/pdo-mysql.md` and `references/security.md`.

### Example: Login + admin permissions

1. Build auth flow using `references/authentication.md`.
2. Build RBAC and guards with `references/authorization.md` and `references/middleware.md`.
3. Validate schema and indexes with `references/auth-schema.md`.
4. Verify CSRF/session/password/security headers with `references/security.md`.

## Deep pre-delivery checklist (long-form)

Use these final checks in addition to backend security checks.

### Visual quality

- No emoji used as structural icons.
- Icon family, stroke, and visual style are consistent.
- Brand assets use official ratios and clear-space rules.
- Press states do not cause layout shifts or jitter.
- Semantic theme tokens are used consistently.

### Interaction quality

- All tappable controls provide clear press feedback.
- Touch targets meet minimum size guidance.
- Micro-interactions are in the 150-300ms range.
- Disabled states are clearly non-interactive.
- Screen-reader labels and focus order are valid.
- Gestures do not conflict with each other or OS navigation.

### Light and dark mode quality

- Primary text contrast is >= 4.5:1 in both themes.
- Secondary text contrast is >= 3:1 in both themes.
- Dividers and control states remain visible in both themes.
- Modal and drawer scrim preserves foreground legibility.
- Both themes are tested directly; no inferred parity.

### Layout quality

- Safe areas are respected for fixed top/bottom bars.
- Scroll content is not hidden behind fixed elements.
- Verified across small phone, large phone, tablet, and landscape.
- Gutters and insets adapt across breakpoints.
- 4/8 spacing rhythm is consistent.
- Long-form text remains readable on wide screens.

### Accessibility quality

- All meaningful media and controls have accessible names.
- Form labels, hints, and errors are explicit and local.
- Color is not the only encoding channel.
- Reduced motion and text scaling do not break layout.
- Selected, disabled, and expanded states are announced correctly.

### Backend and security quality

- SQL is parameterized and query paths are indexed.
- Auth flow regenerates sessions and validates CSRF.
- Permission checks are enforced server-side.
- Error states are useful for users and safe for logs.
- Feature behavior is consistent across browser and API paths.

---
> Source: [sisovin/grocery-delivery](https://github.com/sisovin/grocery-delivery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
