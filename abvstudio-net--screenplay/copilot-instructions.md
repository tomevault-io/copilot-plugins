## screenplay

> This repository is a template for creating AI-generated comics by adapting an original story. It uses a custom Fountain-flavored markdown format with wiki-style links to connect screenplays in `./story/` with visual prompts (characters, locations, etc.) in `./prompts/`.

# Comic Generation Project

This repository is a template for creating AI-generated comics by adapting an original story. It uses a custom Fountain-flavored markdown format with wiki-style links to connect screenplays in `./story/` with visual prompts (characters, locations, etc.) in `./prompts/`.

# Workflow — Screenplay / Comic Pipeline

We outline the end-to-end process for turning source material into AI-generated comics using the repo conventions.

## High-Level Stages

1. **Ingest Original Story**
   - Place raw/clean source in `story/origional/*.md` (plain text or lightly formatted).
   - Keep filenames stable (e.g., `0-prologue.md`).

2. **Extract Reusable Prompts**
   - Pull characters, locations, styles, etc. into `prompts/{characters,locations,styles}/*.md`.
   - Follow Core Rules below for frontmatter, naming, and links.

3. **Adapt into Screenplay**
   - Create Fountain-flavored versions in `story/adapted/*.md`.
   - Follow Core Rules below for frontmatter and links.
   - Maintain scene headings, action, dialogue, etc.

4. **Generate Reference Images (Iterative)**
   - Use the descriptions in `prompts/*.md` as base prompts for your image model (Imagen, Midjourney, Flux, etc.).
   - Iterate until characters/locations look consistent (face, colors, style, lighting).
   - Store final reference images alongside their prompts (see Core Rules).
   - This stage can take many prompt tweaks—keep the `.md` files as the single source of truth.

5. **Build Storyboard**
   - Create `storyboard/*.md` files (one per key scene or panel).
   - Each storyboard file is a focused image generation prompt that **combines**:
     - Character refs (`[[prompts/characters/james.md]]`)
     - Location refs (`[[prompts/locations/barn-interior.md]]`)
     - Style (`[[prompts/styles/equestria-online.md]]`)
     - Specific action, camera angle, lighting, composition from the adapted screenplay.
   - Example structure:
     ```
     ---
     scene: "INT. BARN - Honeycrisp greeting"
     panel: 1
     ---
     [[prompts/characters/james.md]] [[prompts/locations/barn-interior.md]] ...
     Detailed action description + camera notes.
     ```
   - These become the direct prompts for comic panels.

6. **Generate & Assemble**
   - Feed storyboard prompts → image model.
   - Place outputs in `images/storyboard/` or similar.
   - (Future) Add comic layout scripts as needed.


# Core Rules & Naming Conventions
- All `story/*.md` and `prompts/*.md` files: Fountain format + `[[wiki-style links]]` + YAML frontmatter at top.
- Directories: `./story/` (original + adapted) | `./prompts/{characters,locations,styles}/` (reusable descriptions).
- Filenames: kebab-case | Keys in YAML: Title Case.
- Reference images stored alongside their `.md` prompts (e.g. `prompts/characters/james/james-ref.png`).
- Update this file when the workflow evolves.

---
> Source: [abvstudio-net/screenplay](https://github.com/abvstudio-net/screenplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
