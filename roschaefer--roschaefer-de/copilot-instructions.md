## roschaefer-de

> This repository contains the current `roschaefer.de` portfolio site. Treat this file as durable project guidance for coding agents and automation. For human-facing setup and deployment details, start with `README.md`.

# Agent Notes

This repository contains the current `roschaefer.de` portfolio site. Treat this file as durable project guidance for coding agents and automation. For human-facing setup and deployment details, start with `README.md`.

## Product Intent

The site serves two purposes:

1. A web portfolio for recruiters, hiring managers, engineering leads, peers, and collaborators.
2. A high-quality CV/PDF derived from the same resume content source.

Core goals:

- Clarify Robert's positioning quickly for hiring and collaboration contexts.
- Keep the web portfolio and downloadable CV consistent without duplicating content by hand.
- Support German and English, with German treated as the primary audience language.
- Keep the site privacy-conscious, accessible, static-first, and easy to maintain.
- Prefer test-driven changes for data transformations, rendering logic, and critical UI behavior.

Content should stay focused on value and evidence:

- About/bio
- Experience with impact
- Featured projects
- Skills and technology experience
- Open source
- Talks/media where relevant
- Contact
- Resume PDF
- Legal pages

Each section should answer "why this matters" or "what was built". Prefer concrete outcomes over buzzwords.

## Information Architecture

The main portfolio page should remain scannable and direct:

- Hero with name, role, positioning, and primary calls to action.
- Experience with role impact, scope, and relevant stack.
- Featured projects with problem, solution, stack, result, and links.
- Skills and technology experience, including derived years of experience where defensible.
- Open source as a first-class section, not just a link list.
- Talks/media with privacy-friendly behavior.
- Contact options.
- Footer navigation to legal pages.

Optional content can be added when it has enough substance:

- Writing
- Testimonials
- Current-focus/"now" content
- Project detail pages

## Visual And UX Direction

Preserve the current site's visual identity unless a change improves clarity, accessibility, responsiveness, or PDF/print output.

Current direction:

- Dark portfolio identity with cool blue/cyan accents.
- Strong display typography with readable body copy.
- Generous spacing and clear section hierarchy.
- Subtle motion only where it improves orientation or polish.
- Motion must respect `prefers-reduced-motion`.
- Interactive enhancements must degrade to readable static content.

Do not add visual changes just to restyle the site. Make departures intentional and explain the user-facing reason.

## Content Source Of Truth

The authored resume source is:

- `resume.i18n.json`

There are no generated `resume.de.json`/`resume.en.json` files committed to the repo — they're gitignored and regenerated fresh by `pnpm build:json-resume` (`scripts/generate-resume-source.ts`):

1. Decrypt `resume.i18n.json` (`sops -d`) and mask redacted-client fields (see "Redacted Clients" below), writing the result to `.generated/resume-source.json` (locale-agnostic, never committed).
2. `deriveResume` (`src/lib/utils/derive-resume.ts`, a pure function) localizes that into a single locale's JSON Resume data, sorts dated sections, and computes skills. Written to `.generated/resume.de.json` / `.generated/resume.en.json`.

This avoids keeping derived copies in sync, and avoids stale generated output (e.g. duration text computed from "now") ever getting committed. `build:json-resume` is the only place `sops`/decryption is ever invoked — everything downstream (`src/lib/data/resume.ts`, `scripts/build-typst-data.ts`, tests) just reads the generated per-locale files as plain data, so none of it needs to be Node-only or server-only.

The pipeline is:

- `resume.i18n.json` -> (`pnpm build:json-resume`, decrypt + mask, one Node script) -> `.generated/resume-source.json`
- `.generated/resume-source.json` -> (`deriveResume`, pure, once per locale) -> `.generated/resume.de.json` / `.generated/resume.en.json`
- `.generated/resume.<locale>.json` -> web rendering (`src/lib/data/resume.ts` imports both generated files directly)
- `.generated/resume.<locale>.json` -> Typst-ready data (`scripts/build-typst-data.ts` imports the same generated files directly)

By default (`RESUME_MODE` unset, or `RESUME_MODE=masked`) real client identities never leave `generate-resume-source.ts`. `RESUME_MODE=unredacted pnpm build:json-resume` restores real values into the same gitignored `.generated/` files instead, for local, personal use (e.g. printing your own real CV) — never use this mode for a build that gets committed, deployed, or shared, and regenerate the masked version afterward before running `dev`/`build` again.

