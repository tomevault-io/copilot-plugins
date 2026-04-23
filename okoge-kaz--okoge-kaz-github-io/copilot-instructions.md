## okoge-kaz-github-io

> - Build: `hugo` (Homebrew)

# Project Instructions for Claude Code

## Hugo Site
- Build: `hugo` (Homebrew)
- Dev server: `hugo server`

## Citation Count Auto-Update

When asked to update citation counts (or when you notice they may be stale), follow this procedure:

1. Fetch citation data from Google Scholar:
   ```bash
   curl -s -L -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "https://scholar.google.co.jp/citations?user=jHXLs2wAAAAJ&hl=en" | grep -o 'class="gsc_a_at"[^>]*>[^<]*\|class="gsc_a_ac[^"]*"[^>]*>[^<]*'
   ```

2. Update `content/publications/index.md`:
   - Each paper has a `Cited_by-N` badge in its `pub-badges` div
   - Update the number in both the URL and alt text: `Cited_by-{N}-4285F4?style=flat-square&logo=google-scholar`
   - Update the `Total_Citations` badge at the top of the file (sum of all individual counts)
   - Update the `<!-- citation-update: YYYY-MM-DD -->` comment with today's date

3. Papers without citations (0) should not have a citation badge.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okoge-kaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
