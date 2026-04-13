## code-analyzer

> Use the MCP tools in this order for repository questions:

# Rocket.Chat Code Analyzer Extension Instructions

Use the MCP tools in this order for repository questions:

1. Call `repo_index` first with `targetDir`.
2. Use `read_file` only for files required to answer the question.
3. Prefer `symbolsOnly=true` for structural questions.
4. Use `lineRange` and `maxLines` for large files to keep context bounded.
5. Use `index_cache_stats` to inspect cache state and `index_cache_invalidate` after bulk file changes.

## Expected Cache Behavior

The `repo_index` tool uses persistent index caching.

- In-process memory cache is checked first for low-latency repeated calls.
- Disk cache is used when memory cache is cold but fingerprints still match.

- First call for a target directory: cache miss, index is built.
- Repeated call with no relevant file changes: cache hit, cached index is returned.
- After changing indexable `.ts` or `.tsx` files: cache miss, index is rebuilt and cache is refreshed.
- Set `forceRefresh=true` on `repo_index` to bypass cache intentionally.

Cache metadata is returned in `repo_index` responses under `cache`:

- `enabled`: whether caching is enabled
- `hit`: whether the current call used cache
- `layer`: `memory`, `disk`, or `rebuild`
- `cacheFile`: cache file location
- `fingerprint`: file-state fingerprint used for invalidation

`index_cache_stats` returns both index cache stats and `read_file` cache stats.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PrajanManojKumarRekha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PrajanManojKumarRekha)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
