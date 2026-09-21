## ifnotgod-website-landing-page

> Engineering standards for the IFNOTGOD TECH LTD marketing website: a single-page, conversion-focused corporate site (see project brief in `CLAUDE.md`). No auth, no user accounts, no backend domain beyond capturing and delivering contact/lead submissions.

# AGENTS.md - IFNOTGOD TECH LTD

Engineering standards for the IFNOTGOD TECH LTD marketing website: a single-page, conversion-focused corporate site (see project brief in `CLAUDE.md`). No auth, no user accounts, no backend domain beyond capturing and delivering contact/lead submissions.

## Engineering Standards
- Use TypeScript for all application code.
- Use Next.js App Router (`src/app`) for all route definitions.
- Keep the page composed from section components matching the brief's page structure (Nav, Hero, Problem, Solutions, AI, How We Work, Why Us, Who We Help, About, Conversion CTA, Contact, Footer).
- Validate incoming data at boundary layers (client form and server route handler).
- Favor explicit types for domain entities (`ContactSubmission`).
- All modules must be thoroughly tested.
- Minimum expectation:
  - unit tests for form validation logic and edge cases
  - integration test for the contact submission API route
  - UI interaction tests for form states (idle, submitting, success, error)

## Suggested Folder Structure (Next.js Reference)
- `src/app` (App Router: root layout, the single page, `api/contact` route handler)
- `src/components/sections` (Hero, Problem, Solutions, AI, HowWeWork, WhyUs, WhoWeHelp, About, ConversionCta, Contact, Footer, Navigation)
- `src/components/ui` (shared reusable UI primitives: Button, Card, Input, TextArea, Badge)
- `src/lib` (contact-submission service, email/notification client, config)
- `src/shared` (constants, validators, shared helpers)
- `src/types` (`ContactSubmission` and other shared contracts)
- `src/styles` (global styles, theme tokens)

## Next.js Architecture Rules
- Keep route concerns in `src/app`; keep section composition and any submission logic in `src/components/sections` and `src/lib`.
- Prefer Server Components by default; use Client Components only where interactivity is required (contact form, nav hamburger, scroll/fade animations).
- Co-locate section-specific UI/hooks within `src/components/sections`.
- Use shared UI components from `src/components/ui` to maintain design consistency.

## Performance Standards
- Optimize for Core Web Vitals on the landing page:
  - LCP under 2.5s on standard broadband
  - INP under 200ms for core interactions (nav, form, CTAs)
  - CLS under 0.1
- Use lazy loading for below-the-fold media and non-critical sections.
- Minimize client bundle size:
  - avoid unnecessary client components
  - use dynamic imports for heavy optional UI (e.g. animation-heavy sections)
- Optimize images (hero visuals, icons) using Next.js image optimization patterns.
- Monitor performance regressions in staging before release.

## Delivery Priorities (MVP)
1. Page scaffold, layout, navigation (with sticky behavior + mobile hamburger)
2. Hero + Problem + Solutions sections
3. AI section + How We Work + Why Us + Who We Help
4. About + Conversion CTA section
5. Contact form + submission handling (API route, validation, delivery of leads)
6. SEO/meta tags, performance pass, full responsive QA

## Definition of Done (Feature Level)
A feature/section is done when:
- Functional acceptance criteria from the brief are met (copy, CTAs, layout intent)
- Client and server validation checks are in place for any data entry
- Error, empty, loading, and success states are handled where applicable
- Tests are implemented and passing for affected modules
- Documentation is updated if behavior or structure changes

## Change Management
- Keep the `ContactSubmission` shape synchronized between client form, validator, and API route.
- Any change to the contact/lead capture flow must consider what happens to in-flight or failed submissions (no silent data loss).
- Update this `AGENTS.md` when section structure, naming, or module boundaries change.

## Resource Cleanup Rules
- Always clean up subscriptions, timers, event listeners, observers, and custom browser integrations.
- Clean up side effects properly on unmount and dependency changes.
- Abort stale requests when needed.
- Prevent state updates after unmount.
- Do not leak scroll/intersection observers across re-renders.

## Heavy Work Rules
- Never block the browser main thread.
- Do not perform expensive transformations inside render paths.
- Prefer server-side rendering for static content; keep client-side JS limited to interactivity and animation.

## Network Rules
- Use a centralized HTTP/client abstraction for the contact submission call.
- Handle errors with typed exceptions or normalized failures.
- Normalize the contact form payload before it reaches the API route.
- Do not let raw request/response shapes leak into presentation code.

