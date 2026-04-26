## pdf-backend

> CID Workflow — SVG/JSON mapping, Mapper tools, template layout. Use when working on form mapping, mapper.html, or PDF templates.


# CID Workflow

## What this is
- **Mapper**: click-to-place form fields on a 612×792 page (SVG or PDF as background), output `cid.map.v1` JSON.
- **Tools**: `CID_HomeBase/tools/mapper/mapper.html` (load SVG, place fields, Copy JSON) and `CID_HomeBase/tools/mapper/pdf-draft.html` (load PDF, extract AcroForm fields, open in Mapper).
- **Coordinates**: All mapping uses **base units** 612×792 (PDF/SVG truth). No scaling in the JSON.

## Template layout (this repo = BAR)
- **SVGs**: `CID_HomeBase/templates/<TEMPLATE>/assets/page-N.svg` (e.g. `SUPP_BAR/assets/page-1.svg`).
- **Mapping JSON**: `CID_HomeBase/templates/<TEMPLATE>/mapping/page-N.map.json`.
- **JSON schema**: `cid.map.v1`, `units`: `base_612x792`, `page`: `{ "width": 612, "height": 792 }`.
- **Field types**: `text`, `checkbox`, `multiline`. Text fields have `w`, `h`, `fontSize`; checkboxes have `size`. Anchor can be top-left or center.

## Conventions
- Field names: snake_case (e.g. `agency_name`, `producer_phone`).
- One mapping file per page; template and pageId must match the folder and filename.

## When starting a new CID Workflow chat
- Use the **CID Workflow** agent and say what you’re doing (e.g. “mapping SUPP_BAR page-2” or “fix SVG viewBox for 612x792”).
- The agent can read this rule and the repo; no need to re-explain the whole workflow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/G-Love53) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
