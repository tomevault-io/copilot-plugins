## krolique

> This is a 3D dark fantasy game built with the **libGDX** framework and **Kotlin**. The game features a classic dungeon-crawling experience with a focus on atmosphere and exploration. The project is structured into a `core` module for the main game logic and a `desktop` module for the desktop launcher.

# Project: Dark Fantasy GDX

## Project Overview

This is a 3D dark fantasy game built with the **libGDX** framework and **Kotlin**. The game features a classic dungeon-crawling experience with a focus on atmosphere and exploration. The project is structured into a `core` module for the main game logic and a `desktop` module for the desktop launcher.

**Key Technologies:**

*   **Game Framework:** libGDX
*   **Language:** Kotlin
*   **Build Tool:** Gradle
*   **Platform:** Desktop (via LWJGL3)

**Architecture:**

The project follows a modular architecture with a clear separation of concerns. The core systems include:

*   `PlayerController`: Manages player input and actions.
*   `CreatureManager`: Handles AI and behavior of non-player characters.
*   `PhysicsSystem`: Manages physics simulation using Bullet.
*   `MagicSystem`: Handles magic and spell-casting.
*   `CombatSystem`: Manages combat mechanics.

**Improvement Plan:**

The project has a detailed improvement plan that includes:

*   **Technical Improvements:** Enhancing the graphics with shaders and dynamic lighting, improving the AI with behavior trees, and implementing ragdoll physics.
*   **New Features:** Adding a quest system, an inventory system, and a skill system.
*   **New Content:** Adding new levels, enemies, and items.

**Localization:**

The project has a plan to add localization support, starting with Russian. The localization system will be based on JSON files and a `LocalizationManager` class.

## Building and Running

The project is built and managed using Gradle.

*   **To run the game:**
    ```bash
    gradlew desktop:run
    ```
*   **To build the project:**
    ```bash
    gradlew build
    ```

## Development Conventions

*   **Language:** The project is written in Kotlin.
*   **Asset Management:** The `GameAssetManager` class is used to load and manage all game assets.
*   **Configuration:** Game controls are defined in `assets/config/controls.json`.
*   **Shaders:** The game uses custom shaders for rendering. The `ShaderManager` class is used to load and manage shaders.
*   **Quests:** The `QuestManager` class is used to manage quests.
*   **Screens:** The game uses a screen-based architecture. The `DarkFantasyGame` class manages the different screens of the game.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kezhick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kezhick)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
