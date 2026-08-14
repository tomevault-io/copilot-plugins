## lunaeria

> Lunaeria Site is the production website for Lunaeria.

# Lunaeria Site - Codex Project Memory

## Project Overview

Lunaeria Site is the production website for Lunaeria.

Production URL:

https://lunaeria.vercel.app

The site is a Next.js application hosted on Vercel. Supabase is the main data source for the application.

This file exists so Codex can quickly understand the project architecture, ownership boundaries, and safety rules at the start of each session.

## Core Architecture

- The frontend is a Next.js site.
- Vercel hosts and deploys the production website.
- Supabase stores the primary application data.
- Admin systems write and manage data through the site.
- The Discord bot is a separate project and must not be treated as part of the website runtime.

The website must remain able to run independently from any local Discord bot process.

## Vercel Role

Vercel is responsible for hosting, building, and serving the production website.

Important rules:

- Keep deployment compatibility with Vercel.
- Do not introduce local-only runtime dependencies.
- Do not require a local bot, background process, or machine-specific service for the website to work.
- Preserve existing Next.js and Vercel workflows unless an explicit change is requested.

## Supabase Role

Supabase is the source of truth for the site data.

Important rules:

- The site reads from Supabase for application data.
- The site writes data to Supabase through existing workflows.
- Admin features must continue to use Supabase correctly.
- Do not replace Supabase with local files, hardcoded state, or bot-dependent storage.
- Do not change database assumptions, table usage, or query behavior without first understanding the existing implementation.

## Discord Bot Relationship

The Discord bot is a separate project.

Important rules:

- The website must not depend directly on the local bot.
- The bot may read or synchronize some data from Supabase.
- The data relationship flows through Supabase, not through direct coupling between the website and the bot.
- Do not import bot code into the site.
- Do not make the site require the bot to be running.

Expected relationship:

1. The site writes and manages data in Supabase.
2. Supabase stores the shared data.
3. The Discord bot reads or synchronizes selected data from Supabase.

## Announcements Workflow

Announcements are part of the existing site and admin workflows.

Important rules:

- Preserve the current announcement creation, editing, display, and publishing behavior.
- Do not change how announcements are stored unless explicitly requested.
- Do not break public announcement rendering.
- Do not break admin announcement management.
- Check both frontend display and admin behavior when modifying announcement-related code.

## Admin Systems

Admin features are a critical part of the project.

Important rules:

- Do not break admin authentication, navigation, forms, data loading, or data writes.
- Keep admin changes minimal and scoped.
- Preserve existing admin workflows unless a specific change is requested.
- When touching admin code, verify the related public-facing behavior if data is shared.

## Responsive Mobile

Mobile responsiveness is required.

Important rules:

- Do not make desktop-only changes that degrade mobile.
- Check responsive behavior for pages and components touched by a change.
- Preserve readable text, usable navigation, and non-overlapping layouts on mobile.
- Avoid layout changes with broad side effects unless explicitly requested.

## Change Conventions

Before making any modification:

1. Analyze the existing project structure.
2. Identify the exact components, pages, APIs, or utilities involved.
3. Understand current data flow, especially Supabase interactions.
4. Modify only what is necessary.
5. Verify the affected workflow after the change.

Implementation rules:

- Prefer minimal, targeted changes.
- Follow existing patterns and naming conventions.
- Do not perform global refactors without explicit instruction.
- Do not rewrite unrelated code.
- Do not change architecture casually.
- Do not introduce new dependencies unless they are clearly necessary.
- Keep changes compatible with the existing production deployment.

## Never Break

Never break or casually rewrite:

- frontend pages and navigation;
- admin systems;
- responsive mobile behavior;
- Supabase reads and writes;
- announcement workflows;
- existing production workflows;
- Vercel deployment compatibility;
- the separation between the website and the Discord bot.

## Safety Checklist For Codex

For every task, Codex should ask:

- Which existing files implement this behavior?
- Is this frontend, admin, Supabase, deployment, or bot-adjacent?
- Can the change be smaller?
- Could this break mobile?
- Could this break production on Vercel?
- Could this incorrectly couple the site to the Discord bot?
- Does this affect existing Supabase data or workflows?

If the answer is uncertain, inspect the code before editing.

---
> Source: [Want3d34/lunaeria](https://github.com/Want3d34/lunaeria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
