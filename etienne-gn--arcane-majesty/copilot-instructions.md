## arcane-majesty

> > **Instructions for AI Sessions:**

# Arcane Majesty Compendium

> [!IMPORTANT]
> **Instructions for AI Sessions:**
> 1.  **Contextualize:** At the start of a new session, you MUST read the following files to align with our established lore and progress:
>     - `GEMINI.md` (Project Overview & Tech Stack)
>     - `current_progress.md` (Current Status & Next Steps)
>     - `world_lore.md` (Cosmic Structure & Mechanics)
>     - `data/lore/xmls/project_summary.md` (Linear Rework Workflow)
> 2.  **Update Records:** As milestones are achieved, you are responsible for updating `current_progress.md` and this section of `GEMINI.md` to ensure continuity for future sessions.

## Project Overview

This project, "Arcane Majesty," is a multi-dimensional fantasy saga that combines music, lore, and game development to create a rich, immersive universe. The project is presented as a web-based compendium that serves as a central hub for all its components.

The core of the project is a fantasy world with a detailed narrative, characters, and realms. This lore is expressed through:

*   **Music Albums:** The project includes multiple music albums, with songs that tell the story of the "Arcane Majesty" universe. The albums are divided into acts and songs, each with its own plot, tone, and lyrics.
*   **Game Development:** The project is being developed as a web-based Action RPG using Phaser 3 and Vite.
*   **Web Compendium:** The project's main entry point is a web-based compendium that presents all the lore, music, and game development information in a structured and interactive way.

The project is built using the following technologies:

*   **Frontend:** HTML, CSS, JavaScript, Tailwind CSS, and Lucide icons.
*   **Game Engine:** Phaser 3 for the game component.
*   **Data:** XML is used for historical lore storage; Neo4j is used for the master knowledge graph.

## Directory Overview

The project directory is structured as follows:

*   `index.html`: The main entry portal for the project.
*   `world_lore.md`: Defines the fundamental cosmic structure and parallel dimensions.
*   `rework_approach.md`: Documents the narrative songwriting strategy for Suno integration.
*   `neo4j_architecture.md`: Defines the graph schema, node labels, and relationship types.
*   `db_connection_requirements.md`: Comprehensive guide for production Neo4j connection and Gemini CLI integration.
*   `apps/`: Contains the modular applications (Compendium and Game).
*   `data/`: Centralized data store for lore (XML), Neo4j configuration, and utility scripts.
    *   `data/lore/`: Contains the primary XML databases and refined markdown lore.
    *   `data/neo4j/`: Neo4j setup and local development documentation.
*   `assets/`: Multimedia assets (Art, Music, Video).

## Development Conventions

The project follows a set of development conventions that are evident from the codebase:

*   **Graph-First Lore:** The project is migrating from static XML files to a dynamic Neo4j knowledge graph.
*   **Narrative Songwriting:** Music is reworked using the [Rework Approach](./rework_approach.md) to ensure lyrics are narrative-driven and story-specific.
*   **Linear Era Focus:** Lore is developed and imported into the graph era-by-era (starting with 0 GD) to ensure consistency.
*   **Utility-First CSS:** The project uses Tailwind CSS for styling the web components.

## Usage

To view the project, open the root `index.html` file in a web browser. Developers and lore-masters should refer to the markdown documentation in the root and `data/` directories for architectural and connection details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Etienne-GN)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Etienne-GN)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
