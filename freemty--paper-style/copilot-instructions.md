## paper-style

> >


# Paper Style — Academic Color Theme System

5 low-saturation themes (red/blue/gold/green/purple) for unified LaTeX + matplotlib visual identity.

## Subcommands

| Command | Guide | What it does |
|---------|-------|-------------|
| `/paper-style init [--theme NAME] [--inject]` | `guides/init.md` | Scaffold project with theme |
| `/paper-style` or `/paper-style guard` | `guides/guard.md` | Detect theme, assist with figures/tables |

**Arguments for init:**
- `--theme NAME`: red (default), blue, gold, green, purple
- `--inject`: skip `mystyle.cls`, copy color files + print integration instructions for existing cls

**Read and execute the corresponding guide file.**

## Themes

| Name | Accent | Style |
|------|--------|-------|
| `red` | `#5E3545` | Warm burgundy |
| `blue` | `#3C4A57` | Desaturated slate |
| `gold` | `#564D3D` | Warm beige |
| `green` | `#2A4438` | Deep forest |
| `purple` | `#3E2548` | Deep plum |

## Templates

| File | Purpose |
|------|---------|
| `templates/colors.tex` | 5-theme LaTeX color definitions (`\ifthenelse` switch) |
| `templates/mystyle.cls` | Document class (CC BY-SA 4.0, Palatino/XCharter) |
| `templates/preamble.tex` | Shared preamble (hyperref, tcolorbox, abox) |
| `templates/paper_palette.py` | Python: `get_theme()`, `apply_theme()`, `get_colormap()`, `clean_ax()`, `theme_names()` |
| `templates/academic.mplstyle` | matplotlib: serif, 300 DPI, clean spines |

## Quick Reference — Guard Mode

**Python figures:**
```python
from paper_palette import apply_theme, get_colormap
import matplotlib.pyplot as plt
plt.style.use("academic.mplstyle")
theme = apply_theme()  # reads DEFAULT_THEME
plt.plot(x, y, color=theme["primary"])
plt.imshow(data, cmap=get_colormap())
```

**LaTeX tables:**
- Ranking: `\fst{val}` / `\snd{val}` / `\trd{val}`
- Heatmap cells: `\cellcolor{perf30}` through `\cellcolor{perf80}`
- Emphasis: `\textcolor{accent_primary}{text}`

**Theme switch** (guard mode): update `\themename` in `colors.tex` + `DEFAULT_THEME` in `paper_palette.py` atomically.

---
> Source: [freemty/paper-style](https://github.com/freemty/paper-style) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
