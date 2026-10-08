## email-skills

> Skills for AI coding agents focused on email marketing tasks.

# Email Marketing Skills

Skills for AI coding agents focused on email marketing tasks.

## Structure

All skills live in `skills/<skill-name>/SKILL.md`. Each is standalone markdown with YAML frontmatter containing `name`, `description`, and `category`.

One skill (`email-deliverability-expert-skill`) includes a `references/` subfolder with supplementary material.

## Usage

When the user asks about email marketing topics (deliverability, compliance, copywriting, automation, analytics, list growth, segmentation, bounces, warmup, campaign strategy), check the relevant skill file for specialized knowledge.

Skills reference each other. Follow cross-references when the user's question spans multiple areas.

## Available skills

- `ab-testing` — Email A/B testing methodology
- `api-campaign-builder` — SendX API campaign management
- `campaign-strategy` — Campaign type selection and planning
- `compliance-navigator` — Email privacy law compliance (CAN-SPAM, GDPR, CASL, etc.)
- `email-analytics` — Metric interpretation and action planning
- `email-copywriting` — Subject lines, preview text, and body copy
- `email-deliverability-expert-skill` — Deliverability audit, diagnosis, and fixes
- `esp-hardbounce-unlock` — Hard bounce unlock evaluation
- `esp-malicious-behavior` — Malicious sender investigation
- `industry-playbooks` — Vertical-specific email strategies
- `list-growth-engine` — List building with popups, forms, and lead magnets
- `smart-segmentation` — List segmentation and targeting
- `workflow-automation` — Automated email sequences

## Context

Built by SendX (email marketing platform) and SendPost (email infrastructure API). Target audience is non-technical email marketers. Use plain language.

---
> Source: [sendx/email-skills](https://github.com/sendx/email-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
