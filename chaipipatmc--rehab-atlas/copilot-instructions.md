## rehab-atlas

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# RehabAtlas Agent Guidelines

## shadcn/ui v4 Differences

This project uses shadcn/ui v4 which is built on `@base-ui/react` (NOT Radix). Key differences:
- Button uses `@radix-ui/react-slot` for `asChild` prop (custom implementation)
- Select wraps `onValueChange` to filter null values
- Dialog/Sheet use `@base-ui/react/dialog` with different prop names
- Accordion has no `type="single"` or `value` on items

## Design System: "The Quiet Authority"

- Primary color: `#45636b` (deep teal)
- Font: Noto Serif (headings) + Inter (body)
- No hard borders — use `ghost-border` (subtle box-shadow) or surface color shifts
- Buttons: `rounded-full` pill shape with `gradient-primary` for CTAs
- Cards: `rounded-2xl` with `shadow-ambient`
- Nav: `glass-nav` (glassmorphism backdrop-blur)
- Transitions: 300ms, never bouncy

## Critical Patterns

1. **Supabase clients:**
   - `src/lib/supabase/client.ts` — browser
   - `src/lib/supabase/server.ts` — server (with cookies)
   - `src/lib/supabase/admin.ts` — service role (bypasses RLS)

2. **Leads MUST use service role** — never insert leads from client-side

3. **Partner edits create `center_edit_requests`** — never update centers directly

4. **Blog author tracking** — `author_type` field: 'rehabatlas' or 'partner', with `author_center_id` for backlinks

5. **Commission fields are dormant** — `commission_type`, `commission_rate`, `commission_fixed_amount` on `centers` and `lead_forwards` remain in the schema but are not populated or referenced in active flows under current pricing-deferred policy (see CLAUDE.md business rule 3). Do not gate lead forwarding on them.

6. **No pricing/commission in partner-facing comms (as of 2026-05-17)** — outreach emails, follow-ups, response-handler replies, onboarding messages, and partnership agreements must **not** mention commission, referral fees, tier discounts, or launch-campaign pricing. Position partnership as free listing + admin-vetted leads + author backlinks only. Any future fees will be agreed in writing later and applied forward-only.

7. **Outreach pipeline** — `outreach_pipeline` table tracks center recruitment stages (new → researching → outreach_drafted → outreach_sent → responded → terms_agreed → agreement_sent → active)

8. **Gmail API** — outreach emails sent via direct fetch calls to Gmail REST API (NOT googleapis SDK). Token refresh handled manually. All emails CC'd to info@rehab-atlas.com

9. **PandaDoc** — partnership agreements use template `Ctzua6xmeLzCVnMwsmYR9L` with tokens for center details

10. **Content Creator** — auto-generates blog articles via Claude AI with Unsplash images. 100+ predefined SEO topics across 10 categories. Runs weekdays only. Drafts require admin approval before publishing. Before saving, `auto-linker.ts` injects internal links to `/rehab/[condition]`, `/rehab-in/[country]`, and `/rehab-in/[country]/[city]` hubs (first occurrence only, capped at 6 per article, skips existing links/images/headings/code).
    - **Family-first voice (default):** System prompt instructs Claude to write for the family member doing the research (~70% of inquiries come from families, not patients). Use second-person "you/your loved one" by default. Three dedicated family categories: `family-recognition` (warning signs), `family-decision` (choosing rehab), `family-during-after` (during stay + post-discharge).
    - **Pillar architecture (per [CONTENT_STRATEGY.md](CONTENT_STRATEGY.md) §3):** Every blog post maps to one of the 10 `/rehab/[condition]` pillar pages via `inferPillar(topic, category)` in content-creator.ts. The system prompt names the pillar explicitly and requires Claude to link back to it in the first 250 words. If Claude misses the link, `autoLinkArticle()` force-inserts one via `forcePillarLinkInFirstParagraph()`. The `pillar` field is recorded in Claude usage logs.
    - **Editorial guidelines** are documented in [CONTENT_STRATEGY.md](CONTENT_STRATEGY.md): voice, banned phrases, title formulas, hook style, FAQ section format, internal linking rules, pre-publish checklist. Update that doc first; the system prompt encodes its rules.

11. **Lead outcome tracking** — `lead_forwards.partner_status` (pending/admitted/not_admitted). Partners update at `/partner/leads`. `/partner/commission` and `/admin/commission` pages remain in the codebase for future use but are not promoted to partners while pricing is deferred (see rule 6)

12. **Comparison pages** — `/compare/[slug]` where slug = `center-a-vs-center-b` (separator `-vs-`, supports 2-3 centers). Top ~100 same-country pairs pre-rendered via `generateStaticParams` at build time; other valid combos served via ISR (`revalidate = 86400`). Each page emits `FAQPage` + `ItemList` JSON-LD for AI search citation. Center detail pages auto-link to 3 same-country comparisons as "Compare with Similar" section. Legacy `/compare?ids=` route still active for the saved-list compare flow.

13. **Programmatic City + City × Condition pages** — `/rehab-in/[country]/[city]` (e.g., `/rehab-in/thailand/chiang-mai`) and `/rehab-in/[country]/[city]/[condition]` (e.g., `/rehab-in/thailand/chiang-mai/alcohol-addiction`). Both routes use `generateStaticParams` derived from published-center data — they only emit URLs that have ≥1 listing, so no empty pages. ISR 24h. The 10 condition definitions are duplicated across `/rehab/[condition]`, the city × condition route, and `sitemap.ts` — keep all three in sync. `cityToSlug` in `src/lib/utils.ts` (alias of `countryToSlug`) handles the slug normalization.

14. **AISO blog FAQ extraction** — `extractFaqs()` in `src/app/blog/[slug]/page.tsx` parses the `## Frequently Asked Questions` section + `### Question` subheadings from article markdown into `FAQPage` JSON-LD. The Content Creator prompt instructs Claude to end every article with that exact section structure — do not change the heading wording without updating the parser. `ArticleJsonLd` also accepts a `reviewedBy` prop; editorial (non-partner) articles include "Rehab-Atlas Clinical Review Team" for YMYL E-E-A-T.

15. **Data Verifier agent + verification fields** — `data_verifier` agent (`src/lib/agents/data-verifier.ts`) runs nightly (`30 2 * * *` UTC) on a batch of published centers. For each center: scrapes the official website (homepage + 5 subpages via shared `src/lib/agents/scrape.ts`), asks Claude to compare DB fields vs site text, and verifies each `center_photo` two ways — SHA-256 / host match against `<img>` URLs on the site (using `extractImageUrls`) **and** a Claude Vision plausibility score via `analyzeImageWithClaude` in `src/lib/agents/claude.ts`. Writes results into `center_photos.verification_status` (`unverified`/`verified`/`suspicious`) and `centers.data_verification_status` (`unverified`/`verified`/`issues_found`/`no_website`). Always flag-only — admin clicks "Apply suggestions" in the email or `/admin/agents` to delete suspicious photos and mark the center verified (handled in `src/app/api/agents/action/route.ts` under `case "data_verifier"`). Tunable knobs (`site_settings`, exposed at `/admin/agents`): `batch_size`, `recheck_days`, `vision_threshold_verified`, `vision_threshold_suspicious`, `max_image_bytes`. Centers with no `website_url` are flagged `no_website` and skip all Claude calls.

---
> Source: [chaipipatmc/Rehab-Atlas](https://github.com/chaipipatmc/Rehab-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
