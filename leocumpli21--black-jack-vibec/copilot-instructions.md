## black-jack-vibec

> This document provides context for the Gemini code assistant to understand the Black Jack game project.

# Gemini Code Assistant Context

This document provides context for the Gemini code assistant to understand the Black Jack game project.

## Project Overview

This project is a web-based Blackjack game built with Python and **Streamlit**. The application is designed following the principles of **Clean Architecture**, which effectively separates the core game logic from the user interface, making the codebase modular, testable, and easy to maintain.

The project is divided into four main layers:

*   **Domain:** Contains the core game entities like `Card`, `Deck`, `Hand`, `Player`, and `Game`.
*   **Application:** Orchestrates the game flow with use cases like `StartRound`, `PlayerAction`, and `DetermineOutcome`.
*   **Interface Adapters:** Connects the application layer to the UI through controllers and presenters.
*   **Infrastructure:** Contains the user interface implementations. This now includes both the original CLI and the new Streamlit web UI.

## Building and Running

### 1. Dependencies

This project now has external dependencies which are listed in the `requirements.txt` file. To install them, run the following command from the project's root directory:

```bash
pip install -r requirements.txt
```

### 2. Running the Game (Streamlit UI)

The primary way to run the game is through the web interface.

To run the Streamlit application, execute the following command from the project's root directory:

```bash
streamlit run main_streamlit.py
```

### 3. Running Tests

The project uses `pytest` for unit testing. The tests are located in the `tests/` directory and mirror the structure of the `src/` directory.

To run the tests, execute the following command in the root directory:

```bash
pytest
```

## Development Conventions

*   **Architecture:** Adhere to the Clean Architecture principles. Core game logic should remain independent of the UI.
*   **Streamlit UI:**
    *   The UI is a **multipage application** managed by `main_streamlit.py`, which acts as the main router.
    *   Individual pages are located in `src/infrastructure/ui/streamlit/pages/`.
    *   **State Management:** The game's state is maintained across user interactions and pages using `st.session_state`. This is a critical concept for the UI's functionality.
    *   **UI Components:** Reusable UI functions, such as the logic to display a card, are kept in `src/infrastructure/ui/streamlit/ui_components.py`.
*   **Testing:** All new business logic should be accompanied by unit tests using the `pytest` framework.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeoCumpli21)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LeoCumpli21)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
