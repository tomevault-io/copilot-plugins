## ytfeed

> Use when the user wants to collect, review, summarize, or parse videos from their YouTube homepage recommendations, including video links, channel details, metadata, and durations as TOON or JSON.


# YTFeed

Collects videos from a YouTube homepage and returns an LLM-friendly feed.

## Workflow

1. Use the default TOON export unless the user asks for a specific format.
2. Use JSON when the result needs to be parsed, filtered, transformed, or reused
   by another tool.
3. Pass a limit when the user asks for a specific number of videos.
4. Refresh the saved YouTube session only when authentication is needed or the
   feed appears empty because the session has expired.

## Commands

| Command                                      | Description                                             |
| -------------------------------------------- | ------------------------------------------------------- |
| `ytfeed login`                               | Open YouTube in a visible browser and save the session. |
| `ytfeed scrape --limit 10 --format <format>` | Export YouTube homepage videos.                         |

Supported export formats: `json`, `toon`. Default: `toon`.

## Output

Exports include:

- `scrapedAt`
- `limit`
- `count`
- `videos`

Each video may include `title`, `url`, `videoId`, `channel`, `channelUrl`,
`duration`, `views`, `published`, and `metadata`.

## Gotchas

- The default output is TOON with a limit of `20`.
- `ytfeed login` is interactive; use it only when a session refresh is needed.
- Some fields may be `null` or omitted when YouTube does not expose them in the
  homepage card.

---
> Source: [riccardopll/ytfeed](https://github.com/riccardopll/ytfeed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
