## zmediumtomarkdown

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Ruby gem + CLI that downloads Medium posts via Medium's internal GraphQL API and renders them to Markdown. Ships as the `ZMediumToMarkdown` gem and a GitHub Action (`action.yml`).

## Commands

```bash
bundle install                       # install deps
bundle exec rake test                # run the full minitest suite
bundle exec ruby -Ilib -Itest test/integration_test.rb   # single file
bundle exec ruby -Ilib -Itest test/integration_test.rb -n test_converts_real_post_into_expected_markdown   # single test

UPDATE_FIXTURES=1 bundle exec rake test    # regenerate the golden markdown after intentional output changes

bundle exec ruby bin/ZMediumToMarkdown -p "https://medium.com/<USER>/<POST>"   # run from source
```

CI (`.github/workflows/test.yml`) runs `bundle exec rake test` against Ruby 3.2 / 3.3 / 3.4. There is no separate lint step.

### Cookies / Cloudflare

Hitting Medium's GraphQL endpoint reliably requires a logged-in cookie pair (`sid`, `uid`). Pass via `-s` / `-d` or `MEDIUM_COOKIE_SID` / `MEDIUM_COOKIE_UID`. CLI flags take precedence over env vars (see `CLI.loadCookiesFromEnv!` — env vars only fill in missing cookies). Without cookies, the CLI prints `COOKIE_WARNING_BANNER` from `lib/CLI.rb` and may surface a `Request::CloudflareBlockedError` at runtime (HTTP 403/503 + cf-mitigated header or "Just a moment..." body). `bin/ZMediumToMarkdown` rescues that error specifically with a clean exit 1.

Endpoint hosts are env-overridable for proxy/Worker setups: `MEDIUM_HOST`, `MIRO_MEDIUM_HOST`, `TWITTER_SYNDICATION_HOST`.

## Architecture

Entry point flow: `bin/ZMediumToMarkdown` → `CLI.main` (option parsing, cookie loading, cookie warning) → `ZMediumFetcher#downloadPost` or `#downloadPostsByUsername`.

### Conversion pipeline (the core)

`ZMediumFetcher#downloadPost` is the spine:

1. `Post.parsePostInfo(postID, pathPolicy)` POSTs the `PostPageQuery` GraphQL operation to `MEDIUM_HOST` to get post-level metadata (title, tags, creator, dates, collection, preview image). The GraphQL document lives at `lib/Queries/PostPageQuery.graphql` and is read at runtime. (This replaced an earlier HTML/`__APOLLO_STATE__` scraping path which Medium has been progressively dismantling.)
2. `Post.fetchPostParagraphs(postID)` POSTs the `PostViewerEdgeContentQuery` GraphQL operation to get the actual paragraph body. Lives at `lib/Queries/PostViewerEdgeContentQuery.graphql`. **Edit the `.graphql` files, not Ruby strings** — both queries are loaded via `Post.postPageQueryString` / `Post.postViewerEdgeContentQueryString` (file-read + memoized).
3. `preprocessParagraphs` walks raw paragraph dicts and produces `Paragraph` objects: numbers `OLI` items, inserts blank `P` separators between list/quote runs, and **merges consecutive `PRE` paragraphs into one `CODE_BLOCK`** via `flushPreParagraphsInto`. The trailing-PRE flush at the end of the loop is required for posts that end in code (don't remove it).
4. `buildParser` wires a Chain-of-Responsibility: each `Parser` subclass has `parse(paragraph)` and `setNext`. Order in `lib/ZMediumFetcher.rb` matters — earlier parsers claim the paragraph first; `FallbackParser` is always last.
5. For non-code paragraphs, `MarkupParser` → `MarkupStyleRender` applies inline markups (bold, italic, links, code) to `paragraph.text` *before* the paragraph parser runs. `Paragraph#initialize` also injects synthetic `ESCAPE` markups for every char matching `Helper::MARKDOWN_ESCAPE_REGEX`, so escaping is markup-driven, not a post-process step. The 4-byte-char `index += 1` bump in `Paragraph#initialize` is intentional — Medium counts certain emoji as two positions.
6. Skip-already-downloaded: `readExistingFrontMatter` parses the first 15 lines of the existing `.md` for `last_modified_at` / `pin` / `lockedPreviewOnly` and re-renders only if any changed.

### Iframe / embed dispatch

`lib/Parsers/IframeParser.rb` is the most-touched file when handling new embed providers. The dispatch order is: YouTube → unwrap embedly → Twitter/X → Widgetic (skip) → Vimeo → SoundCloud → Spotify → fetch iframe HTML → Gist (rewrite to fenced code) → generic OG-image card. Twitter rendering is delegated to `Parsers/TwitterEmbed.rb`, which calls Twitter's public syndication endpoint. Each provider has a Jekyll branch (real `<iframe>` markup) and a plain branch (downloaded thumbnail card via `renderThumbnailLink`, or OG image, or plain link).

### Output modes

`isForJekyll` flag (set in `CLI.run`) changes path policy and per-parser rendering:

- Plain: `./Output/zmediumtomarkdown/<date>-<slug>.md`, assets under `Output/zmediumtomarkdown/assets/<post_id>/`. Username runs nest under `Output/users/<username>/`.
- Jekyll: `./_posts/zmediumtomarkdown/<date>-<post_id>.md`, assets at `./assets/<post_id>/`, plus `{:target="_blank"}` link markers and `render_with_liquid: false` in front matter.

`PathPolicy` is the abstraction — it's just `(rootPath, relativePath)` and is threaded through parsers that emit asset references.

### Encoding

`Request.readBodyAsUTF8` force-encodes every HTTP body to UTF-8 before Nokogiri sees it. Without this, Nokogiri's encoding sniffer falls back to ISO-8859-1 for inline `<script>` content and mojibakes the embedded JSON (CJK / emoji break). Don't bypass it.

### TLS

`Request.URL` builds a fresh `OpenSSL::X509::Store` with `flags = 0` to avoid CRL-check failures on systems where OpenSSL has CRL checking enabled by default. `SSL_CERT_FILE` / `SSL_CERT_DIR` env vars override the CA bundle.

## Testing notes

- `test/test_helper.rb` defines `$cookies ||= {}` before requiring `Request` — the global is required to load. Always `require_relative 'test_helper'` from new tests.
- `test/integration_test.rb` is the end-to-end golden-file test: it runs the fixture at `test/fixtures/post_7c0974856393.json` through `preprocessParagraphs` + the parser chain and diffs against `post_7c0974856393.expected.md`. All network/filesystem boundaries (`Request.URL`, `Request.html`, `Request.body`, `Helper.fetchOGImage`, `ImageDownloader.download`) are stubbed via `Minitest::Mock`-style `.stub` so the test is hermetic. When making intentional output changes, run `UPDATE_FIXTURES=1 bundle exec rake test` to rewrite the golden, then re-run without it to assert.
- `TestSupport.paragraph(overrides)` in `test_helper.rb` is the canonical way to build `Paragraph` objects in unit tests.

## Conventions

- The codebase predates modern Ruby style — methods are `camelCase`, files are `PascalCase.rb`. Match what's there; don't snake-case-refactor.
- Parsers are stateless except for `nextParser` and a couple of injected configs (`pathPolicy`, `isForJekyll`); avoid adding instance state that survives across paragraphs.
- Don't introduce a global mutable cookie store beyond `$cookies` — it's load-bearing for `Request`, but expanding it makes testing harder.

---
> Source: [ZhgChgLi/ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
