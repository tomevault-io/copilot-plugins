## dynasty9

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Drupal 11 site focused on the New England Patriots Dynasty, featuring content about games, players, podcast episodes, highlights, and timeline events. The site uses a custom Tailwind CSS + DaisyUI theme and includes custom modules for Dynasty-specific functionality.

## Development Environment

### DDEV Setup
The project uses DDEV for local development:

```bash
# Start the environment
ddev start

# Stop the environment
ddev stop

# SSH into the web container
ddev ssh

# Access database
ddev mysql
```

Project URL: https://dynasty9.ddev.site (or http://dynasty9.ddev.site)

### Environment Details
- **Drupal Version**: 11.x
- **PHP Version**: 8.3
- **Database**: MySQL 8.0
- **Web Server**: nginx-fpm
- **Document Root**: `web/`

## Common Development Commands

### Drush Commands
```bash
# Clear cache
ddev drush cr

# Import configuration
ddev drush cim -y

# Export configuration
ddev drush cex -y

# Run database updates
ddev drush updb -y

# Check migration status
ddev drush ms

# Run a specific migration
ddev drush mim [migration_id]

# Rollback a migration
ddev drush mr [migration_id]

# Entity updates
ddev drush entup -y
```

### Composer Commands
```bash
# Install dependencies
ddev composer install

# Require a new module
ddev composer require drupal/module_name

# Update dependencies
ddev composer update
```

### Theme Development (Tailwind CSS)
The custom theme is located at `web/themes/custom/dynasty_tw/`:

```bash
# Navigate to theme directory
cd web/themes/custom/dynasty_tw

# Install npm dependencies (if needed)
ddev exec npm install

# Build Tailwind CSS (if you have a build process configured)
# Note: Check package.json for available scripts
```

The theme uses:
- **Tailwind CSS 3.x** for styling
- **DaisyUI 4.x** for component library
- Custom theme colors based on Patriots branding (primary: #002244, secondary: #c60c30)

## Architecture

### Custom Modules
All custom modules are in `web/modules/custom/`:

- **dynasty_module**: Core Dynasty functionality, helpers, custom forms, and shared utilities
  - Contains DynastyHelpers class with utilities like passer rating calculations
  - Multiple administrative forms for data imports
  - Custom permissions and routing

- **dynasty_podcast**: Podcast-specific functionality
  - Manages podcast episode nodes
  - Handles Pro Football Reference (PFR) link generation
  - Wikipedia link generation for teams
  - Contains player statistics data in `player_stats/` and PFR data in `pfr_data/`

- **dynasty_transcript**: Custom entity for managing podcast transcripts

- **dynasty_timeline**: Timeline-related functionality

- **dynasty_images**: Image handling and processing

- **afc_east**: AFC East division-specific content

### Custom Themes
Located in `web/themes/custom/`:

- **dynasty_tw**: Main front-end theme (Tailwind CSS + DaisyUI based)
  - Template suggestions for pages by node type
  - Custom views field templates
  - Theme preprocessing functions
  - Regions: header, primary_menu, secondary_menu, highlighted, content, sidebars, footer, etc.

- **dynasty_admin**: Custom admin theme (Gin-based)

### Content Types
The site uses these content types (defined in `config/sync/node.type.*.yml`):

- **game**: Football games with extensive stats, Brady stats, opponent info, scores
- **podcast_episode**: Podcast episodes with download stats, transcripts
- **highlight**: Game highlights and video clips
- **player**: Player information
- **team**: Team information with PFR IDs
- **article**: Standard articles
- **event**: Events
- **page**: Basic pages
- **podcast_data**: Podcast metadata
- **twitter_hidden_image**: Social media images

### Migrations
The project includes 9 migrations for importing historical data:

- `dynasty_taxonomy_import`: Import taxonomy terms
- `dynasty_player_import`: Import player data
- `dynasty_team_import`: Import team data
- `dynasty_team_paragraph_import`: Import team paragraphs
- `dynasty_game_import`: Import game data
- `dynasty_old_home_games_csv_import`: Import old home games from CSV
- `dynasty_old_away_games_csv_import`: Import old away games from CSV
- `dynasty_highlight_import`: Import highlight data

Migration configs are in `config/sync/migrate_plus.migration.*.yml`

Custom migration process plugins are in `web/modules/custom/dynasty_module/src/Plugin/migrate/process/`

### Configuration Management
Configuration is stored in `config/sync/` directory and synced via Drush commands (cim/cex).

The site has:
- 59+ Views configurations
- Faceted search using Search API + Solr
- Entity browsers for content selection
- Paragraphs with layout paragraphs for flexible content
- Gutenberg editor integration

### Key Contributed Modules
Notable modules (see `composer.json` for full list):

- **UI/Theme**: gin, gin_login, gutenberg, ui_suite_daisyui, tailwindcss
- **Content**: paragraphs, layout_paragraphs, entity_browser, inline_entity_form
- **Search**: search_api, search_api_solr, facets
- **Migration**: migrate_plus, migrate_tools, migrate_source_csv
- **SEO**: metatag, pathauto, simple_sitemap, redirect
- **Analytics**: google_tag, plausible
- **Media**: charts (with Highcharts 6.1.0 library)
- **Performance**: quicklink, cloudflare_purge
- **Admin**: admin_toolbar, field_group, field_permissions

### External Libraries
Highcharts 6.1.0 is installed via custom repository definitions in composer.json for data visualization and charts.

### Patches
The project applies patches to:
- `drupal/facets`: Select2 compatibility fix (#3446040)
- `drupal/views_ajax_history`: jQuery $.unique deprecation fix (#3499860)

## Development Workflow

### Making Code Changes
1. Custom module code goes in `web/modules/custom/[module_name]/`
2. Custom theme code goes in `web/themes/custom/dynasty_tw/`
3. Template files use `.html.twig` extension
4. Hook implementations go in `.module` or `.theme` files
5. Clear cache after code changes: `ddev drush cr`

### Configuration Changes
1. Make changes in the UI
2. Export: `ddev drush cex -y`
3. Review changes in `config/sync/`
4. Commit configuration files
5. On other environments: `ddev drush cim -y`

### Working with Migrations
1. Check status: `ddev drush ms`
2. Run migration: `ddev drush mim migration_id`
3. Rollback if needed: `ddev drush mr migration_id`
4. Reset and re-run: `ddev drush mr migration_id && ddev drush mim migration_id`

## File Locations

- **Custom modules**: `web/modules/custom/`
- **Custom themes**: `web/themes/custom/`
- **Configuration**: `config/sync/`
- **Contributed modules**: `web/modules/contrib/`
- **Contributed themes**: `web/themes/contrib/`
- **Libraries**: `web/libraries/`
- **Drush commands**: `drush/Commands/contrib/`
- **Private files**: Not in repository
- **Public files**: `web/sites/default/files/` (not in repository)

## Important Notes

- Configuration is managed in code via `config/sync/` - always export changes
- The site uses Composer for dependency management - never commit `vendor/` or `web/core/`
- Theme uses Tailwind with custom Patriots color scheme
- Many views are configured - check existing views before creating new ones
- Migration source data is stored in custom modules (player_stats, pfr_data)
- The dynasty_module contains a DynastyHelpers class with shared utility functions
- Podcast episodes automatically generate PFR and Wikipedia links on save
- The site uses Gutenberg editor for content editing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oooShiny) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