Guidelines:

- Anything that needs localized resume data should import `.generated/resume.<locale>.json` directly, the same way `resume.ts`/`build-typst-data.ts` do — never add another parallel decrypt path.
- Keep German and English resume structures aligned.
- Validate both localized outputs against the same TypeScript/schema model.
- Use stable `id` fields for translatable entries such as projects and talks.
- IDs should not change just because a title or URL changes.
- Derive technology durations from project date ranges, merging overlapping intervals so "years of experience" remains defensible.
- Derive reverse links from each technology to the projects where it appears.
- `codeVisibility` (`open-source`/`closed-source`) describes whether a project's *code* is public. It says nothing about whether the client/entity name is public — don't repurpose it for that. Only set it when there's an actual codebase behind the entry (e.g. a linked repository); a `type: "presentation"` entry whose only public link is a recording is not "open-source" just because the talk itself was public — omit the field rather than guess.

## Redacted Clients

Fields prefixed `sopsEncrypted` (`sopsEncryptedEntity`, `sopsEncryptedUrl`, `sopsEncryptedLinks`) hold real client identities. `sops` encrypts these fields in place in `resume.i18n.json` (`encrypted_regex: '^sopsEncrypted[A-Za-z]*$'` in `.sops.yaml`, `mac_only_encrypted: true` so plaintext edits elsewhere never break decryption). The goal is that no *current* build artifact — the live website, `/resume.json`, either PDF variant, or the current `resume.i18n.json` blob — ever exposes a real client name or a URL that would reveal one.

This is a scoped mitigation against low-effort CV/data farming (bots and scrapers reading the current tree or the deployed site), not a guarantee that these identities are unreachable in this public repo altogether. Earlier commits still contain the plaintext values in `resume.i18n.json` blobs from before they were encrypted, and that's an accepted tradeoff, not an oversight — a recruiter who goes digging through Git history is out of scope for this protection. **Do not rewrite Git history to purge old blobs** as a way to "fully" fix this, whether in response to review feedback or otherwise, without being explicitly asked; that's a judgment call for Robert, not something to do preemptively. If plaintext history turns out to be trivially discoverable by an AI agent in a way that matters, that's a reason to revisit later, not a bug to fix now.

Edit with `sops resume.i18n.json` or view with `sops -d resume.i18n.json`; `biome.json` excludes this file so it never fights `sops`'s formatting. `.sops.yaml` lists two age recipients: a personal key for local interactive edits, and a CI-only key (its private half lives in the `SOPS_AGE_KEY` GitHub Actions secret, attached only to the individual CI steps that actually invoke `sops` — never at job level, to keep it out of reach of unrelated steps like checkout or dependency install) so `pnpm build`/`pnpm check:quick` can decrypt in CI without ever exposing the personal key.

## PDF And Print

The PDF is a dedicated document output, not a browser printout of the website.

PDF priorities:

- Consistent pagination.
- Strong typography.
- Stable link presentation.
- Predictable output in CI.
- Locale-specific output matching the localized route.

Stable public PDF URLs:

- `/de/robert-schaefer-resume.de.pdf`
- `/en/robert-schaefer-resume.en.pdf`

Browser print is only a fallback. It should not try to reproduce the full CV layout once the Typst PDF exists.

PDF implementation rules:

- Keep Typst layout code separate from the SvelteKit app.
- Generate Typst data, not Typst template code.
- Keep checked-in Typst templates hand-written and modular.
- Do not use GitHub Actions artifacts as canonical public PDF URLs.
- Treat workflow artifacts only as CI inspection output when useful.

## Localization

Supported locales:

- `de`
- `en`

Routing should stay explicit and shareable:

- `/de`
- `/en`
- `/de/resume.json`
- `/en/resume.json`

Locale behavior:

- Infer the initial locale from the `Accept-Language` request header.
- Provide an explicit language switcher.
- Prefer URL-based locale persistence.
- Keep deterministic fallback behavior, with German as the default.
- Treat platform-level redirects as an enhancement, not the only way locale routing works.

Translate presentation copy and content summaries where needed:

