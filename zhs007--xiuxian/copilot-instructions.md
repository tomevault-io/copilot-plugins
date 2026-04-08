## xiuxian

> This document provides instructions for AI agents working on this codebase.

# Agents.md

This document provides instructions for AI agents working on this codebase.

## Project Setup

This is a `pnpm` monorepo. To get started, run `pnpm install` in the root directory to install all dependencies for all workspaces.

## Key Technologies

- **Rendering:** Pixi.js v8
- **Build Tool:** Vite
- **Testing:** Vitest
- **Server:** Fastify
- **Linting/Formatting:** ESLint v9, Prettier

## Development Workflow

The project uses `pnpm` as a package manager and `Turborepo` to manage the monorepo scripts. All commands should be run from the project root.

1.  **Code Style:** All code must be formatted with Prettier and pass ESLint checks. Use `pnpm format` and `pnpm lint` before committing.
2.  **Logic and Rendering Separation:** A strict separation between logic and rendering is enforced.
    - Game logic (state, rules) goes into `packages/logic-core` or `apps/game/src/game/logic`. These files should be plain TypeScript and not import any Pixi.js modules.
    - Rendering code (display objects, scenes) goes into `apps/game/src/scenes`. These files are responsible for visualizing the state from the logic layer.
3.  **Testing:**
    - All new logic must be accompanied by unit tests.
    - Run tests with `pnpm test`.
    - Aim for a high test coverage (>90%) for the logic parts. Check coverage with `pnpm test` (it is configured to generate coverage).
4.  **Commits:** Follow conventional commit standards for your commit messages.

## Running the Application

- **Development:**
  - Run `pnpm dev`. This will use Turborepo to start the development servers for both the `game` client and the `server` backend concurrently.
  - Open the client URL shown in the terminal (usually `http://localhost:8080`).

- **Production:**
  - Run `pnpm build` to create production-ready builds for all applications.
  - To start the production server, you would typically run `pnpm start --filter=@xiuxian/server`.

## Asset Handling

- Static assets (images, fonts) are located in the `assets` directory.
- To use an asset in the code, import it directly. Vite will handle the URL resolution.
  - Example: `import myImage from '../assets/images/my-image.png';`
  - This `myImage` variable can then be used with a loader like `PIXI.Assets.load()`.

## Architectural Patterns

### Scene Management

The project uses a singleton `SceneManager` (`src/scenes/SceneManager.ts`) to control the game's scenes.

- **Usage:** To switch scenes, get the instance and call `SceneManager.getInstance().switchScene(NewScene)`.
- **Lifecycle:** The manager automatically handles destroying the previous scene (calling `.destroy({ children: true })`) to prevent memory leaks. Do not manually add or remove scenes from the stage.

### Logic and View Separation

A strict separation between logic and view is enforced, particularly for complex scenes.

- **Example (`MainScene`):** The complex drag/swipe calculations are not performed in `MainScene.ts`. Instead, they are located in pure functions in `src/game/logic/mainScreen.ts`.
- **Workflow:** The view (`MainScene`) captures user input, passes it to the pure logic functions, and then applies the returned values to the Pixi.js objects.
- **Testing:** All new gameplay logic should be in a `logic` file and be covered by unit tests.

### Card Data Structure

The core `logic-core` package defines how cards work.

- **`CardData` (`packages/logic-core/src/types.ts`):** This is the raw data definition for a card, loaded from JSON. It includes properties like `id`, `name`, `type`, and the new `rarity`. It is a discriminated union, making it type-safe.
- **`Card` (`packages/logic-core/src/card.ts`):** This is a lightweight class that represents a card instance. It **does not** duplicate data. Instead, it holds a reference to a `CardData` object and uses getters to expose the data. When you are working with a card in the game, you will be using a `Card` object.

## Agent Instructions Verification

After making changes, please verify them by running the following commands from the project root:

1.  `pnpm lint`
2.  `pnpm format:check`
3.  `pnpm test:ci` (Note: use the `test:ci` script for non-interactive testing)
4.  `pnpm build`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhs007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zhs007)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
