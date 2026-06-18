## ai-news-scrapper

> >


# AI Intelligence Skill

## When to use this skill

Use when the user wants to:
- Get the latest AI/ML news and announcements
- Browse trending open-source AI projects from GitHub
- Read AI discussion highlights from Reddit, X (Twitter), HackerNews
- See what AI KOLs (Key Opinion Leaders) are saying
- Get a comprehensive daily AI briefing

## Workflow

### Default: Fetch latest AI briefing

Run the fetcher script from the skill directory:

```bash
uv run scripts/pulse_fetcher.py
```

If `uv` is not available, install dependencies manually and use:
```bash
pip install requests pyyaml
python scripts/pulse_fetcher.py
```

### Filtered fetch

Map the user's intent to CLI flags:

| User says | Flags |
|-----------|-------|
| "Only Reddit discussions" | `--include-categories discussion --include-sources reddit` |
| "Open source, no Github" | `--include-categories opensource --exclude-sources github` |
| "50 items, past 48 hours" | `--limit 50 --hours 48` |
| "Show full summaries with links" | `--show-long-summary true --show-link true` |
| "AI news in Chinese" | `--include-categories news --language zh` |

### Available options

Run `uv run scripts/pulse_fetcher.py --help` for the full list. Key flags:

- `--include-categories "cat1,cat2"` — Only fetch these categories: `news`, `opensource`, `discussion`, `kol`
- `--exclude-categories "cat1,cat2"` — Skip these categories
- `--include-sources "src1,src2"` — Only from these sources
- `--exclude-sources "src1,src2"` — Skip these sources
- `--hours INT` — Time window in hours (default: 24)
- `--limit INT` — Max items to request. The server clamps to the tier cap (guest=3 / free=6 / member=100); anything larger is silently truncated.
- `--language en|zh` — Output language (default: auto-detect from system locale)
- `--show-short-summary true|false` — Show short summary (default: true)
- `--show-long-summary true|false` — Show long summary (default: false)
- `--show-link true|false` — Show article URLs (default: false)
- `--output-file FILE` — Export full JSON to file (default: pulse_output.json). Pass empty string to disable.

## Output format

Each item is printed as:
```
1. [NEWS] Title of the article (from TechCrunch)
   Short Summary: One-line summary text
----------------------------------------
```

When `--output-file` is used, the full JSON response is saved to disk. See [references/API.md](references/API.md) for the response schema.

## Gotchas

- **HTTP 429** means the user hit an IP rate limit (anti-spam interval or daily cap). Tell them to wait a few minutes and try again.
- **Tier caps on result count.** The response includes a `tier` field and items are capped per request based on it:
  - `guest` (no / invalid `api_key`): max **3** items
  - `free` (logged in, non-member): max **6** items
  - `member`: max **100** items
  If the user asks for more items than the tier cap, the server silently clamps to the cap. To raise the cap, tell the user to register at [InBrief.info](https://inbrief.info), open **Settings → PulseAI Agent Skill**, copy the API Key, and paste it into the `api_key` field of `config.yaml` (see [assets/config.default.yaml](assets/config.default.yaml)).
- Use `--output-file` when links are included — terminal line-wrapping can corrupt long URLs.
- If `config.yaml` exists in the skill root, it overrides default settings. See [assets/config.default.yaml](assets/config.default.yaml) for the template.
- The `--include-*` flags take precedence over `--exclude-*` flags for the same dimension.

## AI Product Engineer course (12-week track)

When the user asks for news related to their study platform course, PR-Critic, Document Copilot, or weekly learning topics:

1. Fetch from InBrief (see workflow above), always with `--output-file pulse_output.json` and `--show-link true`.
2. Filter for course relevance:
   ```bash
   python scripts/course_brief.py --phase auto --week N
   ```
   Use explicit `--week` when the user names a week; `--phase 1` or `--phase 2` when they name a phase.
3. Read keyword map and CLI examples in [references/course-ai-product-engineer.md](references/course-ai-product-engineer.md).
4. Present results in Ukrainian unless the user asks for English. Tag each item with Phase/Week relevance.
5. If tier cap returns too few items (guest=3), suggest registering at [InBrief.info](https://inbrief.info) and adding `api_key` to `config.yaml`.

---
> Source: [sanchahous/ai-news-scrapper](https://github.com/sanchahous/ai-news-scrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
