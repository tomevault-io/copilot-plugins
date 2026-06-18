## pptxskill

> Use this skill to turn a go-to-market "motion" (a sales/partner play with audience-specific paths and KPIs) into a single, polished PowerPoint slide. Trigger whenever the user wants to summarize a motion, strategy play, or whiteboard/photo of a delivery plan onto ONE slide — especially with two audience tracks (e.g. SI vs ISV, Migration vs Adoption), KPIs, an enablement vehicle, and an "ask". Also use when the user references a whiteboard image of a business motion and wants it translated into a clean executive slide. Produces a Microsoft Fluent-styled .pptx via a parametric PptxGenJS generator (motion.js) driven by a JSON config. Do NOT use for generic multi-slide decks (use the base pptx skill) — this is specifically for one-slide motion summaries.


# pptxmotions

Turn a **go-to-market motion** into **one** clean, executive-ready slide.

A "motion" here = a delivery/partner play that usually has:
- **two audience paths** (e.g. *System Integrators* vs *ISVs*, *Migration* vs *Adoption*),
- a **KPI** per path,
- an **enablement vehicle** (e.g. "SQL in a Day", "Micro Hack"),
- and an **ask** (sponsorship, funding, meetings).

This skill renders that structure with a consistent **Microsoft Fluent** visual
system (navy / blue / teal, Segoe UI) using a parametric generator.

## Quick Reference

| Task                              | How                                                        |
| --------------------------------- | --------------------------------------------------------- |
| Install as a Copilot CLI skill    | Clone into `~/.copilot/skills/pptxmotions` (see README)   |
| Build a slide from a config       | `node motion.js my-motion.json out.pptx`                  |
| See the schema                    | Read [README.md](README.md)                               |
| Start from an example             | Copy [examples/demo-motion.json](examples/demo-motion.json) |
| QA render to PNG (LibreOffice)    | `soffice --headless --convert-to png --outdir out out.pptx` |

## Workflow

1. **Gather the content.** If the source is a whiteboard photo, read the image and
   extract *every* element (titles, both paths, KPIs, partners, vehicle, the ask).
   Translate to the target language if requested.
2. **Write a JSON config.** Copy an example and fill in `header`, `pathways` (1–2),
   and `bottom` (`work` / `vehicle` / `ask`). See the schema in README.md.
3. **Generate:** `node motion.js my-motion.json out.pptx`
4. **QA (required).** Render to PNG and visually inspect: no overflow, no overlap,
   strong contrast, all source content present. Fix the config and re-render.

## Design System

- **Layout:** 16:9 widescreen (13.3" × 7.5"). Dark navy header band with a teal
  underline; two light pathway cards with a colored left accent bar; a dark
  bottom band (`work` panel + centered `vehicle` highlight + `ask` panel).
- **Palette (default):** navy `0B2447`, blue `0078D4`, teal `00B4A6`, ink `1B2A3A`,
  card tints `EAF3FB` / `E6F7F4`. Override per-deck via `theme` in the config.
- **Type:** Segoe UI Semibold (headers) + Segoe UI (body). Title 25pt, section
  headers 13pt, body 11–12pt.
- **Motif:** colored left accent bar on every card/panel; rounded "pill" tags and
  chips; one accent-colored highlight line per bullet list.

## Dependencies

- Node.js + `pptxgenjs` (`npm install pptxgenjs` — already vendored in this skill).
- LibreOffice (`soffice`) for PNG/PDF QA rendering (optional but recommended).

## Examples

| File | Renders |
| ---- | ------- |
| [examples/demo-motion.json](examples/demo-motion.json) | Fictional "Contoso Cloud" motion: migration (SI) + adoption (ISV), "Cloud in a Day" vehicle |

The example uses **fake sample data** — swap in your own content. A rendered
preview lives in `assets/demo-motion.png`.

## Installation

This is a folder-based Copilot CLI skill. Install by cloning the repo into
`~/.copilot/skills/pptxmotions` and running `npm install`, then restart the CLI
and check `/skills`. Full steps (macOS / Linux / Windows) are in the README.

---
> Source: [fredgis/pptxskill](https://github.com/fredgis/pptxskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
