## texicode

> TeXicode (TeX to Unicode) is a tool that renders TeX math expressions into Unicode art. It is designed for use in environments that support code blocks but not native LaTeX (like Reddit, Discord, or terminal previews).

# TeXicode Project Overview

TeXicode (TeX to Unicode) is a tool that renders TeX math expressions into Unicode art. It is designed for use in environments that support code blocks but not native LaTeX (like Reddit, Discord, or terminal previews).

## Architecture

The project is split into a Python core and a web-based frontend that runs the Python core via Pyodide.

### Core Logic (`src/`)
- **`lexer.py`**: Tokenizes raw TeX strings into typed tokens (alphabetic, numeric, symbols, commands).
- **`parser.py`**: Processes tokens into a hierarchical Abstract Syntax Tree (AST) based on rules defined in `node_data.py`.
- **`renderer.py`**: Traverses the AST and converts it into a grid of Unicode characters (rows of strings).
- **`pipeline.py`**: Orchestrates the lexing, parsing, and rendering phases. It provides high-level functions like `render_tex` and `render_tex_web`.
- **`arts.py` & `symbols_art.py`**: Databases of Unicode glyphs and mappings for different fonts (serif vs. normal) and mathematical symbols.
- **`main.py`**: The CLI entry point, handling argument parsing and file processing (including markdown math extraction).

### Webapp
- **`index.html`**: The main page for the web interface.
- **`main.js`**: Initializes Pyodide, fetches the Python source files, and sets up a reactive rendering loop in the browser.
- **`style.css`**: Provides the UI styling for the webapp.
- **`pyodide/`**: Contains the local Pyodide runtime files.

## Building and Running

### CLI Usage
The CLI can be run directly via the Python script or using the `txc` wrapper.

**Basic Command:**
```bash
python3 src/main.py '\frac{a}{b}'
```

**Using the wrapper:**
```bash
./txc '\sum_{i=0}^n i'
```

**Markdown Processing:**
```bash
./txc -f document.md
```

### Installation
The project can be installed using `setup.py`:
```bash
pip install .
```
This installs the `txc` command-line tool.

### Webapp Development
To run the webapp locally, serve the project root using a simple HTTP server:
```bash
python3 -m http.server 8000
```
Then navigate to `http://localhost:8000`.

## Development Conventions

- **Unicode Focus**: The project avoids legacy ASCII art in favor of modern Unicode glyphs (italics, box-drawing characters, etc.).
- **Modular Pipeline**: Changes to TeX command support usually involve:
    1. Adding a new node type or mapping in `node_data.py`.
    2. Implementing the rendering logic for that node in `renderer.py`.
    3. Adding necessary glyphs to `arts.py` or `symbols_art.py`.
- **Testing**: Testing in this repository is data-driven. A visual regression script is available at `run_tests.py`, and an automated assertion suite is available via `pytest tests/`. Both identically read from the central data arrays (like `CORE_TESTS` and `ALIGNMENT_TESTS`) defined in `tests/test_data.py`. As you implement future features (like new environments or symbols), you can simply open up `tests/test_data.py` and add the new LaTeX strings and their expected visual grids to those arrays. This immediately registers them as both an automated regression test and a visual terminal demo! Additionally, `example.md` serves as a visual regression test for markdown rendering.

## Key Files
- `src/main.py`: CLI entry point.
- `src/pipeline.py`: Main API orchestration.
- `src/node_data.py`: Parser/Renderer configuration.
- `main.js`: Webapp glue code.
- `txc`: Bash wrapper for the CLI.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inpalpra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/inpalpra)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
