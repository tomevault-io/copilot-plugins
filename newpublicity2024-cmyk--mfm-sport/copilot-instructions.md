## mfm-sport

> Arabic-language Moroccan football news site. Next.js 16 (App Router) + Payload CMS 3 on Neon Postgres (Frankfurt), deployed on Vercel.

# MFM Sport — working notes

Arabic-language Moroccan football news site. Next.js 16 (App Router) + Payload CMS 3 on Neon Postgres (Frankfurt), deployed on Vercel.

---

## Session state — SEO remediation

**Updated: 28 July 2026, phase boundary — 2024 import batch in flight.** Update this at every phase boundary. It is deliberately ground truth on disk rather than in a conversation summary.

### Resume here

**The 2024–2026 staged release is IMPORTED. Next action is yours, not the code's: review Search Console before importing any older year.**

| Batch | created | archive-full | archive-brief | video | no-date | empty | failed |
|---|---|---|---|---|---|---|---|
| 2024 | 6,570 | 5,706 | 864 | 0 | 0 | 0 | 0 |
| 2025 | 1,509 | 1,285 | 224 | 1 | 0 | 0 | 0 |
| 2026 | 463 | 461 | 2 | 0 | 0 | 0 | 0 |
| **total** | **8,542** | **7,452** | **1,090** | 1 | 0 | 0 | **0** |

Every batch matched its independently-audited prediction exactly — thin counts of 864 / 224, and 2026's `463 + 101 already-imported = 564`. Two code paths, same numbers.

Database after the release: **8,940 articles** (8,542 imported + 398 `editorial`), **8,742 redirects** (= 200 + 8,542), **0** null publish dates, **0** articles older than 2024. The staged boundary held; nothing leaked.

Spot-checked 10 evenly-spaced articles per batch against the XML — **30/30 at ratio 1.00**, dates and redirects correct (`pnpm spotcheck --year=<Y>`).

**BLOCKED on a deploy:** `/sitemap.xml` still serves the pre-import 906 URLs / 398 articles, byte-identical to before. It caches for 24h and the route that busts it is in this branch, undeployed. Once deployed, POST `{"collection":"sitemap"}` to `/api/revalidate` — or the importer now does it automatically at the end of a run. Expect the sitemap to go to roughly **7,850** article URLs (398 editorial + 7,452 released `archive-full`); `archive-brief` must stay out.

`/news-sitemap.xml` is verified still at **13 URLs** — the archive did not leak into the 48h feed.

```bash
# after this branch deploys, confirm the release is advertised:
curl -s https://www.mfmsport.ma/sitemap.xml | grep -o '<loc>' | wc -l

# then STOP. Do not import 2023 or earlier — see Hard gates.
```

**Still unfixed:** `<html>` carries no `lang`/`dir` (see open defects).

### Merged and deployed

| PR | What |
|---|---|
| #52 | Real 404s, ad-free error pages, `www` canonical, 307→308, Arabic description, match whitelist, archive importer |
| #53 | Reverted sitemap sharding (it served zero URLs and 404'd `/sitemap.xml`) |
| #54 | hreflang → Arabic only, canonicals on all page types, `docs/verification-principles.md`, this file |
| #55 | Upstream API failure no longer serves 404 for fixtures that exist |

All four are **live on production and verified against the served bytes** (deployment `dpl_9mR6QBDznjofSoHUSSn9xbafdG1i`):

- `/ar/transfers` and a missing article slug → **404** (real, not soft)
- apex → `/ar` → **308**
- `/sitemap.xml` → 200 with **906 `<loc>`**, of which 398 are articles
- `/news-sitemap.xml` → 200 with **13 `<loc>`** — the 48h window is holding, not leaking the archive
- article page → exactly **2** hreflang alternates, `ar-MA` + `x-default`, both `/ar`; canonical on `www`; no `robots` meta (indexable)
- 404 page → **0** real `adsbygoogle.js` script tags

### Redirect map — repaired and verified end-to-end

All 200 rows were stored in the dead format (100% had *both* a trailing slash and lowercase hex). `pnpm redirects:normalize` rewrote all 200; **0 duplicate collisions**.

Verified on the artefact, not the table:

| Check | Result |
|---|---|
| `/api/redirects?from=…` — the exact request middleware makes | **200/200** return the correct target |
| Stale `{to: null}` cached by the CDN before the repair | **0** (the deploy reset the cache) |
| Destination article URLs | **200/200** return HTTP 200 |
| One full legacy chain, end to end | `308` → `301` → `200` |

