## my-strudel

> This project is a minimal Strudel song manager and educational platform built with [Eleventy (11ty)](https://www.11ty.dev/). it allows users to organize, document, and play Strudel and TidalCycles live-coding music patterns directly in the browser.

# Project Overview: My Strudel

This project is a minimal Strudel song manager and educational platform built with [Eleventy (11ty)](https://www.11ty.dev/). it allows users to organize, document, and play Strudel and TidalCycles live-coding music patterns directly in the browser.

## Key Technologies
- **Eleventy (11ty)**: Static site generator.
- **Strudel REPL**: Interactive editor and audio engine for live coding music.
- **ABCJS**: Used for rendering music notation from ABC code blocks in Markdown.
- **Custom Metadata Parser**: Extracts song information (title, author, tags) from comments within `.strudel` or `.tidal` files.

## Project Structure
- `src/songs/`: The primary directory for song files.
  - Individual `.strudel` or `.tidal` files become standalone pages.
  - Directories containing an `index.md` and `song.strudel` create "lesson" pages where the Markdown provides instructional content and the Strudel file provides the interactive playground.
- `src/_includes/layout.njk`: The main template for song pages, featuring a full-page REPL, a virtual piano, and a sound reference sidebar.
- `eleventy.config.js`: Configures custom handling for `.strudel` and `.tidal` extensions and manages static asset passthrough.
- `metadata-parser.js`: Logic for parsing comments and `@tags` from Strudel code.

## Building and Running

### Development
To start the local development server with hot reloading:
```bash
npm start
```
The site will typically be available at `http://localhost:8080`.

### Production Build
To generate the static site in the `_site` directory:
```bash
npm run build
```

## Development Conventions

### Adding Songs
- Drop a `.strudel` or `.tidal` file into `src/songs/`.
- To add metadata, use comments at the top of the file:
  ```javascript
  /* "My Awesome Song" */
  // @author Alice
  // @genre ambient
  
  stack(
    s("bd sd").every(2, rev),
    s("hh*4")
  )
  ```

### Creating Lessons
For more complex pages with documentation:
1. Create a folder in `src/songs/`.
2. Add an `index.md` for the text content.
3. Add a `song.strudel` for the interactive code.
4. The site will automatically combine them, using the Markdown for the documentation area and the Strudel file for the editor.

### Music Notation
You can include sheet music in Markdown using ABC notation:
\```abc
T: Example
M: 4/4
K: C
C D E F | G A B c |
\```
The site will automatically render this into an SVG notation block.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InTEGr8or)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/InTEGr8or)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
