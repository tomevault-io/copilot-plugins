## local-seo-agent

> Nine Claude Code commands that take a local business from an empty Google Business Profile to a live website, ranked pages, and social posts that publish themselves.

# Local SEO Agent

Nine Claude Code commands that take a local business from an empty Google Business Profile to a live website, ranked pages, and social posts that publish themselves.

**Start here:** run `/gbp-build`. Everything else assumes the profile exists.

---

## The commands, in the order they are meant to run

| # | Command | What it does |
|---|---|---|
| 1 | `/gbp-build` | Fills the whole Google Business Profile: 10 categories, 50 services ranked by search volume, 20 products, the 750-character description, hours, attributes, service area, and the citation list |
| 2 | `/reviews` | Writes the review request sequence and the reply templates, and wires the automation so every finished job asks |
| 3 | `/gbp-post` | Generates a month of profile posts, queues them, and drips them 2 to 3 a week |
| 4 | `/keyword-research` | Builds the keyword map: every term, filtered, clustered, and routed to a page type |
| 5 | `/service-page` | One money page per service and area pair, built from the map |
| 6 | `/blog-post` | One local blog post, written to rank and to link down to a money page |
| 7 | `/build-website` | A full Next.js site if there is no website yet |
| 8 | `/publish` | Deploys: GitHub, Vercel, robots, sitemap, and the Search Console submission |
| 9 | `/blotato` | Turns each published blog post into native social posts and schedules them |

`/gbp-build citations` runs just the citation campaign if that is all you need.

---

## What you need

Nothing to start. Every command asks for what it needs the first time it runs, and records the answer so it never asks twice.

As you go, some commands will want:

- **Semrush** for real search volumes. Without it, volumes are marked as estimates.
- **A Make.com webhook** so `/gbp-post` can publish to the profile. The Business Profile API needs manual approval and takes weeks; Make is free and works immediately.
- **A Pexels API key** for stock photos. Free, no card.
- **GitHub and Vercel logins** for `/publish`. One-time, then every future change is one push.
- **A Blotato key** for `/blotato`. Paid-only, so skip it if you are not scheduling social posts.

Put them in `.env`. Copy `.env.example` to start.

---

## How the repo is laid out

- `.claude/commands/` — the nine commands
- `references/` — the specs each command reads: templates, checklists, the citation tiers, the on-page rules
- `code/` — the gates that check work before it ships, plus the stock photo fetcher and the scheduled publishers
- `design/` — the design kit `/build-website` builds from
- `website/` — the Next.js site, empty and ready

---

## The rules that matter

**The gates decide when a page is done, not you.** `check_page_rhythm.py`, `check_voice.py` and `check_page_done.py` run on every page. A failing page gets fixed and re-run, never shown with an explanation attached.

**Never invent a number.** Prices, review counts, results and search volumes come from real data or they are marked as estimates. A placeholder is honest; a fabricated figure is not.

**The city never goes in a service name.** Services are ranked by `[service] [city]` search volume because that is the only way to measure local demand, but the city is a measuring device, not part of the answer. "Emergency Plumber", never "Emergency Plumber Toronto". It is a suspension risk, not a ranking one.

**Local volumes run low and that is fine.** A local term at 50 searches a month with buying intent beats a national one at 500 without it. The floor is 50 for a root keyword and 10 for a secondary, against 100 and 50 for national terms.

**Built is not published.** Pages get publish dates, not an immediate deploy. `/publish` ships what is ready.

---
> Source: [youtube-jono/local-seo-agent](https://github.com/youtube-jono/local-seo-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
