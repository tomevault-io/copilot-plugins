## choice-controls

> Question rows are a wash, not a checkbox; pickers that still need a mark use ChoiceMark


# Choice controls

Ask-user option rows are the wash only. No `ChoiceMark`, no trailing check, no index.

Do not ship `<input type="checkbox">` or `<input type="radio">` as the visible control. Native chrome follows the OS (Aqua, Fluent) and does not match Lyra's ink/accent cards.

Keep a visually hidden native input (`sr-only`) when a real form control is needed so tests and assistive tech still see `input:checked`.

Selected rows use `bg-accent/[0.08]`. Idle rows use the bubble wash `bg-card`, not `bg-elevated`. No hairline between rows.

The recommended chip sits as a sibling of the title-and-description stack and stays vertically centered on that block.

`ChoiceMark` stays for pickers that are actually on/off controls (model fetch, and the like).

---
> Source: [kittors/Lyra](https://github.com/kittors/Lyra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
