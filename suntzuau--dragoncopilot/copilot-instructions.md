## dragoncopilot

> Australian reference site for Microsoft Dragon Copilot and Dragon Medical One.

# dragoncopilot.com.au — build and content rules

Australian reference site for Microsoft Dragon Copilot and Dragon Medical One.
Published by Voice Recognition Australia, an authorised Australian reseller.

## What this site is for

Citation, not clicks. The primary channel is answer engines — ChatGPT, Claude, Gemini,
Perplexity — quoting these pages as the source on Dragon Copilot questions. Ranking in
Google is the secondary benefit. Every structural decision follows from that.

The commercial thesis: Dragon Copilot has no confirmed Australian general availability as
at 25 August 2026. Building the accurate, dated, well-sourced Australian reference now means
owning the answer before the demand exists.

## Non-negotiable build rules

1. **Zero client-side JavaScript on content pages.** No React, no Vue, no hydration.
   If a fact needs JS to appear, it does not exist to a crawler.
2. **Nothing behind interaction.** No accordions, tabs, modals or "read more".
   Content that requires a click is content that does not get cited.
3. **Real `<table>` markup.** Never CSS-grid divs. The evidence tables are the most
   citable asset on the site; parsers must be able to read them as tables.
4. **Every page answers a question.** The H1 is the question. The `answer` frontmatter
   field is 40–55 words, states the answer directly, and carries no preamble.
5. **Every page carries a verification date.** Rendered on the page and emitted as
   `dateModified`. An undated claim is worthless to an answer engine and to a buyer.
6. **No pricing figures.** Deliberate editorial decision. Licensing *mechanics* — Per User
   versus Flex, session definitions, pooling, rollover — are covered in full; dollar
   amounts are not. Pricing goes stale and Microsoft routes numbers through partners.
   Pages end in a request-a-quote call to action instead.

## Evidence rules

- Zero tolerance for unverified claims stated as fact. Every factual assertion needs a
  numbered source in `src/data/sources.json`.
- `unknown` / `unverified` is a publishable answer. Guessing is not. Being the site that
  says "no primary source confirms this" is a large part of why an engine will trust it.
- **Never conflate `native` with `compatible`.** A native integration is vendor-built and
  supported. Compatible means dictation types into a text field. Competitors blur this.
- Named-customer and named-hospital claims are the highest-risk content here. No
  organisation's deployment is described without a primary source.
- Do not claim Dragon Medical One is end-of-life. Only Dragon Professional Anywhere and
  Dragon Legal Anywhere are.

## Content architecture

- Four hubs: `product`, `australia`, `compliance`, `evidence` — set in `src/site.config.json`.
- Reference pages: markdown in `src/content/docs/`, routed at `/<slug>/`.
- Australian EMR systems: data files in `src/content/emr/`. One record drives both the
  per-system page and the comparison matrix at `/emr/`. Update the record, never the page.
- Government and hospital networks: `src/content/institution/`.
- `llms.txt` is generated from the collections at build time. Never hand-edit it.

## Components

| Component | Use |
|---|---|
| `AnswerBlock` | The extractable answer. One directly under the H1, one under each FAQ question. |
| `EvidenceTable` | Semantic table in a scroll container. Always give it a `caption`. |
| `StatusPill` | Integration status. Uses the shared vocabulary in `content/config.ts`. |
| `Callout` | `note`, `warn`, `stop`. Use `warn` for anything time-sensitive. |
| `SourceList` | Renders numbered sources from frontmatter `sources: [ids]`. |

## Deployment

GitHub → Cloudflare Pages, matching the rest of the VRA network.
Build command `npm run build`, output directory `dist`.

## Before adding cross-network links

Check `SuntzuAU/seo-authority-register` — `DECISIONS.md` first, then the rules.
3 links/month velocity cap, 60-day donor cooling, 30% anchor diversity limit, no UTM
parameters on cross-network links.

## Network standards — these override anything above

This repo is part of the VRA gateway network and follows the same rules as the other
SuntzuAU site repos. The canonical copies live in `astro-gateway-master`; the copies in
`.claude/` here are kept identical.

- `.claude/CONTENT-GUIDELINES.md` — ACL compliance, voice, competitor comparisons,
  healthcare claims. **Read before writing any copy.**
- `.claude/INTERLINK-RULES.md` — no footer links ever; 1–3 internal and 1–2 external links
  per article; body links must match frontmatter declarations.
- `.claude/INFRASTRUCTURE.md` — deployment checklist, GA4 IDs, Cloudflare and R2 config.
- `.claude/IMAGE-STANDARDS.md` and `.claude/IMAGE-GENERATION.md` — image sizes and the
  Worker generation workflow.
- `.claude/BUILD-RULES.md` — Astro architecture, performance, YouTube facade pattern.

Additional constraints that bite on this site specifically:

- **Healthcare claims.** Never imply the software improves patient outcomes, provides
  clinical advice, is a medical device, or is clinically validated. Microsoft's own
  UKCA Class I marking in Great Britain is a fact about a jurisdiction, not a claim
  this site may generalise.
- **Authority claims** come only from the `authority` block in `src/data/link-network.json`.
  Never improvise credentials or years of experience.
- **`[VERIFY]`** is the network convention for an unconfirmed claim. Used throughout the
  `emr` and `institution` collections. It is publishable; an invented status is not.
- **Approval workflow applies.** Draft in chat, flag `[VERIFY]` items, get approval,
  then commit. Do not commit content Russ has not seen.

---
> Source: [SuntzuAU/dragoncopilot](https://github.com/SuntzuAU/dragoncopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
