## aliossfortypecho

> This repository is a Typecho plugin that redirects attachment storage to Alibaba Cloud OSS. [`Plugin.php`](/home/mango/code/AliOssForTypecho/Plugin.php) contains the main plugin class, hook registration, OSS client setup, and upload/delete handlers. [`oss-files.php`](/home/mango/code/AliOssForTypecho/oss-files.php) provides the admin file manager UI and AJAX endpoints. Bundled third-party code lives in `oss/` and `assets/photoswipe/`; treat those as vendored assets and avoid editing them unless upgrading dependencies. Primary docs are in `README.md`.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Typecho plugin that redirects attachment storage to Alibaba Cloud OSS. [`Plugin.php`](/home/mango/code/AliOssForTypecho/Plugin.php) contains the main plugin class, hook registration, OSS client setup, and upload/delete handlers. [`oss-files.php`](/home/mango/code/AliOssForTypecho/oss-files.php) provides the admin file manager UI and AJAX endpoints. Bundled third-party code lives in `oss/` and `assets/photoswipe/`; treat those as vendored assets and avoid editing them unless upgrading dependencies. Primary docs are in `README.md`.

## Build, Test, and Development Commands

There is no build step. Work directly in PHP files and validate syntax before opening a PR.

- `php -l Plugin.php` checks the main plugin file for parse errors.
- `php -l oss-files.php` checks the admin panel page.
- `php -l *.php` is a quick repo-wide syntax pass for top-level PHP files.

For manual verification, install the plugin under `usr/plugins/AliOssForTypecho`, activate it in Typecho, then test upload, replace, delete, and OSS file listing flows from the admin panel.

## Coding Style & Naming Conventions

Use 4-space indentation and keep brace placement consistent with the existing code. Follow current PHP naming patterns: class names in `PascalCase`, methods in `camelCase`, and constants in `UPPER_SNAKE_CASE` (for example `UPLOAD_DIR`). Keep Typecho hook handlers static where appropriate, and prefer short, defensive checks around OSS and admin operations. Preserve existing inline comments when modifying Typecho-specific behavior.

## Testing Guidelines

This repo does not include an automated test suite yet. At minimum, run PHP syntax checks and perform manual regression testing in a Typecho instance with valid OSS credentials. Focus on upload lifecycle hooks, attachment URL generation, admin permissions, pagination, and delete behavior. If you add new PHP entry points, name them clearly by feature, following existing patterns like `oss-files.php`.

## Commit & Pull Request Guidelines

Recent commits use short, imperative subjects such as `Fix admin permission check for AJAX endpoints` and `Add image preview with zoom support`. Keep commit titles concise, capitalized, and behavior-focused. Pull requests should describe the user-visible change, note any Typecho or OSS configuration assumptions, and include screenshots for admin UI updates. Mention manual test coverage in the PR body.

---
> Source: [TeohZY/AliOssForTypecho](https://github.com/TeohZY/AliOssForTypecho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
