## obsidianxranklin-jl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Run tests:**
```bash
julia --project=. -e 'using Pkg; Pkg.test()'
# or from Julia REPL:
# ] test
```

**Run a single test file:**
```bash
julia --project=. test/runtests.jl
```

**Activate the environment (REPL):**
```julia
using Pkg; Pkg.activate(".")
```

**Install dependencies:**
```bash
julia --project=. -e 'using Pkg; Pkg.instantiate()'
```

## Architecture

ObsidianXranklin.jl bridges [Obsidian](https://obsidian.md) note vaults with [Xranklin](https://xranklin.franklinjl.org/) static site generators. The main entry point is `sync_vault()` in `vault.jl`.

### Pipeline (`sync_vault`)
1. **Discover** notes via `discover_notes()` — walks vault, parses YAML frontmatter, filters by `publish: true` or folder rules
2. **Index** notes via `build_note_index()` — maps note slugs/names → URLs for link resolution
3. **Transform** each note:
   - `transform_callouts()` — `> [!TYPE] Title` → HTML `<div class="callout ...">` blocks
   - `convert_frontmatter()` — YAML `---` → Xranklin TOML `+++` (filters Obsidian-internal keys)
   - `transform_wikilinks()` — `[[wiki-links]]` → markdown links, collects graph edges
   - `process_base_embeds()` — `![[file.base]]` → inline HTML tables
4. **Write** output to `<site>/<output_dir>/<slug>/index.md`
5. **Optionally copy** `index_note` slug's output to `<output_dir>/index.md` (section homepage)
6. **Copy** media assets to `_assets/vault/`
7. **Generate** `_assets/graph_data.json` for D3.js force-directed graph

### Key Modules
| File | Purpose |
|------|---------|
| `src/types.jl` | `NoteInfo` struct (src_path, slug, title, tags, frontmatter) |
| `src/vault.jl` | Main orchestration: `sync_vault`, `discover_notes`, `should_publish`, `build_note_index` |
| `src/frontmatter.jl` | YAML↔TOML conversion; handles dates, booleans, arrays |
| `src/wikilinks.jl` | Resolves `[[note]]`, `[[note#section]]`, `[[note|alias]]`, `![[image]]` |
| `src/callouts.jl` | Converts Obsidian callout blocks to HTML divs |
| `src/bases.jl` | Renders Obsidian Bases `.base` files as HTML tables |
| `src/hfuns.jl` | Xranklin template function: `{{obsidian_graph}}` |
| `assets/graph-view.js` | D3.js v7 interactive knowledge graph visualization |

### Publish Rules (in priority order)
1. `publish: false` in frontmatter → never publish
2. `publish: true` in frontmatter → always publish
3. Note path matches any prefix in `publish_folders` → publish
4. Default → do not publish

### Xranklin Integration
Add to the site's `utils.jl`:
```julia
using ObsidianXranklin
# hfun_obsidian_graph is now available as {{obsidian_graph}} in templates
```
Call `sync_vault()` from `utils.jl` or a site build script. The `index_note` kwarg sets which note slug is copied to `<output_dir>/index.md` (the section root URL); it should match `obsidian_home` in the site's `config.jl`.

### Frontmatter Handling
Obsidian YAML keys filtered out (not written to TOML): `cssclasses`, `aliases`, `position`, `file`, `publish`. Dates are parsed and emitted as `Date(year, month, day)` with a `using Dates` import prepended.

### Wikilink Resolution
Unresolved `[[links]]` become `[text](#wikilink-missing)` and emit a `@warn`. Image embeds `![[photo.png]]` → `/assets/vault/photo.png` (spaces replaced with `-`). Note embeds `![[other-note]]` become blockquote-style inline links.

### Code Style
- Guard clauses use short-circuit `&&` rather than `if/end`: `isnothing(x) && return default`
- Type-based branching uses multiple dispatch (separate method signatures), not `isa` chains inside a single function body. Use abstract supertypes in signatures (`Integer`, not `Int`; `Union{Bool, Integer, AbstractFloat}` for numerics)
- Triple-quoted heredocs are fine for multi-line string construction; Julia strips the leading newline and common indentation automatically
- Use `ispath(dst)` (not `isfile(dst)`) when guarding before `cp`. `isfile` returns `false` for broken symlinks (e.g. Zotero-managed files that exist locally but not in CI), so the guard silently fails and `cp` crashes with `EEXIST` when trying to recreate the symlink at `dst`.

### Test Fixtures
`test/fixtures/vault/` contains representative notes: `published-note.md` (publish: true, has links/callouts), `private-note.md` (publish: false), `folder-published.md` (no frontmatter, covered by folder rule), `another-note.md` (link target), `sample.base` (Bases view).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BonhamLab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
