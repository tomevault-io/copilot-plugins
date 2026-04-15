## git-presented

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Git Presented is a Jigsaw-based static site generator that transforms a Git repository into a presentation website. It reads Git history (commits, branches) and renders commits tagged with `#presentation` as navigable slides.

## Build Commands

```bash
# Install dependencies
composer install

# Build for development (outputs to build_local/)
php jigsaw build

# Build for production (outputs to build_production/)
php jigsaw build production

# Start development server on localhost:8000
php jigsaw serve

# Regenerate autoload after adding PHP classes
composer dump-autoload
```

## Architecture

### Core Components

The project has two main layers:

1. **Git Domain Layer** (`app/Git/`)
   - `Model/` - Value objects: `Commit`, `Branch`, `Presentation`, `Step`, `SubSlide`, `Author`, `FileChange`
   - `Parser/` - Git CLI wrappers: `GitCommandRunner`, `CommitParser`, `BranchParser`, `PresentationBuilder`
   - `Repository.php` - Facade that ties everything together
   - `Provider/GitDataProvider.php` - Exposes repository data to Jigsaw

2. **Jigsaw Layer** (`source/`, `config.php`, `helpers.php`, `bootstrap.php`)
   - `config.php` - Main configuration, builds collections from Git data
   - `helpers.php` - Helper functions for Git configuration (loaded by config.php)
   - `bootstrap.php` - Jigsaw event handlers
   - `source/_layouts/` - Blade layout templates for each page type
   - `source/_partials/` - Reusable Blade components

### Data Flow

```
Git Repository
    ↓
GitCommandRunner (executes `git` CLI commands with proper escaping)
    ↓
Parsers (CommitParser, BranchParser)
    ↓
PresentationBuilder (finds commits with #presentation tag, groups into presentations)
    ↓
Repository (facade exposing presentations, branches)
    ↓
config.php (builds Jigsaw collections with IDs only - objects looked up in templates)
    ↓
Blade templates (look up full objects from global collections)
    ↓
Static site output
```

### Key Concepts

- **Presentation**: A branch containing commits with `#presentation` tag
- **Step**: A single commit in a presentation (becomes a slide)
- **SubSlide**: A section within a step (created from `## Heading` in commit body)
- **Code Snippet**: Referenced code displayed alongside content (`[file.php:10-20]`)

### Collections

Jigsaw collections are built in `config.php`. Collections store only primitive data (IDs, titles) - the actual objects are looked up from global `$page->presentations`, `$page->branches` in templates:

- `presentations` → `/presentations/{id}` (presentation overview)
- `presentationSlides` → `/presentations/{id}/{step}` (slide view)
- `branches` → `/branches/{slug}` (branch detail)

## Configuration

Environment variables in `.env` control behavior:
- `GIT_REPO_PATH` - Target repository path (defaults to project directory)
- `GIT_EXCLUDE_PATTERNS` - Comma-separated glob patterns for files to hide from diffs
- `DEFAULT_THEME` - Color theme (light, dark, presentation, schiphol, etc.)
- `DEFAULT_MODE` - Display mode (browser or beamer)

## Working with the Code

### Adding a New Git Model

1. Create class in `app/Git/Model/`
2. Add parsing logic in appropriate parser (`app/Git/Parser/`)
3. Expose via `Repository.php`
4. Add to collection in `config.php` if needed (store ID only, not the object)
5. Create layout in `source/_layouts/` that looks up the object
6. Run `composer dump-autoload`

### Modifying Templates

- Layouts are in `source/_layouts/`
- Partials are in `source/_partials/`
- Collection pages receive IDs via `$page->presentationId`, `$page->stepId`, etc.
- Look up actual objects from global collections: `$page->presentations`, `$page->branches`
- For expensive data (diffs, file trees), load on-demand via `$page->gitRepo`

### Git Command Execution

All Git commands go through `GitCommandRunner`. Format strings with special characters (like `<`, `>`, `%`) must be properly escaped:

```php
$runner = new GitCommandRunner('/path/to/repo');
// Use escapeshellarg for format strings with special chars
$format = escapeshellarg('--format=%H %an <%ae>');
$output = $runner->runRaw("log {$format} -10");
```

### Important Implementation Notes

1. **Collection items must be primitive data** - Jigsaw serializes collection items as YAML, which doesn't handle PHP objects or large strings well. Store IDs and look up objects in templates.

2. **Large data should be loaded on-demand** - Diffs and file trees are loaded in templates via `$page->gitRepo->getDiff()` rather than stored in collection items.

3. **Shell escaping is critical** - Git format strings with `<>` or `()` will break if not escaped with `escapeshellarg()`.

## File Structure

```
├── app/Git/              # PHP domain layer
│   ├── Model/            # Value objects
│   ├── Parser/           # Git CLI wrappers
│   ├── Provider/         # Jigsaw data provider
│   └── Repository.php    # Main facade
├── source/               # Blade templates
│   ├── _layouts/         # Page layouts (presentation, branch)
│   ├── _partials/        # Reusable components
│   └── index.blade.php   # Home page
├── config.php            # Jigsaw configuration + collection building
├── helpers.php           # Git config helper functions
├── bootstrap.php         # Jigsaw event handlers
├── .env                  # Environment configuration (gitignored)
└── .env.example          # Example configuration
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/remyhonig)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/remyhonig)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
