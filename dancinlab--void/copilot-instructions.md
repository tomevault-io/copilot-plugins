## void

> @V := "tape" :: spec [active]

@V := "tape" :: spec [active]
  version = "1.2"

@I := "void" :: identity [active]
  kind     = "Grid-first terminal (Ghostty hard fork · N×M tiling as core rendering surface · 🕳️)"
  brief    = "Beta: grid mode only. P1 grid-mode, dancinlab hexagon icon."
  parent   = "dancinlab"
  ssot     = "github.com/dancinlab/void (`hx install void`)"
  siblings = "hexa-lang"

@D grid_auto_layout := "N×M grid topology auto-rebalance" :: governance [required active]
  do   = "grid auto-layout preserves cols = ⌈√N⌉, rows = ⌈N/cols⌉, cols ≥ rows; per-cell cwd isolation"
  dont = "break grid topology on cell-count change · add manual resize handles / splits (auto-only)"

---
> Source: [dancinlab/void](https://github.com/dancinlab/void) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