- Navigation labels
- Hero and section copy
- CTA labels
- Project and talk summaries
- Page titles and descriptions

Dates, links, and technical keywords can remain language-neutral when that is clearer.

## Contact And Privacy

Contact options should be clear without creating unnecessary spam exposure.

Preferred approach:

- Keep email usable for humans and no-JS users.
- Avoid repeating the raw email address across the site.
- Consider a dedicated alias or server-side form only if spam becomes a real problem.
- Do not rely on JavaScript-only email reveal patterns.

Optional channels can include Signal or Mastodon when they are actively maintained.

Privacy defaults:

- Do not load third-party video iframes until explicit user action.
- Provide direct external links as fallbacks.
- Use contextual click-to-load behavior for embeds.
- Avoid generic consent walls.
- Explain relevant data processing in the privacy policy.

## Media Content

Keep canonical public talk/media links in resume data. Website-specific media metadata can live in a local TypeScript data file when needed.

Useful media fields:

- Stable `id` or `slug`
- Provider
- Event
- Date
- Description
- Thumbnail
- Duration
- Featured flag
- Embed mode

Guidelines:

- Join resume entries with website metadata by stable ID or slug.
- Keep provider-specific behavior in code.
- Keep metadata provider-agnostic where possible.
- Prefer prominent direct links when an embed adds little value.
- For PDF output, prefer readable domain-owned short aliases over long video URLs.

## Open Source Content

Open source should be curated as portfolio content, not dumped from an API.

Guidelines:

- Render open source content statically at build time.
- Separate raw GitHub facts from curated portfolio context.
- Include both owned/maintained repositories and notable third-party contributions.
- Do not fetch GitHub data client-side at runtime.

Useful fields:

- Repository name
- URL
- Description
- Primary language
- Contribution type
- Impact summary
- Featured flag
- Screenshots or supporting assets when useful

## Core Components

Expected UI building blocks:

- Layout, header, footer, and section wrappers.
- Language switcher.
- Hero.
- Resume PDF CTA.
- Tech experience visualization.
- Experience list.
- Project cards and optional project detail routes.
- Skills list.
- Open source section.
- Media link/embed cards.
- Contact module.
- Legal footer navigation.

Progressive enhancement requirements:

- Core content must render without JavaScript.
- Navigation, links, contact options, and content hierarchy must remain usable without JavaScript.
- Interactive technology animations must degrade to static readable content.
- Click-to-load media cards must always provide direct external links.
- The downloadable PDF must not depend on client-side rendering.
- Locale selection and locale-specific routing must work without JavaScript.

## Routing And Redirects

Keep route behavior predictable:

- `/de`
- `/en`
- `/` redirects or falls back to a localized route.
- `/de/resume.json`
- `/en/resume.json`
- `/resume.json` redirects or falls back to a localized resume JSON route.
- `/impressum` and localized legal routes.
- `/datenschutz` and localized privacy routes.
- Short aliases for print-friendly links on the primary domain.

Short links should be human-readable and owned by the domain. Keep definitions in `src/lib/data/short-links.ts` and `netlify.toml` aligned.

## Quality Bar

Accessibility:

- Semantic HTML.
- Keyboard navigation.
- Clear heading hierarchy and landmarks.
- Accessible names for links, buttons, media controls, and external destinations.
- Sufficient contrast.
- No essential information conveyed only through color, motion, hover, or animation.

SEO and metadata:

- Localized titles and descriptions.
- Canonical URLs.
- `hreflang` tags.
- Open Graph tags.
- Twitter card tags.
- Social preview image.

Performance:

- Static generation.
- Minimal JavaScript.
- Lazy-load non-critical images.
- No third-party video requests before explicit activation.

Testing:

- Unit test data mapping, duration calculations, content joins, and provider/embed decisions.
- Unit test Typst data generation.
- Test locale preference parsing and localized resume parity.
- Test short-link parity between TypeScript data and `netlify.toml`.
- Use accessibility tests for rendered pages where practical.
- Verify generated PDFs in CI for both locales.

## Commands

Install dependencies:

```bash
pnpm install
```

Start development server:

```bash
pnpm dev
```

Every `build:*` script does exactly one job and can be run standalone; `pnpm build` runs them all in the order a full production build needs:

