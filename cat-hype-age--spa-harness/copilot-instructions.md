## spa-harness

> Extract routes, human-readable copy, and metadata from JS-rendered SPA sites (Vite/React/Lovable) without a browser. Use for content audits, migration dumps, and claim extraction on sites the user owns or is authorized to audit.


# spa-harness (agent interface)

A zero-dependency CLI that mines a SPA's compiled JS bundle for content. No browser, no rendering — string extraction from the same files a visitor's browser downloads.

## When to use

- The user asks to audit, extract, inventory, or migrate the content of a site that returns a near-empty HTML shell (JS-rendered SPA)
- You need a site's route map, copy strings, or meta tags as structured data
- `web_fetch`-style tools return only `<div id="root"></div>` and meta tags

## When NOT to use

- The site requires JavaScript execution, auth, or interaction → use a browser tool
- Server-rendered sites → plain fetch already works
- Sites the user does not own / is not authorized to audit → decline; this tool is for authorized audits (it enforces a robots.txt courtesy check)

## Workflow (always in this order)

```bash
spa-harness probe <url>            # 1. discover assets + meta tags
spa-harness pull <url>             # 2. cache bundles (per-domain, ~/.spa_harness)
spa-harness routes <url> --json    # 3. route map
spa-harness strings <url> --json   # 4. human-readable copy (--min N to tune)
spa-harness grep <url> "<pattern>" --fixed --context 150 --json   # targeted search
spa-harness dump <url> > out.json  # everything, structured
```

All commands support `--json`. `grep` supports regex by default, `--fixed` for literal.

## Output contract

`dump` returns: `{base_url, title, meta_tags{}, routes[], strings[], string_count}`.
`strings` filters for human-readable copy (≥75% letters, contains spaces, no code smells) — tune `--min` (default 25 chars) if copy is missing or noisy.

## Failure modes

- "No cache" → run `pull` first
- robots.txt refusal → confirm with the user that they own/are authorized for the site before using `--ignore-robots`
- Empty `strings` on a site that clearly has copy → bundle may use template literals or chunked lazy routes; try `grep` for known phrases, or lower `--min`

---
> Source: [cat-hype-age/spa-harness](https://github.com/cat-hype-age/spa-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
