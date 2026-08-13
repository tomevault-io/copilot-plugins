## portfolio-template-one

> Guidance for any AI assistant (or human) editing this codebase. Read this

# CLAUDE.md

Guidance for any AI assistant (or human) editing this codebase. Read this
before touching files — it explains conventions that aren't obvious from
any single file in isolation.

## What this is

A single-page, no-scroll-by-default portfolio. Next.js App Router +
Tailwind CSS v4 + `motion` (the successor to framer-motion, same API,
imported as `motion/react`) + `lucide-react`. Free, MIT-licensed template —
see `LICENSE`.

## Golden rule: content lives in `lib/config.ts`

Every piece of copy on the page — name, bio, role history, project details,
color themes — is a value in `lib/config.ts`. **Never hardcode display text
inside a component.** If you're adding a sentence, it goes in config, not JSX.

UI-chrome strings (section headings, button labels — things a user of the
*template*, not the portfolio owner, would translate) live separately in
`lib/i18n.ts`. The dividing line:

- Chrome text ("Work", "Projects", "Copy email") → `lib/i18n.ts` → `dict`
- Owner's actual content (bio, role highlights, project descriptions) →
  `lib/config.ts` → as `LocalizedText` / `LocalizedList` objects

## Localization pattern

Four locales: `en` (default), `fr`, `es`, `bn` (Bengali — matches the
portfolio owner's location, Kolkata). Defined in `lib/i18n.ts` as the
`Locale` type and `locales` array (id + flag emoji + native label).

Any config field that should change with the language switcher is typed as:

```ts
type LocalizedText = Record<Locale, string>;       // one string per locale
type LocalizedList = Record<Locale, string[]>;      // array per locale (e.g. bullet highlights)
```

and read in components via `const { locale } = useLocale(); field[locale]`.

**Fields that stay untranslated on purpose:** project names (`Opensource UI`
is a proper noun in every language), person's full name, dates/ranges, URLs,
email, location string, hex colors. Don't wrap these in `LocalizedText` —
it'd be a no-op that adds noise.

**Translation quality**: the fr/es/bn strings currently in `config.ts` are
AI-generated, not reviewed by native speakers. Treat them as a solid draft,
not ground truth — flag this to the user if you're asked to add more locale
content, and suggest a native-speaker pass before anything ships publicly.

To add a locale: add its `Locale` union member, add it to `locales[]` and
every `dict[locale]` entry in `i18n.ts`, then add the matching key to every
`LocalizedText`/`LocalizedList` object in `config.ts`. TypeScript will error
on any object missing a key — that's intentional, treat it as a checklist.

## Theming pattern

`ThemeProvider` (`components/theme-provider.tsx`) holds the active theme in
React state and writes it as inline CSS custom properties (`--bg`, `--fg`,
`--muted`, `--border`, `--accent`, `--accent-fg`) on a wrapper `div`. Tailwind
utility classes (`bg-bg`, `text-fg`, `text-muted`, `border-border`,
`bg-accent`) resolve to those variables via the `@theme inline` block in
`globals.css`. Add a new theme by adding an entry to `siteConfig.themes` —
no component changes needed, the switcher renders from that array.

There is intentionally **no pink/rose theme** — removed on request. Don't
re-add one without asking; if a new theme is wanted, favor muted/desaturated
tones consistent with `paper` / `fog` / `moss` / `ink`.

## Spacing scale — keep it boring and consistent

Do not introduce arbitrary spacing values (`gap-2.5`, `py-3.5`, `m-0.5`, etc.)
without a real reason. The scale in use:

- `gap-1.5` — icon sitting next to its label (tightest allowed gap)
- `gap-2` / `gap-3` — grouping within a row or a small cluster of controls
- `gap-4` — separation between distinct clusters (e.g. header's identity
  block vs. its switchers)
- List row padding is always `px-2 py-3` — Work rows, Project rows, and
  anything else in a bordered `<ul>` list should match this exactly so rows
  feel uniform when scanning down the page.
- Section vertical rhythm is `py-3` per `<section>` (Work, Projects, About);
  the header uses `pt-8 pb-4` since it's the top of the page, not a `py-3`
  section.

If you catch yourself typing `py-3.5` or `gap-2.5`, stop — round to the
nearest value above instead.

## Motion / animation conventions

- **Springs** (`type: "spring", bounce: 0.4–0.6`) are for anything that
  should feel snappy and alive: hover states, popovers opening, icon swaps,
  scale/opacity transitions, the cursor-following project preview.
- **Tweens with an eased curve** (`ease: [0.22, 1, 0.36, 1]`, explicit
  `duration`) are for **height** animations specifically (the Work accordion).
  Springs on `height` look janky/laggy because the browser has to reflow
  layout every frame — this was a real bug, keep the fix.
- The project hover-preview `<motion.div>` is keyed by `active.id`
  (`key={active.id}` inside `AnimatePresence`). This is required — without
  it, switching from hovering Project A directly to Project B re-renders the
  *same* element with a new background instead of unmounting/remounting, so
  the gradient just snaps instead of crossfading. Don't remove the key.
- `AnimatePresence mode="popLayout"` on that same block prevents the
  outgoing and incoming card from both existing in the layout at once.

## Interaction / UX conventions

- Email contact is a **copy-to-clipboard button**, not a `mailto:` link —
  see `about-section.tsx`. Deliberate choice (mailto assumes a configured
  mail client, which many people don't have). Icon swaps `Mail` → `Check`
  with `AnimatePresence mode="wait"` on click.
- Social links (GitHub/LinkedIn/Twitter) and Resume are **plain text**, no
  icons — deliberate, not a placeholder to "fix" later.
- The project row's outbound link icon is `ArrowUpRight` from lucide-react.
- Every clickable `<button>` needs `cursor-pointer` explicitly (Tailwind
  preflight sets buttons to `cursor: default`), though `globals.css` also
  sets a global `button:not(:disabled) { cursor: pointer }` fallback — belt
  and suspenders.
- Reduced-motion is respected globally via a `prefers-reduced-motion` media
  query in `globals.css`. Don't add animations that bypass it.

## Layout goal: one screen, no scroll (on normal viewports)

`app/page.tsx`'s `<main>` uses `sm:h-dvh` with `flex-col justify-center` —
the intent is that on a typical laptop/desktop viewport, the whole page
(header, Work, Projects, About/contact) fits without scrolling. This is why:

- Work highlights are capped at **2 bullets per role**, not full history
- There's no certifications/awards section
- There's no in-page nav to "scroll to Work" — there's nothing to scroll to

If you're asked to add a section, first ask whether it should collapse by
default (like Work's accordion) rather than always render expanded — that's
the pattern for fitting more content without breaking the one-screen goal.
On narrow/short viewports the layout falls back to `min-h-dvh` and will
scroll naturally — that's an accepted fallback, not a bug to "fix" by hiding
content.

## Assets not included in this repo

`siteConfig.resume` points at `/pdfs/Software_Engineer-Bidyut_Kundu.pdf`,
which doesn't exist in `public/` — the owner needs to add it. There are no
other missing assets; project previews are pure CSS gradients
(`gradientFrom`/`gradientTo` per project in config), not screenshots.

## File map

```
app/
  layout.tsx     Geist font (next/font/google), <html>/<body>, metadata
  page.tsx        <ThemeProvider><LocaleProvider><main>...
  globals.css      Tailwind v4 import, default theme CSS vars, global resets
components/
  theme-provider.tsx     React context; writes active theme as CSS vars
  theme-switcher.tsx       popover of theme swatches (reads siteConfig.themes)
  locale-provider.tsx     React context; holds active Locale + t()
  language-switcher.tsx    popover of flag buttons (reads i18n.locales)
  popover-button.tsx        shared click-outside-to-close popover shell
  nav-bar.tsx                 name, open-to-work badge, tagline, socials
  work-section.tsx             role accordion (click row to expand)
  projects-section.tsx          project rows + cursor-tracked gradient preview
  about-section.tsx              current-focus line, contact, footer credit
lib/
  config.ts    ALL content — read this first when asked to change copy
  i18n.ts       chrome-string dictionary + locale list
LICENSE      MIT, plus a non-binding "please star" note (not a license term)
```

## Things NOT to do without being asked

- Don't add a certifications/skills/tech-stack section back — removed
  deliberately to keep the page short.
- Don't add real project screenshots back — gradients were a deliberate
  choice over images.
- Don't re-introduce `mailto:` for the email contact.
- Don't add a pink/rose theme.
- Don't add icons to the social/resume links in the header.
- Don't use `ChevronRight`/`ChevronLeft` for the project outbound-link
  icon — it's `ArrowUpRight`, per the owner's explicit code.

---
> Source: [bidyut10/portfolio-template-one](https://github.com/bidyut10/portfolio-template-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