## Caching and Data Ownership Rules
- Static marketing content is source-of-truth in code (or CMS, if later added) — not duplicated in client state.
- Keep any filter/tab UI state (if introduced, e.g. solutions filtering) in local state; use URL state only if it needs to be shareable.
- Derived UI state should be computed, not redundantly stored.

## Data Parsing and Mapping Rules
- Do not perform expensive data shaping inside components.
- Validate and normalize the contact form payload in the data/API layer, not in presentation code.

## State Management Rules
- Keep state minimal.
- Use local state for isolated interactions (form fields, nav toggle, accordion/tab UI).
- Use immutable updates.
- Avoid global state; this page does not need a client store.

## Error Handling Rules
- Use exceptions in the data/integration layer (e.g. email/lead delivery failures).
- Use typed failures or normalized error objects in the application layer.
- Never expose raw exceptions directly to the UI.
- Presentation layer must map failures into clear, human user-facing messages (e.g. "Something went wrong sending your message — try again or reach us on WhatsApp").
- The contact form must support loading, empty, error, and success states.
- Add retry-friendly UX on submission failure (do not clear the user's input on error).

## Dependency Management Rules
- Centralize the email/lead-delivery client construction in `src/lib`.
- Avoid ad hoc instantiation of shared clients across sections.
- Keep the contact-delivery dependency swappable (e.g. email provider, form backend, CRM webhook) behind one interface.

## UI, Responsive, and Motion Rules
- Follow consistent spacing, typography, hierarchy, and interaction patterns across all sections.
- Use reusable components and avoid duplicated UI code (cards, buttons, section headings).
- UI files must remain focused on presentation and interaction wiring, not business/validation logic.
- Standardize repeated UI patterns: buttons, inputs, cards, and the section-header pattern used throughout the page.
- All sections must support adaptive layouts across mobile, tablet, and desktop.
- Do not build fixed mobile-only layouts.
- Use responsive primitives: CSS Grid, Flexbox, container constraints, and breakpoint composition.
- Ensure text remains usable under zoom and larger text settings.
- Before marking any phase complete, test key flows (nav, scroll, form submission) at mobile, tablet, and desktop widths.
- Before marking any phase complete, confirm no overflow, clipped CTAs, or inaccessible controls.
- Use motion sparingly and intentionally, per the brief: fade-in on scroll, slight card movement, smooth section transitions, gentle hero animation.
- Avoid continuous or decorative motion.
- Animations must never block interaction.
- Respect reduced-motion preferences.
- Verify no visible jank on scroll or hover interactions.

## Brand and CTA Color Rules
- Brand palette:
  - Maroon/Burgundy `#6B1F2A` — primary brand color; primary CTA background, key accents.
  - Black `#000000` — primary dark background/base for the site's dark aesthetic.
  - White `#FFFFFF` — primary text on dark surfaces, CTA text on maroon/orange.
  - Orange `#E07B2A` — secondary accent; secondary/tertiary CTA background, highlights, icon accents, matches the logo's gradient warmth.
- Define all colors as theme tokens (`src/styles`), not hardcoded hex in components; derive hover/active shades from the tokens (e.g. `color-mix` or a small shade scale) rather than inventing new one-off hex values per component.
- Do not overuse orange as decorative/neon — reserve it for CTAs, highlights, and small accent moments per the brief's "avoid overusing neon colours" direction.
- Keep button/CTA text contrast accessible (minimum WCAG AA): white text on maroon and on orange both pass AA for normal-size CTA text; verify any smaller/lighter text combinations before shipping.

## UI File Size and Composition Rules
- Avoid oversized UI files; split components when a file grows beyond a maintainable size.
- As a guideline, refactor UI files approaching ~250-300 lines, especially when multiple concerns are mixed.
- Separate concerns clearly:
  - presentation in section/component files
  - validation/parsing/mapping in `src/lib` or `src/shared`
- Extract repeated JSX sections (e.g. card grids) into subcomponents instead of long monolithic files.
- Keep the page-level file thin by composing section components.
- Keep component props explicit and typed.

## Frontend Security Rules
- Never trust client input; validate the contact form again at the server boundary.
- Never expose secrets (email API keys, webhook URLs) in the client bundle.
- Keep client-safe and server-only environment variables clearly separated.
- Sensitive actions (sending the contact submission) must go through a trusted server-side route handler, not a direct client-side call to a third-party service.
- Rate-limit or otherwise guard the contact endpoint against spam/abuse.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [ifnotGodTech/ifnotGod_website_landing_page](https://github.com/ifnotGodTech/ifnotGod_website_landing_page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
