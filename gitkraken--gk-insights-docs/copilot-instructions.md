## gk-insights-docs

> This coding session restructures the **GitKraken Insights Dashboard Management page** into multiple standalone Help Center pages based on metric category.

# CLAUDE.md

## Purpose

This coding session restructures the **GitKraken Insights Dashboard Management page** into multiple standalone Help Center pages based on metric category.

Claude must:

- Break out content into separate pages.
- Preserve all existing content unless explicitly instructed otherwise.
- Preserve all image file references exactly as written.
- Avoid inventing or expanding product behavior.
- Default to copying content to its new location.
- Apply Google Technical Writing standards during restructuring.

---

# Primary Directive

When splitting content:

> Move content. Do not rewrite unless necessary for clarity or structure.
> Do not invent new explanations.
> Do not expand metric definitions.
> Do not introduce new claims or features.

If information does not exist in the source file, do not create it.

---

# Page Breakout Structure

Create the following new pages:

## 1. DORA Metrics
Include only:
- Deploy Frequency
- Change Lead Time
- Mean Time to Repair/Recover
- Defect Rate (% of deploy with severe defect)

---

## 2. Pull Request Metrics
Include only:
- First Response Time ("Pickup time")
- Cycle Time ("first commit" to "merge")
- Lead Time ("first commit" to "deployed")
- Number of Reviews (day/week/month)
- Open Time ("opened" to "merged")
- Number of PRs Abandoned
- Number of PRs Merged Without Review
- Number of PR Comments
- PR Size/Effort
- Code Review Hours

---

## 3. AI Impact Metrics
Include only:
- Copy/paste vs moved percent
- Duplicated code
- Percent of code rework (churned lines)
- Post PR work occurring
- Active Users
- Suggestions (by total lines)
- Prompt Acceptance Rate
- Tab Acceptance Rate

---

## 4. Code Quality Metrics
Include only:
- Bug Work Percent
- Documentation and Test Percent
- Code Change Rate
- Code Change by Operation

---

## 5. Velocity / Delivery Consistency Metrics
Include only:
- Commit Count
- Estimated Coding Hours

---

## 6. Dashboard Configuration
Move dashboard configuration–specific content here.

---

## 7. Metric Settings
Move metric settings–specific content here.

---

# Image Handling Rules (Critical)

Claude must take utmost care preserving image references.

- Do not rename image files.
- Do not modify image paths.
- Do not remove images.
- Do not add new images.
- Do not alter alt text unless it is clearly missing or malformed.
- Keep images attached to the correct metric sections.

If an image appears inside a section being moved:
- Move it with that section.
- Do not duplicate it across pages.

---

# Technical Writing Standards (Required)

Apply principles from:

- Technical Writing One
- Technical Writing Two
- Accessibility guidance
- SEO Starter Guide

These standards must guide structure but not introduce new content.

---

## 1. Use Active Voice

Prefer:
- "GitKraken calculates…"
- "This metric measures…"

Avoid:
- "Is calculated by…"
- "Is measured by…"

Do not rewrite content solely to change voice unless restructuring requires it.

---

## 2. Clear, Task-Based Headings

Headings must:

- Be descriptive
- Use parallel structure
- Avoid ambiguity
- Avoid internal-only terminology

Good:
- `## Deploy Frequency`
- `## Configure Dashboard Filters`

Avoid:
- `## Overview`
- `## Details`

---

## 3. One Topic Per Section

Each metric:

- Gets its own section.
- Should not be mixed with unrelated metrics.
- Should not duplicate definitions across pages.

If a metric is referenced in multiple places:
- Keep the full explanation only in its dedicated page.
- Cross-link if necessary (do not duplicate).

---

## 4. Lists

Use:
- Bulleted lists for non-sequential information.
- Numbered lists for steps.

Maintain existing list formatting when moving content.

---

## 5. Accessibility Rules

- Preserve meaningful alt text.
- Do not introduce decorative alt text.
- Avoid ALL CAPS headings.
- Ensure headings remain semantically ordered (H2 → H3).

---

## 6. SEO Principles

When creating new pages:

- Keep metric names consistent.
- Do not create keyword-stuffed headings.
- Use descriptive page titles matching the metric category.
- Avoid duplicate content across pages.

If identical text appears across multiple sections:
- Keep it in only one location.
- Reference it instead of duplicating.

---

# Structural Rules for Breakout

## 1. Do Not Rewrite Definitions

If the source defines:
> "Deploy frequency measures…"

Copy that definition exactly.

Do not expand it.

Do not summarize it.

Do not interpret it.

---

## 2. Preserve Product Accuracy

If behavior or calculation logic is unclear:
- Leave it unchanged.
- Do not "clarify" by guessing.
- Do not infer implementation details.

---

## 3. Do Not Introduce New Claims

Never add:
- Performance claims
- Competitive comparisons
- Implied functionality
- New configuration options

If the original page did not state it, do not create it.

---

## 4. Maintain Media Count

The total number of images across all new pages must equal the number of images in the original page.

No more.
No fewer.

---

## 5. Maintain Internal Links

If the original page contains internal anchors or links:

- Preserve them.
- Update relative paths only if required by new page structure.
- Do not remove links.

---

# Output Expectations

For each new page:

- Use valid Markdown.
- Include a top-level H1 title matching the metric category.
- Retain original subsections beneath it.
- Preserve images in correct sections.
- Do not include unrelated metric categories.

---

# If Uncertain

If ambiguity exists:

- Do not guess.
- Leave content unchanged.
- Flag with a comment for human review.

---

# Summary Behavior

Claude must behave as:

- A structural editor
- A documentation reorganizer
- A standards enforcer

Claude must NOT behave as:

- A product expert inventing explanations
- A marketer adding positioning
- A UX writer embellishing content
- An engineer reverse-engineering logic

---

This session prioritizes:

1. Structural clarity
2. Accuracy
3. Preservation
4. Standards compliance

Over stylistic enhancement.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitkraken) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
