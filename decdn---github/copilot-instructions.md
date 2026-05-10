## github

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is the **org-level `.github` repo** for the `decdn` GitHub organization (`git@github.com:decdn/.github.git`). It is GitHub's special "community health" repo: files placed here become **defaults for every repo in the `decdn` org that does not ship its own equivalent**.

Currently empty (no commits). Anything added here has org-wide blast radius — changes affect `decdn/decdn`, `decdn/finance`, `decdn/website`, and any future repos. Treat additions accordingly.

## What belongs here vs. elsewhere

| Path in this repo | Effect |
|-------------------|--------|
| `profile/README.md` | Renders on the `decdn` GitHub org profile page. |
| `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, `CODE_OF_CONDUCT.md`, `FUNDING.yml` | Org-wide defaults. A repo's own copy overrides these. |
| `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md` | Default issue/PR templates for repos without their own. |
| `workflow-templates/` | Starter workflows offered to org members in the Actions UI (these are *templates*, not workflows that run). |

**Does not belong here:** repo-specific CI workflows, repo-specific docs, code. Those live in the per-repo `.github/` directory inside `decdn/`, `finance/`, or `website/`.

## Before adding anything

1. Check whether the per-repo `.github/` directory in `decdn/`, `finance/`, or `website/` already provides the file — a repo-local copy overrides this org-level default and may be intentional.
2. Confirm the change is appropriate for *all three* (and any future) org repos. If it's only right for one, it belongs in that repo, not here.
3. The parent workspace `CLAUDE.md` at `/home/thiras/dev/decdn/CLAUDE.md` is the entry point for understanding the org's repos and their cross-dependencies.

---
> Source: [decdn/.github](https://github.com/decdn/.github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
