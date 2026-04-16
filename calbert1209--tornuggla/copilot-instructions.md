## tornuggla

> This project, "tornuggla," is a web-based flashcard-style learning application. It contains two distinct mini-games:

# Project: tornuggla

## Project Overview

This project, "tornuggla," is a web-based flashcard-style learning application. It contains two distinct mini-games:

1.  **Math Quiz (`index.html`)**: This game presents simple arithmetic problems (addition or subtraction) with two numbers. The user decides if they have answered correctly and presses "OK" or "NG" (No Good). Progress is tracked with images of Pikachu.
2.  **Hiragana Flip (`flip.html`)**: This game helps with learning Japanese hiragana characters. It displays two identical characters, one of which is flipped horizontally. The user's goal is to recognize the character. Progress is tracked with images of pineapples.

The application uses a modular structure with a core state management system (`modules/state.js` and `modules/retries.js`) that handles the game logic, including advancing to the next item, retrying missed items, and resetting the game.

The user interface is built with plain HTML, CSS, and vanilla JavaScript.

## Building and Running

This is a simple web project that does not require a complex build process. It is served using `lite-server`.

**To run the application:**

1.  Install the dependencies:
    ```bash
    npm install
    ```
2.  Start the development server:
    ```bash
    npm run dev
    ```
    This will open the application in your default web browser. By default, it runs on port 8342.

    *   To play the Math Quiz, navigate to `index.html`.
    *   To play the Hiragana Flip game, navigate to `flip.html`.

**Available Scripts:**

*   `npm run dev`: Starts the `lite-server` for development.
*   `npm test`: Currently not implemented.

## Development Conventions

*   **Code Style**: The code is written in vanilla JavaScript using ES Modules. It follows a clean and readable style with a focus on separating concerns (e.g., state management in `modules/`, UI interaction in the main `.js` files).
*   **State Management**: The application uses a custom `State` class to manage the game's state in a structured way. This class handles the game loop, user progress, and retries.
*   **Testing**: There are currently no automated tests configured for this project.

## Project Goal

For a detailed overview of the project architecture and the upcoming work, please refer to the `docs/Overview.md` file. The goal is to refactor the `index.html` game using TypeScript, Preact, Vite, and Vitest.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calbert1209) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