**Breakdown of the 200: all 200 → live published article.** Zero category-hub fallbacks, zero dead targets.

`pnpm redirects:verify` (`scripts/verify-redirects.ts`) re-runs the whole check any time. It probes the lookup endpoint rather than the legacy URLs, deliberately: fetching all 200 legacy URLs would consume the untouched sample someone may want for an independent spot-check.

The legacy URL consumed for the end-to-end chain test was **`/الجامعة-تبرم-اتفاقية-شراكة-مع-المكتب-ا`** (row 356). The other 199 are untouched.

### Database

Archive-fields DDL is **applied to production** (`broad-snow-50246164` / branch `br-royal-wildflower-a21skzaw`): `wp_post_id`, `legacy_slug`, `seo_tier`, three indexes, `payload_migrations` batch 8. All pre-existing articles default to `editorial`, so they stay indexable.

Verification branch **`br-gentle-hat-a2bzeay0`** is alive deliberately — keep it until the full import is done. It holds ~2,378 imported archive articles and 2,178 normalised redirects, and is useful to diff against.

**The 2024 import COMPLETED against production.**

| | |
|---|---|
| created | 6,570 (exactly the corpus-predicted count) |
| `archive-full` / `archive-brief` | 5,706 / 864 |
| skipped as already-imported | 101 (the backfill working) |
| redirects created | 6,570 |
| failures / zero-date / empty bodies | **0 / 0 / 0** |
| imported date range | 2024-01-03 → 2024-12-11 (confirms `--max-year`) |

Two invariants to re-check after each batch: `redirects - 200 == imported`, and `archive-full + archive-brief == imported`. Both mean one redirect and one tier per article, with nothing silently dropped.

The strong result is not the percentage: `audit:body-length` independently predicted **864** thin posts for 2024, and the importer produced **864** `archive-brief`. Two code paths, same number. An earlier reading of 21.2% was a partial-sample artifact of a chronological export.

**A backfill was required before any import could run safely.** All 398 pre-existing articles had `wp_post_id` NULL, so the importer's checkpoint could not see them and would have re-created every one at `<slug>-2` — indexable, sitemap-listed, and with no redirect, since the legacy redirect already points at the original. `pnpm backfill:wp-ids` matched 101 via the redirect map; the other 297 postdate the export and cannot collide. The proof those numbers are complete: exactly 101 articles have `published_at` on or before the export date.

### Import order

**DDL → deploy → normalize → import.** All of DDL, deploy and normalize are done. The order matters: without `lib/seo/indexation` deployed, every imported article is immediately indexable and listed in the sitemap, which defeats the staged release.

See **Resume here** at the top for the next command.

### Open defects — found, not yet fixed

- **`<html>` carries no `lang` and no `dir`.** Production serves `<html data-dpl-id="…">` on every page type including the 404. `dir="rtl"` / `lang="ar"` are set on an inner `<div>` in `[locale]/layout.tsx:44` instead. `<html lang>` is the language signal Google and screen readers read first, and the missing `dir` means anything rendered outside that div — the 404 page among them — lays out left-to-right on an Arabic site. The `<html>` element lives in `(frontend)/layout.tsx`, above `[locale]`, which is why it never got the locale; now that the site is Arabic-only it can simply be hardcoded. Small fix, real signal.

- **A real mouse click computed from a paragraph's bounding box can collapse the caret to the paragraph's END instead of the click point, in the admin's RTL Lexical editor.** Observed while building Task 8's browser verification (`scripts/verify-toolbar-admin-ux.ts`): clicking at a `<p>` element's horizontal midpoint — the natural way to target "the middle of a paragraph" from outside the editor — landed the caret after the last character instead, because the box spans the full editor column width while short, right-aligned Arabic text only occupies part of it, leaving the midpoint in empty margin. Unresolved rather than benign: this is ordinary CSS box-model hit-testing, not headless-specific and not a Lexical bug, so it would reproduce in any real browser a journalist uses — and it was only worked around (placing the caret via the native Selection API at an exact text-node offset), never re-tested against a coordinate actually known to sit on a rendered glyph. Whether a real click on visible Arabic characters (as opposed to a paragraph's empty margin) behaves correctly is still open. Would be settled by clicking at the centre of a rect from `Range.getClientRects()` on the text node and checking where the caret lands. Block insertion at a programmatically-placed caret is verified correct (Task 8); this is about click targeting only, not insertion.

### Open blockers

