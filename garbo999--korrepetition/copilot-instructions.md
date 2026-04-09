## korrepetition

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static HTML time-tracking form ("Korrepetition") for Julia Hess, a musical accompanist at a music school. It tracks accompaniment session minutes across dates and students/groups for monthly accounting.

## Structure

- `form.html` — Self-contained HTML page with inline CSS. No JavaScript or external dependencies.
- `36635.jpg` — Image file (not referenced in the HTML).

## Running

Open `form.html` directly in a web browser. No build step or server required.

## Architecture

The form is a single HTML table with:
- Rows for each student or group (with their teacher noted)
- Columns for specific dates in the month
- A summary section at the bottom tracking carried-over minutes, required monthly minutes, performed minutes, and the resulting balance

All styling is inline via a `<style>` block. The document is in German.

## Conventions

- When creating new monthly forms, update: the month/year in the header, the date columns, and the carried-over balance from the previous month's closing balance.
- The file ends with an artifact line "Copied to clipboard" that should be removed if cleaning up.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/garbo999)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/garbo999)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