```bash
pnpm build:paraglide     # compile Paraglide UI messages
pnpm build:json-resume   # decrypt + mask resume.i18n.json -> .generated/resume.{de,en}.json (gitignored, requires a sops key)
pnpm build:typst-data    # -> typst/content/{de,en}.typ.json (implies build:json-resume)
pnpm build:pdf           # -> static/{de,en}/*.pdf (implies build:json-resume; internally also runs build-typst-data.ts)
pnpm build:vite          # vite build -> build/
pnpm build:sitemap       # crawl build/ -> build/sitemap.xml, build/robots.txt
pnpm build               # build:json-resume -> build:pdf -> build:paraglide -> build:vite -> build:sitemap
```

Verification, from fastest to slowest:

```bash
pnpm check:types  # svelte-check only
pnpm check:quick  # paraglide + resume source + lint + check:types + unit tests - what CI's "Quick verification" step runs
pnpm check        # check:quick + test:e2e - test:e2e's own webServer already runs a full pnpm build (incl. build:pdf) to serve the site, so a working build is proven without a separate build step; slow, don't run this reflexively
```

For documentation-only changes, `git diff --check` is usually enough.

## Git Worktree And Subtree Branch Model

Create or reuse a feature worktree from the folder that contains the bare repo `.git` directory:

```bash
git fetch origin
git worktree list
```

If `<feature>` is already listed, reuse it. Otherwise create it:

```bash
git worktree add <feature>
```

Work in the feature worktree:

```bash
cd <feature>
pnpm install
pnpm check:quick
pnpm test
```

Keep pull requests focused on one coherent change:

- Do not let unrelated edits sneak into a pull request.
- If you notice useful follow-up work that is not required for the current change, leave it out of the current branch.
- Create a separate feature branch and pull request for unrelated changes.
- Prefer this split even for small cleanups because it keeps `git bisect` useful and makes `git blame` easier to reason about.

Keep the branch up to date with `main`:

```bash
git fetch origin
git rebase origin/main
```

When asked to squash commits since `origin/main`, first rebase the branch onto the current `origin/main`, then squash only the commits that still remain on top:

```bash
git fetch origin
git rebase origin/main
git log --oneline origin/main..HEAD
```

Do not use `git reset --soft origin/main` before rebasing onto the current remote base. If `origin/main` advanced and contains equivalent or related commits, a reset can stage unrelated base changes that a normal rebase would have dropped or reconciled. If this happens, use `git reflog` to find the pre-reset commit, reset back to it, then rebase normally.

Before considering the work ready for `git push`, run `pnpm check:quick`. This script is also used by CI (as its "Quick verification" step) and should stay in sync with it — the quick, inexpensive checks, not the full `pnpm check` suite. It should keep successful output concise and print the current check name plus the failing command's output when a check fails.

Publish the branch:

```bash
git push -u origin <feature>
```

## Commit And PR Messages

When a coding agent makes repository changes, commit the completed agent-authored changes before finishing unless the user explicitly asks to leave them uncommitted.

Use Conventional Commits for commit subjects:

```text
<type>(optional-scope): <summary>
```

Examples:

```text
docs: replace planning notes with agent guidance
feat(resume): add open source contribution metadata
fix(i18n): preserve explicit locale selection
```

Pull request commits are squashed before merge, so the final commit message matters. Write the squashed commit message for a future `git blame` reader:

- Use the subject to summarize the behavioral or documentation change.
- Use the body to explain why the changed lines should exist, not what changed. The diff already shows what changed.
- Preserve the user's stated intention in the commit body when it was explained in chat with the coding agent. Capture the problem they wanted solved, the UX or maintenance goal, and any rejected alternatives that clarify why this approach exists.
- Include the motivation, context, constraints, and tradeoffs that are not visible from a single blamed line.
- Mention relevant verification commands and any skipped checks.
- Keep the PR description in sync with the intended squash commit message.
- After changing the commit message during review, update the PR description to match.

## Before Finishing

- Run the narrowest useful verification for the files changed.
- Check `git status --short` and confirm the diff contains only the intended edits.
- Mention any command that could not be run, especially if the change touches generated content, tests, or deployment behavior.

---
> Source: [roschaefer/roschaefer.de](https://github.com/roschaefer/roschaefer.de) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
