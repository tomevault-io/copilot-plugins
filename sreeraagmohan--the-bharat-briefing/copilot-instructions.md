## the-bharat-briefing

> Nightly pipeline that scrapes publications, ranks stories, and emails a briefing

# darpan-newsletter

Nightly pipeline that scrapes publications, ranks stories, and emails a briefing
that feeds The Bharat Briefing (TBB), a daily newsletter on Indian government
policy, business, and geopolitics for foreign executives, investors, and diplomats.

## Reference docs

Read these before working on selection, ranking, or output formatting.

- `docs/placement-rules.md` governs **selection**. It is the scoring rubric for
  lead, AITN, fold, and skip. This is the file the ranker uses.
- `docs/sources.md` covers **source integration**. Fetch constraints, retry
  behaviour, deduplication, and failure handling for each publication.
- `docs/editorial-prompt.md` governs **writing**, and only applies once a story has
  been selected. Do not use it for ranking. Scoring stories against a style guide
  ranks them by how enjoyable they are to write about rather than by how much they
  matter.

## Pipeline map

`app.py` orchestrates: fetch, scope filter, deduplicate, select, rank, draft, email.

| File | Role |
|---|---|
| `sources.py` | Source registry. URL, kind, weight, and which fetcher handles it. |
| `fetchers.py` | Fetch layer. Bespoke fetchers for PIB, MEA, and Reuters; generic RSS for the rest. Returns candidates plus a per-source report. |
| `dedupe.py` | Scope filter and merging. Turns candidates into one Story per event. |
| `rubric.py` | Every weight, threshold, and gate. Tune the briefing here. |
| `ranker.py` | Scores against `docs/placement-rules.md`. Applies gates and overrides. |
| `archive.py` | Prior TBB coverage from the beehiiv archive, cached, for novelty. |
| `drafting.py` | Writes selected stories against `docs/editorial-prompt.md`. |
| `test_pipeline.py` | Smoke tests for the pure logic. No network, no model calls. |

```bash
python app.py --dry-run      # every source, counts and samples, sends nothing
python app.py --preview      # full run, prints the email instead of sending
python app.py --backfill 2026-07-28
python test_pipeline.py
```

## Source facts, verified live on July 30, 2026

Do not re-derive these from memory. They cost a morning to establish.

- **PIB RSS carries no pubDate on any item.** Every PIB release was being
  dropped by the lookback filter before this was found. The dated listing at
  `allRel.aspx?reg=3&lang=1` plus the individual release page supply the
  timestamp. Release times are IST.
- **PIB's dated listing is an ASP.NET postback.** Date parameters passed in the
  query string are ignored. The post needs every hidden input from the page,
  not just `__VIEWSTATE` and `__EVENTVALIDATION`, or it silently returns today.
- **mea.gov.in blocks datacentre traffic.** Every path returns HTTP 403 from a
  cloud runner or a VPN, and loads normally from an ordinary connection. The
  MEA fetcher tries direct first and falls back to a Google News relay, so it
  works locally and degrades in CI rather than failing.
- **indianexpress.com and thehindu.com block by network too, in opposite
  directions.** Verified August 5, 2026 after Indian Express broke the
  August 4 issue. On that CI run Indian Express returned zero and The Hindu
  read directly at 60 items; from a development machine the next day the two
  swap, with Indian Express at 200 items and The Hindu returning Cloudflare's
  "Just a moment..." as HTTP 403 on every attempt. Identical headers both
  sides, so it is the network and not the request, and there is no single
  network on which every source works.

  Any RSS source can declare a `relay_url` in the registry to get direct-first,
  relay-second behaviour. Do not skip the direct route because the relay is
  easier: a relay halves the item count, lags a day or more, carries no body
  text, and yields Google News URLs that cannot be resolved server-side. The
  fallback fires only when direct is unreadable, so in CI it costs nothing.

  Do not test a block by header tinkering alone; both of these ignore Accept,
  User-Agent, and a full navigation header set. Change the network instead.
  The Indian Express feed carries no body text on either route; its `summary`
  and `content` elements are present and empty on every item.
- **PIB no longer carries MEA releases.** Ministry filter id 4 returns zero
  while Defence returns dozens. There is no primary route that works everywhere.
- **Google News relay URLs cannot be resolved server-side.** The base64 payload
  no longer carries the target URL and the redirect happens in the browser.
  They are usable citations for a human, poor ones for a published issue, so a
  direct publisher URL always wins the citation slot and a relay-only story is
  flagged.
- **Backfill only genuinely replays PIB.** RSS feeds serve their current window
  only, so a backfill of an older day is a test of the ranker rather than a
  reconstruction of that day.

## Working rules for this repo

- Verify every feed URL live before building on it. Do not assume a URL from
  memory. Print sample items as proof that a source resolves.
- Never let a source fail silently. Zero items from a source is a loud failure.
- Preserve the existing nightly email format when adding fields. Additions go
  alongside what is there, not in place of it.
- Ask before adding a paid dependency or an API that requires a key.

## House style for any generated copy

Applies to anything the pipeline writes that a human might publish.

- UK spelling. Oxford comma.
- No em dashes.
- Never start a sentence with "And."
- No "it's not X, it's Y" construction in any word order.
- Expand acronyms on first use.
- Dates month-first: June 18, 2023.
- "West Asia," never "Middle East." "Centre" capitalised for India's central
  government.

---
> Source: [sreeraagmohan/the-bharat-briefing](https://github.com/sreeraagmohan/the-bharat-briefing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
