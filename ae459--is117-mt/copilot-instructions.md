## is117-mt

> This repository is a **museum project**, not just a website. Any future work should preserve the idea of the Marvel Bauhaus Museum as a curated exhibition with durable documentation, not a generic fan page or process demo.

# AGENTS.md

## Purpose

This repository is a **museum project**, not just a website. Any future work should preserve the idea of the Marvel Bauhaus Museum as a curated exhibition with durable documentation, not a generic fan page or process demo.

## Project Identity

- Treat the project as a **genuine museum exhibition** about Marvel history, creators, studios, artifacts, and publication lineage.
- The public-facing experience should feel like an exhibit with rooms, objects, evidence surfaces, captions, and curatorial logic.
- Do **not** turn the visitor-facing site into a page that explains the design process to visitors.
- The site should present artifacts, images, objects, people, chronology, and interpretation.

## Source Of Truth

- `docs/` is the **single source of truth** for this repository.
- The live GitHub Pages site lives in `docs/`.
- The curatorial, specification, sprint, QA, and foundation documentation also lives in `docs/`.
- Do **not** recreate or rely on the old `static-site/` directory. It was intentionally removed.
- When referring to active pages, use paths under `docs/`, such as `docs/index.html`.

## Site And Design Direction

- Use the AI museum project as a **reference for process and structural rigor**, not as something to copy literally.
- Preserve the existing **Bauhaus-inspired visual discipline** adapted to Marvel subject matter.
- Favor strong layout logic, restrained composition, clear hierarchy, and exhibit-style interpretation.
- The site should feel curated, intentional, and educational.
- Avoid decorative drift that weakens the exhibition structure.
- Treat the public entry sequence as part of the exhibition: the first page may function as a threshold or hero splash page, but it should still feel like a museum arrival surface rather than a generic marketing landing page.

## Imagery And Objects

- Prefer **pictures of what is being presented** over abstract decorative shapes.
- Use object-based, comic-based, and portrait-based imagery that supports the exhibit narrative.
- New visuals should behave like museum objects or interpretive supports, not random graphics.
- Keep imagery tied to room purpose, chronology, cast, or evidence.

## Museum Information Architecture

Preserve the exhibition logic:

- chronology is the primary spine
- companion rooms support the chronology
- rooms should feel like parts of one exhibition system
- people, studios, publications, and objects should connect back to the larger narrative

When adding or revising content:

- strengthen chronology rather than flattening it
- prefer interpretive grouping over directory-style listing
- treat publication lineage, people/studios, and collection highlights as supporting curatorial rooms

## Content Rules

- Write like a curator, not like a product marketer.
- Favor interpretation over trivia dumps.
- Explain why an item, person, studio, or publication matters.
- Reduce directory-style or encyclopedic presentation when it weakens the exhibit.
- Maintain coherence across chronology, artifacts, people, and institutional context.

## Visual QA Rules

- Fonts, sizes, spacing, and alignment must match the surrounding system.
- Titles must fit inside their borders and title strips.
- Caption boxes, labels, frames, and nameplates should align consistently.
- Review for overflow, misalignment, uneven spacing, and one-off styling.
- Standardize card systems instead of fixing one element in isolation.

## File Structure Expectations

Key active areas:

- `docs/index.html` — public entry point / hero splash page
- `docs/home.html` — main homepage / opening gallery
- `docs/eras/` — chronology rooms
- `docs/people-and-studios.html` — cast and institution room
- `docs/reading-maps/` — publication lineage room
- `docs/guides/` — collection highlights room
- `docs/media/` — comic, object, and portrait assets
- `docs/foundation/` — durable design and implementation references
- `docs/_specs/` — specs and sprint docs
- `docs/content/` — collection model

## Process Model

Follow the lightweight specs-driven workflow:

**spec -> sprint -> implementation -> QA**

Use the existing process layer in `docs/_specs/` and `docs/foundation/` before making large structural changes.

### When To Use Specs And Sprints

- Major features should not begin without a spec.
- Multi-surface changes should not begin without sprint docs.
- Structural, curatorial, and visual-system changes should be documented.
- Tiny copy edits or trivial fixes may not need a new workstream.

## Sprint Document Requirements

Every sprint doc must include these sections in this order:

1. Goal
2. Scope (with **In** and **Out**)
3. Tasks
4. Files to touch
5. Acceptance criteria
6. Verification checklist
7. Completion evidence

### Completion Evidence Should Cover

Examples include:

- repository paths or URLs
- screenshots
- PR links
- commit hashes
- preview links when applicable

## Editing Rules

- Make focused changes that match the current repository structure.
- Prefer updating existing systems over creating parallel versions.
- Do not reintroduce deleted or duplicate app surfaces.
- Keep documentation aligned with the actual implementation.
- If a file path changes, update the related docs and specs.
- Respect recent edits in sprint/spec files by reading current contents before modifying them.

## QA And Verification

Normal checks for this project include:

- HTML validation
- CSS validation
- SVG validation
- internal link checks
- media-path checks
- visual QA across cards, captions, rooms, and headings
- consistency review across the full exhibition

If files are touched, verify the edited files and review the affected pages as a connected exhibit, not as isolated screens.

## GitHub Pages Deployment

The repo is set up for GitHub Pages using:

- Branch: `main`
- Folder: `/docs`

Deployment assumptions:

- `docs/index.html` is the public Pages entry point
- `docs/home.html` contains the primary opening-gallery homepage reached from the entry page
- `docs/.nojekyll` must remain present so `_specs/` is not ignored

## What To Avoid

- Do not build a process-explainer site for visitors.
- Do not substitute abstract shapes where real exhibit imagery should be used.
- Do not let supporting rooms overpower the chronology.
- Do not allow visual misalignment to accumulate across cards or captions.
- Do not create duplicate source folders when `docs/` is already canonical.
- Do not let documentation drift away from the real implementation.

## Default Goal For Future Agents

When in doubt, choose the option that makes the project feel more like a coherent, curated, well-documented museum exhibition published from `docs/` and maintained through specs, sprints, and QA.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ae459) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
