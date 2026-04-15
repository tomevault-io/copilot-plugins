## php-scoper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a wrapper library for [PHP Scoper](https://github.com/humbug/php-scoper) that packages the PHAR file for distribution via Composer. The repository is **automatically updated daily** via GitHub Actions to track the latest PHP Scoper releases.

**Key characteristic**: This is a wrapper/distribution project with minimal custom code. The main asset is `bin/php-scoper.phar`, which is downloaded from the upstream humbug/php-scoper project.

## Architecture

### Core Components

1. **bin/php-scoper.phar**: The PHP Scoper PHAR file downloaded from humbug/php-scoper releases
2. **composer.json**: Declares the PHAR as a binary and tracks PHP version requirements from upstream
3. **.github/scripts/update-php-scoper.php**: Automated update script that:
   - Fetches the latest release from humbug/php-scoper via GitHub API
   - Downloads the PHAR file to bin/
   - Updates composer.json with the required PHP version from upstream
   - Uses MD5 checksums to detect actual changes
   - Sets environment variables for the GitHub Actions workflow

### Automated Update Flow

The repository uses a scheduled GitHub Action (`.github/workflows/update-php-scoper.yml`) that:
- Runs daily at 00:00 UTC (or manually via workflow_dispatch)
- Executes update-php-scoper.php to check for new releases
- Creates a pull request if a new version is detected
- Creates a git tag matching the upstream version after update

**Version tracking**: Git tags match upstream PHP Scoper versions (e.g., 0.18.11, 0.18.14)

## Development

### Manual Update Testing

To manually test the update process:
```bash
php .github/scripts/update-php-scoper.php
```

This script compares the latest git tag with the latest upstream release.

### Triggering Updates

The GitHub Actions workflow can be triggered:
- Automatically via daily cron schedule
- Manually via GitHub Actions UI (workflow_dispatch)

### Version Management

Versions are managed through git tags that correspond to upstream PHP Scoper releases. The update script automatically:
1. Compares current version (latest git tag) with latest upstream release
2. Only updates if upstream version is newer
3. Updates both the PHAR and PHP version requirement in composer.json

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wpify)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wpify)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
