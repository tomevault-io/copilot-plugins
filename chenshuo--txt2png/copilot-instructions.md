## txt2png

> | `linebreak.h` | C++ Knuth-Plass algorithm (header-only) |

# Agent Guidelines

## Repository layout

| Path | Contents |
|------|----------|
| `linebreak.h` | C++ Knuth-Plass algorithm (header-only) |
| `linebreak.cpp` | C++ CLI demo of the algorithm (monospaced output) |
| `linebreak.py` | Python port of the same algorithm |
| `textrender.cpp` | C++ CLI renderer (HarfBuzz + Cairo + ICU + libhyphen) |
| `textrender.py` | Python CLI renderer (uharfbuzz + pycairo + freetype-py + pyphen) |
| `sample_cjk.txt` | Sample input with CJK, English, and mixed paragraphs |
| `Makefile` | Builds `linebreak` and `textrender` |

## Build

```bash
sudo apt-get install -y \
    build-essential libcairo2-dev libfreetype-dev \
    libharfbuzz-dev libicu-dev libhyphen-dev \
    fonts-noto-cjk hyphen-en-us

make        # builds linebreak and textrender
```

## Python dependencies

```bash
pip install uharfbuzz pycairo freetype-py pyphen
```

## Running

```bash
# C++
./textrender --size 14 --width 800 --height 1000 sample_cjk.txt out.png

# Python
python3 textrender.py --size 14 --width 800 --height 1000 sample_cjk.txt out.png

# Disable hyphenation (on by default when hyph_en_US.dic is found)
./textrender --nohyphen --size 14 --width 800 --height 1000 sample_cjk.txt out.png
```

## Branch

Active development branch: `claude/modest-wright-ioer1p`

## Notes for agents

- Do not commit compiled binaries (`linebreak`, `textrender`).
- The Python and C++ renderers produce visually equivalent output but are not pixel-perfect due to different advance hinting (HarfBuzz unhinted vs FreeType hinted) and different hyphenation backends (pyphen vs libhyphen).
- `linebreak.py` and `linebreak.h` implement the same Knuth-Plass algorithm; keep them in sync when making algorithmic changes.

---
> Source: [chenshuo/txt2png](https://github.com/chenshuo/txt2png) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
