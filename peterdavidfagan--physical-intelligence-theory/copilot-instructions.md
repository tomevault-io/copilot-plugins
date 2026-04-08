## physical-intelligence-theory

> All figures in the manuscript must use the following colour palette for

## Plotting Colour Palette (Paper Standard)

All figures in the manuscript must use the following colour palette for 
visual consistency and publication quality.

### Primary Colours
- Teal:      #1B9E77
- Orange:    #D95F02
- Indigo:    #7570B3
- Crimson:   #E7298A
- Goldenrod: #E6AB02
- SlateGrey: #666666

### Greyscale Support Colours
- DarkGrey:       #4D4D4D
- MidGrey:        #7F7F7F
- LightGrey:      #BFBFBF
- VeryLightGrey:  #E6E6E6

### Matplotlib Style Settings
Use the following default rcParams:

import matplotlib as mpl

mpl.rcParams.update({
“figure.figsize”: (6, 4),
“figure.dpi”: 300,
“savefig.dpi”: 300,
“font.size”: 12,
“axes.titlesize”: 14,
“axes.labelsize”: 12,
“axes.edgecolor”: “#4D4D4D”,
“axes.linewidth”: 1.0,
“axes.prop_cycle”: mpl.cycler(color=[
“#1B9E77”,  # Teal
“#D95F02”,  # Orange
“#7570B3”,  # Indigo
“#E7298A”,  # Crimson
“#E6AB02”,  # Goldenrod
“#666666”,  # Slate Grey
]),
“xtick.color”: “#4D4D4D”,
“ytick.color”: “#4D4D4D”,
“text.color”: “#3A3A3A”,
“axes.labelcolor”: “#3A3A3A”,
“legend.frameon”: False,
“grid.color”: “#E6E6E6”,
“grid.linestyle”: “–”,
})

### Figure Export
- All final figures must be exported as vector PDFs.
- Use consistent line widths (1.0–1.5) and font sizes (≥12 pt).
- Avoid nonstandard colormaps; use only the palette above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterdavidfagan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peterdavidfagan)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
