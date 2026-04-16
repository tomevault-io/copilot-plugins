## pokemon-remix

> This document provides a brief analysis of the project to be used by the Gemini agent.


# Gemini Project Analysis

This document provides a brief analysis of the project to be used by the Gemini agent.

## Project Overview

This is a Pokémon-style game prototype built with Remix, TypeScript, and p5.js. The core game logic resides in JavaScript files, while the Remix framework serves the game canvas and assets.

## Tech Stack

- **Framework:** Remix (React)
- **Language:** TypeScript, JavaScript
- **Styling:** Tailwind CSS
- **Game Engine:** p5.js
- **Package Manager:** npm

## Project Structure

- `app/`: Contains the Remix application code.
  - `root.tsx`: The main application component, which loads the p5.js game.
  - `routes/`: Defines the application's routes.
- `entities/`: Contains game entities like the player, NPCs, and camera.
- `scenes/`: Contains the different game scenes (menu, world, battle).
- `assets/`: Contains game assets like images and fonts.
- `maps/world.json`: This file (specifically `public/maps/world.json` which is served) contains the map layout and crucial spawn point data for players and NPCs.
- `main.js`: The main entry point for the p5.js game, handling the game loop and scene management.
- `package.json`: Defines project dependencies and scripts.

## Development Scripts

- `npm run dev`: Starts the development server.
- `npm run build`: Builds the application for production.
- `npm run lint`: Lints the codebase.
- `npm run typecheck`: Runs the TypeScript compiler to check for type errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shinguakira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
