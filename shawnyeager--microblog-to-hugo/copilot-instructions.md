## microblog-to-hugo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Microblog-to-Hugo is a collection of Python 3 scripts for migrating and refactoring content from Micro.blog exports to Hugo static sites. The scripts handle frontmatter standardization, filename cleanup, slug generation, and image migration.

## Architecture

### Core Migration Pipeline

The repository implements a **pipeline architecture** where each script performs one specific transformation:

1. **Frontmatter Standardization** (`standardize_frontmatter.py`) - Cleans TOML frontmatter
2. **Filename Simplification** (`simplify_filenames.py`) - Removes date prefixes from filenames
3. **Slug Generation** (`add_slugs.py` or `fix_slugs.py`) - Generates URL-friendly slugs

The `migrate.py` wrapper orchestrates these steps with safety checks and error handling.

### Script Relationships

```
migrate.py (wrapper)
    ├── standardize_frontmatter.py (always runs)
    ├── simplify_filenames.py (optional, --skip-filenames)
    └── add_slugs.py (optional, --skip-slugs)

fix_slugs.py (standalone - regenerates all slugs)
migrate_images.py (standalone - moves images to assets/)
```

### Key Design Patterns

**File Processing Pattern**: All scripts follow this structure:
- Read file with UTF-8 encoding
- Split content on `+++` TOML delimiters (frontmatter vs body)
- Process frontmatter with regex transformations
- Write modified content back to file

**Slug Generation Logic**:
- If post has a title: `slugify(title)`
- If titleless: `slugify(first_6_words_of_content)`
- Collision handling: append `-2`, `-3`, etc.

**Collision Detection**: Scripts track used slugs/filenames in sets/dicts to prevent overwrites.

## Content Expectations

- **Directory structure**: Scripts expect `content/post/` for posts (configurable via `post_dir` variable)
- **Frontmatter format**: TOML with `+++` delimiters
- **Encoding**: UTF-8
- **File type**: Markdown (`.md`)

Note: `simplify_filenames.py` has a discrepancy - it looks for `content/posts/` and `content/notes/` instead of `content/post/`. This may need adjustment based on actual site structure.

## Common Development Commands

### Running Scripts

```bash
# One-command migration (recommended)
python3 migrate.py                  # Full migration
python3 migrate.py --dry-run        # Preview only
python3 migrate.py --skip-filenames # Skip filename cleanup
python3 migrate.py --skip-slugs     # Skip slug generation

# Individual scripts (for granular control)
python3 standardize_frontmatter.py
python3 simplify_filenames.py
python3 add_slugs.py
python3 fix_slugs.py

# Image migration
python3 migrate_images.py              # Migrate all images
python3 migrate_images.py --dry-run    # Preview changes
python3 migrate_images.py --year 2024  # Migrate specific year
```

### Making Scripts Executable

All scripts have shebang (`#!/usr/bin/env python3`) and can be run directly:

```bash
chmod +x *.py
./migrate.py --dry-run
```

## Important Implementation Notes

### Path Configuration

To use with different content directories, update the `post_dir` variable in each script:

```python
# Example from standardize_frontmatter.py
post_dir = Path('content/post')  # Change this path as needed
```

### Regex Patterns

Key regex patterns used across scripts:

- **Date extraction**: `r'^\d{4}-\d{2}-\d{2}-(.+)$'` (YYYY-MM-DD prefix)
- **Frontmatter fields**: `r'^field_name\s*=\s*"([^"]+)"'` (MULTILINE)
- **Empty fields**: `r'^title\s*=\s*""?\s*$'`
- **Slugification**: Removes non-word chars, converts spaces to hyphens

### Frontmatter Transformations

`standardize_frontmatter.py` performs these operations:
- Removes empty `title`, `slug`, `summary` fields
- Quotes unquoted dates: `date = 2024-01-15...` → `date = "2024-01-15..."`
- Standardizes category arrays: `[ "foo" ]` → `["foo"]`
- Strips trailing whitespace

### Slug Generation Differences

**`add_slugs.py`** (incremental):
- Only adds slugs where missing
- Uses 4 words for titleless posts
- Preserves existing slugs

**`fix_slugs.py`** (complete regeneration):
- Regenerates ALL slugs (replaces existing)
- Uses 6 words for titleless posts
- Use when improving previously generated slugs

## Testing After Changes

After running migration scripts:

```bash
# Review changes
git diff

# Test Hugo site builds successfully
cd /path/to/hugo/site
hugo server

# Verify permalink structure works
# Ensure hugo.toml has: [permalinks] post = "/:slug/"
```

## Image Migration Architecture

`migrate_images.py` moves images from `static/images/` to `assets/images/` to enable Hugo's image processing (WebP conversion, responsive srcsets, lazy loading).

**Important**:
- Hugo automatically serves images from `assets/images/` at `/images/` URL path
- Frontmatter should use absolute paths: `/images/2024/photo.jpg`
- Requires Hugo Extended for image processing
- Videos should stay in `static/` (not processed)

## Safety Considerations

- **Always commit to git before running scripts** - All modify files in place
- **Use --dry-run first** - Preview changes with `migrate.py --dry-run`
- **Check for collisions** - Scripts report filename/slug collisions
- **Backup recommended** - Scripts don't create automatic backups

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shawnyeager) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
