## klinip

> For this project only, every frontend or visual UI change must include all of these checks in the same task:

# Klinip Project Rules

## Frontend UI Standard

For this project only, every frontend or visual UI change must include all of these checks in the same task:

1. `ui-theme-parity`
   Every visual change must work correctly in light mode and dark mode.
   Review cards, dialogs, filters, forms, tables, charts, labels, helper text, icons, borders, empty states, and interactive states.

2. `mobile-compact-ui`
   Every visual change must be reviewed on phone-sized screens.
   Reduce unnecessary height, spacing, duplicated blocks, and oversized controls so the mobile UI stays compact and readable.

3. `ui-copy-integrity`
   Every frontend or user-facing text change must be reviewed for encoding problems, broken accented characters, mojibake, and spelling mistakes.
   Review headings, cards, dialogs, filters, labels, helper text, empty states, alerts, onboarding copy, navigation text, AI summaries, and landing content.

## Done Criteria For UI Work

A frontend task in Klinip is not complete until:

- Light mode is correct.
- Dark mode is correct.
- Mobile layout is compact and usable.
- Visible text has no encoding corruption or obvious spelling mistakes.
- Cards, filters, dialogs, charts, and summaries were checked if the change touches them.

## Reporting

When closing a frontend task for Klinip, explicitly mention:

- light/dark theme support
- mobile compact adjustments
- UI copy integrity checks

## Encoding y caracteres

- Todos los archivos del proyecto se guardan en UTF-8 sin BOM.
- En Python, cualquier lectura o escritura de texto debe usar `encoding="utf-8"` explicito.
- No introducir strings hardcodeados con mojibake, Latin-1 roto o caracteres corruptos.
- Si se generan archivos `.md`, `.txt`, `.json` o `.csv`, deben escribirse en UTF-8 explicito.
- La salida del servidor debe quedar configurada en UTF-8 al arranque para evitar corrupcion en logs y textos.
- Consultar `prevencion_caracteres_corruptos.md` como referencia operativa antes de corregir o generar contenido textual.

## Skills
A skill is a set of local instructions to follow that is stored in a `SKILL.md` file. Below is the list of skills that can be used. Each entry includes a name, description, and file path so you can open the source for full instructions when using a specific skill.
### Available skills
- mobile-compact-ui: Ensure every frontend or visual UI change is reviewed for phone-sized screens with compact spacing and readable density. Use when modifying pages, components, cards, filters, forms, tables, dialogs, headers, bottom navigation, dashboards, charts, or any responsive CSS that can consume too much space on mobile. (file: C:/Users/hp/.codex/skills/mobile-compact-ui/SKILL.md)
- ui-theme-parity: Ensure every frontend or visual UI change ships with explicit support for both light and dark themes. Use when modifying pages, components, cards, dialogs, charts, forms, tables, filters, headers, navigation, mobile layouts, or any CSS/design token behavior that could look different across themes. (file: C:/Users/hp/.codex/skills/ui-theme-parity/SKILL.md)
- ui-copy-integrity: Verify visible UI copy for encoding issues, mojibake, malformed accents, stray replacement characters, and spelling mistakes. Use when modifying frontend text, labels, cards, dialogs, helper text, empty states, alerts, onboarding copy, navigation text, landing content, or any user-facing strings in Klinip. (file: C:/Users/hp/Desktop/Klinip/.project-skills/ui-copy-integrity/SKILL.md)

---
> Source: [Diegogigi/klinip](https://github.com/Diegogigi/klinip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