- **Ahrefs connector not authorised** — blocks referring-domain data, which would set import *order* (highest-value URLs first). Does not block the import itself.
- **Vercel connector scoped to the wrong account** (`lallafatimamagazine-4500s-projects`, not `newpublicitys-projects`) — blocks the ASN/user-agent breakdown behind the WAF rules. GA4 says >50% of traffic is datacenter-region bots.
- **API-Football daily quota exhausted** — match pages return null upstream. Blocks verifying that a *whitelisted* fixture is indexable; the non-whitelisted `noindex` case is verified.
- **`wp-content/uploads` backup** — owner is checking. All 43,584 legacy images already 404; the WordPress REST API is gone. Bodies import with `<img>` stripped. Images can be backfilled later against `legacy_slug` without re-importing text.

### Agreed plan

Tiering at 500 characters of body text: **26,982** `archive-full`, **10,006** `archive-brief`, **4** genuinely empty. Import everything, 301 everything, but stage *indexation*: 2024–2026 first (~8,600), then pause for Search Console before releasing older years. `archive-brief` stays `noindex, follow` indefinitely.

---

## Verification

**Read [`docs/verification-principles.md`](docs/verification-principles.md) before claiming that anything works.**

> Assert on the artefact a crawler or user actually receives — not on a proxy for it.

That document is not general advice. It is five specific bugs this codebase shipped, each of which had already been "verified" by something that looked like evidence: a row count, a green build, a `grep -c`, a computed statistic. They were found only by fetching the bytes a client would actually get.

The short version, when you don't have time for the long one:

- A green build is not a behavioural assertion. Fetch the URL and count the elements.
- Check the HTTP **status**, not just the rendered body. A page can render correctly and respond wrongly.
- `grep -c` counts matching *lines*. Minified HTML is one line. Use `grep -o | wc -l`, and match the parsed construct — a string can appear in an RSC payload without being a real tag.
- A dry run validates parsing, not writing. Run it for real against a throwaway Neon branch.
- Say "untested" rather than "works". Your phrasing becomes the next person's premise.

---

## Hard gates

**2023-and-earlier batches MUST NOT run until `pnpm audit:body-length --year=<Y>` reports `MIS-TIERED to archive-brief: 0` for that year.**

Gate on **MIS-TIERED**, not on "posts where they disagree". The disagreement count compares the importer's total against a plain-text measure that *ignores ACF entirely* — so when the ACF read returns nothing, both measures agree on the same wrong answer and the count stays 0 while articles are silently mis-tiered. It happened to be non-zero for 2021, but incidentally, and it does not measure the harm.

**2021 is confirmed affected: 417 posts under-counted, 183,545 characters lost, 246 full articles mis-tiered into `archive-brief`** — which is never released, so they would be `noindex` permanently. 2022 and 2024 measure clean.

The mechanism: `readTag()` matches `<tag>…</tag>` on a *single line*. ACF flexible-content values are multi-line HTML, so `readTag` returns `null`, the value contributes **0 characters**, and `lastMetaKey = null` discards the remainder. A full article scores under 500, lands in `archive-brief`, and is `noindex` indefinitely — with no error, no failed row, and nothing in the import summary to distinguish it from a genuinely short post.

This is verified *absent* from 2024, 2025 and 2026 (zero disagreements in all three, and all three match `docs/wp-corpus-analysis.md` to the decimal). It is verified *present* in the corpus as a whole. Since the per-year disagreement count is zero across the whole released window, all 651 sit in earlier years — most likely 2021–2022, which is 53% of the corpus.

Run the audit per year. If MIS-TIERED is non-zero, fix the multi-line read in `scripts/import-wp-archive.ts` (~line 258) **before** importing that year — re-tiering after the fact is a bulk write, which is exactly what the tier field exists to avoid.

Measured per year so far:

| Year | under-counted | chars lost | mis-tiered | verdict |
|---|---|---|---|---|
| 2026 | 0 | 0 | 0 | clean |
| 2025 | 0 | 0 | 0 | clean |
| 2024 | 0 | 0 | 0 | clean |
| 2022 | 0 | 0 | 0 | clean |
| **2021** | **417** | **183,545** | **246** | **BLOCKED — fix first** |

2023, 2020, 2019 and 2010 are unmeasured. Run the audit before each.

## Landmines

**Never add a `loading.tsx` to a route segment that has 404-capable children.** Its Suspense boundary flushes the response shell before the page body runs, committing HTTP 200 — so `notFound()` renders its page inside an already-successful response and every 404 on the site silently becomes a soft 200. This happened; see the principles doc. `/search` has the only `loading.tsx`, and it has no child routes and never calls `notFound()`.

