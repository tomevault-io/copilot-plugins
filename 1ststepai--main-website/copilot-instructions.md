## main-website

> This project is the 1stStep.ai business website and lead funnel.

# 1stStep.ai Codex Instructions

This project is the 1stStep.ai business website and lead funnel.

## Business Context

1stStep.ai builds practical apps, websites, dashboards, internal tools, and workflow automations for founders, small businesses, and professionals.

Core positioning:

Turn your app idea or business workflow into a working first version.

Do not describe 1stStep.ai as generic AI consulting. Do not overuse buzzwords. Do not promise guaranteed revenue, rankings, funding, or app success.

## Primary Conversion Goal

Send qualified visitors to `/app-idea-viability-checker.html`.

Primary CTA:

Check Your App Idea

Secondary CTA:

Book a Build Strategy Call

## Preserve

- `/app-idea-viability-checker.html`
- `/api/app-idea-checker`
- Resend admin notification
- `localStorage` key `appIdeaCheckerLastResponse`
- deterministic context pack generation
- `repo_creation_status: locked_until_signed_and_paid`
- GHL forwarding disabled unless explicitly enabled

## Constraints

- Do not introduce paid AI APIs unless explicitly requested.
- Do not enable GHL forwarding unless explicitly requested.
- Do not enable repo creation.
- Do not add fake reviews, fake logos, fake metrics, or unverifiable claims.
- Do not mix Real Rank/local SEO product positioning into the 1stStep.ai main site.

## Improvement Priorities

When improving the site:

1. Improve clarity.
2. Improve trust.
3. Improve CTA visibility.
4. Reduce friction.
5. Route users to the checker.
6. Keep the build simple and production-safe.

## Required Skill: Premium Web Design

For all public website, landing page, homepage, hero, pricing, testimonial, portfolio, case study, and screenshot-to-code tasks, read and apply:

`.agents/skills/premium-web-design.md`

The design reference is the source of truth. Do not improvise a generic layout. Implement, render, screenshot, compare, and iterate before reporting done.

## Required Skill: Visual Asset Rendering

For any request to create images, mockup pictures, hero graphics, pricing visuals, testimonial visuals, social banners, OG images, or marketing graphics, read and apply:

`.agents/skills/visual-asset-rendering.md`

Default method:

Build the visual as an HTML/CSS composition, render it in browser, screenshot it with Playwright or the available browser screenshot workflow, and save the final PNG/WebP.

Do not respond with only code or instructions. Produce the actual image file whenever possible.

## Optional Generic Methodology Layer: Superpowers

For planning, debugging, implementation discipline, verification, and code review workflows, agents may read and apply:

`.agents/skills/generic/superpowers-methodology.md`

Superpowers is a lower-priority generic methodology layer. It must never override this file, project-specific 1stStep.ai skills, user instructions, brand/design/conversion rules, App Idea Checker constraints, route preservation, Resend behavior, GHL forwarding safeguards, or repo creation lock rules.

Priority order:

1. Current user request.
2. Project-specific `AGENTS.md` / `CLAUDE.md` instructions.
3. 1stStep.ai skills in `.agents/skills/firststep/`.
4. Required design/rendering skills when applicable.
5. Superpowers methodology.
6. Default agent behavior.

Do not use Superpowers to add secrets, credentials, paid APIs, new runtime behavior, GitHub repo creation, GHL forwarding, or app code changes unless the user explicitly asks for those changes.

## UI/UX Guardrails

See [AI_UI_UX_GUARDRAILS.md](./AI_UI_UX_GUARDRAILS.md) for UI/UX Pro Max design rules and project-specific brand constraints.

---
> Source: [1ststepai/main-website](https://github.com/1ststepai/main-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
