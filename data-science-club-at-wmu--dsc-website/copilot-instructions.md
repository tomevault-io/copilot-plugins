## dsc-website

> Notes for Claude Code sessions working on this repo.

# CLAUDE.md

Notes for Claude Code sessions working on this repo.

## Chat attachments ARE retrievable — don't claim otherwise

Images/files the user uploads to the conversation are embedded as base64 in the
session transcript at `~/.claude/projects/<project-slug>/<session-id>.jsonl`.
If an attachment is needed as an actual file, extract just the image data
blocks (e.g. `grep -o '"data":"[A-Za-z0-9+/=]\{5000,\}"' <transcript> | sed
's/^"data":"//; s/"$//' | base64 -d > out`) rather than telling the user the
upload can't be accessed. Note that images returned by the Read tool are
embedded in the transcript the same way — dedupe candidates by byte-comparing
against existing files in the repo.

## PR workflow — check merge state before reusing anything

The user merges PRs fast, often minutes after they're opened. Before pushing
follow-up commits to the session branch or touching an existing PR's
title/body, re-fetch and check whether that PR already merged. Once merged:
rebase any unmerged commits onto fresh `origin/main` (dropping the merged
ones), force-with-lease push, and open a NEW PR — pushes to a merged PR's
branch don't reopen it, and editing a merged PR's description corrupts the
historical record of what it shipped.

## Site quick reference

- The homepage "slideshow" is the About-section gallery: `GALLERY_IMAGES` in
  `components/sections/About.tsx`, image files in `public/` (WebP preferred,
  photos are 1024x768).

---
> Source: [Data-Science-Club-at-WMU/dsc-website](https://github.com/Data-Science-Club-at-WMU/dsc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