**Raise `notFound()` in `generateMetadata`, not only in the page body.** Metadata resolves before the response streams, so the 404 status can still be set there.

**All ads live in `[locale]/(site)/layout.tsx` and nowhere above it.** `not-found.tsx` is a sibling of that route group, so raising `notFound()` discards the whole ad-bearing subtree. This makes "no ads on error pages" a property of the tree rather than a rule to remember — Google prohibits ads on screens without publisher content. If you move the AdSense loader up a level you silently reopen that exposure; a merge conflict already tried to. Verify with a real `<script src=…adsbygoogle>` tag count on a 404, not a grep.

**Legacy redirect lookups must go through `normalizeLegacyPath()`.** WordPress wrote permalinks in lowercase percent-encoding with a trailing slash; the platform normalises requests to uppercase without one. The map is an exact string match, so unnormalised lookups match nothing — this is why the original 200-row redirect map never fired once.

**`src/migrations` is gitignored, and `payload migrate` warns of data loss on this database** (it detects dev-push drift and would reconcile against a stale snapshot). Apply DDL by hand — the statements live in `docs/archive-import-runbook.md`.

**Percent-decode before stripping tags.** `<![CDATA[` opens with `<` and `]]>` closes with `>`, so `/<[^>]+>/` swallows an entire CDATA section as one "tag". This produced a fake "6% of the archive is empty" statistic.

**Any future re-tier of admin-authored articles must go through the block-aware path.** `tierFor()` in `src/lib/seo/wpArchive.ts` has no caller today except the raw-HTML WordPress import (`scripts/import-wp-archive.ts`) — it has no notion of a block, so the naive text walk scores media blocks (`socialEmbed`, `gallery`, `audio`, `embedFrame`, or a bare inline image) at zero and will silently noindex media-heavy articles. Use `tierForLexicalBody()` in `src/lib/seo/blockAwareTiering.ts` instead. This exposure is currently *latent* — nothing calls `tierForLexicalBody` yet either, since admin-authored articles default to `seoTier: "editorial"` and nothing recomputes it — but a guard nobody calls is a guard that does not exist, so this is written down before the first bulk re-tier tool gets built, not after.

**If a CSP is ever introduced,** `frame-src` must include `www.facebook.com`, `www.instagram.com`, `www.youtube-nocookie.com`, `platform.twitter.com`. There is no Content-Security-Policy anywhere in this codebase today (confirmed by reading `next.config.ts` and `src/middleware.ts`), so nothing enforces this yet — but the day someone adds one without these four hosts, every social/video embed on the site dies **in production only**, the worst possible failure timing and this project's signature failure mode.

---

## Key documents

| Document | What it covers |
|---|---|
| `docs/verification-principles.md` | How to check that something works. Read first. |
| `docs/seo-recon-findings.md` | Ground truth vs. the external SEO audit, with corrections |
| `docs/traffic-integrity-findings.md` | GA4 analysis: >50% of page views were error pages |
| `docs/wp-corpus-analysis.md` | The 36,992-post WordPress export: tiering and the decision behind it |
| `docs/archive-import-runbook.md` | DDL, batched import, staged indexation release |

---

## Conventions

- Arabic is the only served locale. `/fr` and `/en` 301 to `/ar`; Payload still stores those translations and the change is reversible.
- All user-facing SEO text — titles, descriptions, schema, alt text — is Arabic. English boilerplate is a bug.
- `SITE_URL` (`src/lib/seo/siteUrl.ts`) normalises to the `www` origin regardless of the env var. Don't build URLs from `process.env.NEXT_PUBLIC_SITE_URL` directly.
- Archive articles are released into the index in batches via `src/lib/seo/indexation.ts`. Releasing a batch is a config edit and a deploy — never a re-import or a bulk DB write.
- The sitemap and the `robots` meta tag must always agree. Listing a `noindex` URL in a sitemap is a contradictory signal.
- Any component with directional behaviour — carousels, scroll math, chevrons, slide transitions, swipe — carries an RTL assertion in its test file as a matter of course. A missing one is a review finding. On an Arabic-only site this class of bug is guaranteed to recur and is invisible to anyone reading the code in English. (Third occurrence: the Gallery carousel button swap, after PRs #30 and #31 — see `docs/superpowers/review-checklist.md`.)

---
> Source: [newpublicity2024-cmyk/MFM-sport](https://github.com/newpublicity2024-cmyk/MFM-sport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
