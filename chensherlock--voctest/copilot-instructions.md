## voctest

> This project is a web-based vocabulary memorization application designed to help users learn and practice English vocabulary. The application is built entirely with vanilla HTML, CSS, and JavaScript, and it runs completely on the client-side.

# GEMINI.md

## Project Overview

This project is a web-based vocabulary memorization application designed to help users learn and practice English vocabulary. The application is built entirely with vanilla HTML, CSS, and JavaScript, and it runs completely on the client-side.

The core features of the application include:

*   **Unit-based Learning:** Vocabulary is organized into units, which can be loaded from JSON data files.
*   **Interactive Flashcards:** Users can practice vocabulary using interactive flashcards that flip to reveal the translation.
*   **Quizzes:** A quiz feature allows users to test their knowledge of the vocabulary.
*   **Local Progress Tracking:** The application uses the browser's local storage to track the user's progress and mastery of words.
*   **Audio Pronunciation:** The application can play audio pronunciations of words.

## Building and Running

This is a client-side application with no build process. To run the application, you can either:

1.  **Open `index.html` directly in your web browser.** The `run.cmd` script is provided as a shortcut for this.
2.  **Use the provided Python web server.** The `server.py` script starts a simple, no-cache HTTP server on port 8000. This is useful for development to ensure you always get the latest version of the files. You can start the server by running `python server.py` or by using the `server.cmd` script.

## Development Conventions

*   **File Structure:** The project follows a clear file structure, separating HTML, CSS, and JavaScript files into their respective directories.
*   **Data:** Vocabulary data is stored in JSON files in the `data/` directory. `units-index.json` serves as an index to the individual unit files.
*   **JavaScript:** The JavaScript code is modular, with different files responsible for different aspects of the application:
    *   `data.js`: Handles loading and caching of vocabulary data.
    *   `script.js`: Contains general application logic and initialization.
    *   `units.js`: Manages the display of vocabulary units and words.
    *   `flashcards.js`: Implements the flashcard functionality.
    *   `quiz.js`: Implements the quiz functionality.
*   **Styling:** The application uses a single CSS file, `css/styles.css`, for all its styling.
*   **Dependencies:** The project has no external dependencies other than Font Awesome for icons, which is loaded from a CDN.
*   **State Management:** The application's state, including user progress, is managed using the browser's local storage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chensherlock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
