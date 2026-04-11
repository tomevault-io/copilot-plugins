## 14113-x-vault

> This document provides an overview of the `twc-vault`, a highly structured Personal Knowledge Management (PKM) system.

# GEMINI Context: The TWC Vault

This document provides an overview of the `twc-vault`, a highly structured Personal Knowledge Management (PKM) system.

## 🧠 Directory Overview

The `twc-vault` is a sophisticated knowledge vault organized using a customized **PARA (Projects, Areas, Resources, Archives)** methodology, augmented with an **Enneagram-based taxonomy**. It serves as a central repository for research, notes, and active projects.

The primary topics covered include:
- Consciousness Studies & Esoteric Philosophy
- Technology, particularly Bioelectronics and AI
- Health, Wellness, and Natural Medicine
- Personal Development and Skills Acquisition
- Creative Writing (the "Somatic Canticles" trilogy)

The vault is designed to be **text-first**, with large binary files (PDFs, images) excluded from Git tracking. It appears to be managed using Obsidian, given the `.obsidian` configuration directory.

## ⚙️ System Architecture

The vault's content management is not manual. It operates on a custom **Skills-based architecture** defined in the `_System/` directory and executed via Claude skills located in `.claude/skills/`.

- **Orchestration**: A set of 7 modular Claude skills handles the entire content processing pipeline, from file discovery and text extraction to classification, routing, and integration into the vault's Maps of Content (MOCs).
- **Taxonomy**: A controlled vocabulary based on an Enneagram + PARA model is the single source of truth for organization. See `_System/TAXONOMY-REFERENCE.md` for details.
- **Legacy System**: This skills-based system replaces a previous, deprecated Python-based orchestrator.

## 🚀 Key Projects & Components

This vault contains several key projects and components that are crucial to its function.

### 1. 10865xseed (Tryambakam Noesis)

This is a significant, self-contained Python project located in `_System/10865xseed/`.

- **Purpose**: It's described as a "consciousness architecture system" that bridges Eastern spiritual frameworks (like Pancha Kosha) with modern AI development. Its primary function is to gather context ("Prana") from the vault and inject it into AI development tools.
- **Key Command**: The main utility is `openclaw-seed`. A common use case is generating context for Claude:
  ```bash
  openclaw-seed prana --root . --platform claude
  ```
- **Installation**: The project has its own local installation script:
  ```bash
  cd _System/10865xseed && ./install-local.sh
  ```
- **Documentation**: For more details, refer to the extensive documentation within its directory, starting with `_System/10865xseed/README.md` and `_System/10865xseed/QUICKSTART.md`.

### 2. Somatic Canticles

This is a major creative project within the vault, located at `01-Projects/Somatic-Canticles/`.

- **Purpose**: It is a sci-fi trilogy manuscript. The folder contains the text-focused development and world-building documentation.

### 3. Knowledge Indexes (Maps of Content)

The vault is heavily organized around "Maps of Content" (MOCs) which act as indexes for various topics.

- **Example**: `Books-Master-Index.md` is a detailed, analytical index of a 2,000+ book personal library, classified using the vault's custom Enneagram+PARA taxonomy. It demonstrates the deep, structured approach to knowledge within the system.

## 📝 Development & Usage Conventions

- **Text-First**: The repository is intended for text-based files. Large binaries are not tracked in Git.
- **Content Processing**: Adding and organizing content should be done through the established Claude skills pipeline to ensure consistency with the vault's taxonomy.
- **Project-Specific Workflows**: When working on embedded projects like `10865xseed` or `Somatic-Canticles`, refer to their specific README files for instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sheshiyer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sheshiyer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
