## fraxinus-00tw

> This repository is the hardware repository for Fraxinus 00tw.

# AGENTS

This repository is the hardware repository for Fraxinus 00tw.

## Terms

- `Fraxinus 00tw` is a community project derived from Fraxinus 00w, designed by SummerOrange.
- It is not an official Fraxinus product; it is one project within the Fraxinus community.

## Repository Scope

- Keep all design files that belong to Fraxinus 00tw in this repository.
- Do not add designs from other projects or other community members to this repository.

## Directory Rules

- Place 3D-printable part files under `3MFs/`.
- Place panel cut files under `DXFs/`.
- Organize files within `3MFs/` by functional group (e.g. frames, bed, extruder).

## Naming Rules

- Directory names use PascalCase. For multi-word names, use an underscore separator.
- Directories that follow a widely-used GitHub convention (e.g. `docs/`, `assets/`, `.github/`) are kept lowercase to match that convention.

## Recommended Layout

```text
Fraxinus-00tw/
  README.md
  AGENTS.md
  3MFs/
    <functional group>/
  DXFs/
```

## Editing Guidance

- When adding a new part, place it in the appropriate functional group directory under `3MFs/`.
- When adding a new panel file, place it under `DXFs/`.
- Keep the README focused on what the project is and how to get started.
- Do not put maintenance policy or repository housekeeping rules in the README.

## Language Rules

- README files in this repository are written in Japanese.
- When editing a README.md, also update the corresponding README.en.md in the same directory. If README.en.md does not exist, create it.
- README.en.md must reflect the same content as README.md. Do not add or omit information between the two.
- Short supplemental README files may include Japanese and English in one README.md instead of creating a separate README.en.md.
- In Japanese README text, do not pack long explanations into a single bullet line. Put the lead phrase on the bullet line, then continue the explanation on the following indented line.

## Documentation Guidance

- Treat the README as an entry point for builders who want to use this design.
- Keep maintainer-only guidance out of the README even if it affects how the repository is organized.

---
> Source: [Fraxinus-3D/Fraxinus-00tw](https://github.com/Fraxinus-3D/Fraxinus-00tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
