## scryfall-ui

> This file provides context for the Gemini AI assistant to understand and effectively assist with this project.

# Gemini Project Context: ScryFall-UI

This file provides context for the Gemini AI assistant to understand and effectively assist with this project.

## Project Overview

This project is a Python-based desktop application with a graphical user interface (GUI). Its primary purpose is to interact with the Scryfall and Archidekt APIs. Users can likely search for Magic: The Gathering cards, view card details (including images), and manage or view decklists from Archidekt.

## Key Technologies

- **Language:** Python
- **APIs:**
    - **Scryfall:** For Magic: The Gathering card data and images. The core logic is likely in `core/scryfall_api.py`.
    - **Archidekt:** For decklist fetching and management. Test files like `test_archidekt.py` suggest its integration.
- **GUI Framework:** A Python GUI framework is used (e.g., Tkinter, PyQt, PySide). The specific framework can be identified by inspecting the imports in the `ui/` directory files.
- **Dependencies:** Managed via `Pipfile` (using `pipenv`) and a `requirements.txt` is also present.

## Project Structure

- `main.py`: The main entry point for the application.
- `core/`: Contains the core application logic, separated from the UI.
    - `scryfall_api.py`: Handles interactions with the Scryfall API.
    - `image_loader.py`: Manages downloading and caching card images.
- `ui/`: Contains all GUI-related code.
    - `main_window.py`: The main application window.
    - `detail_window.py`: A window to show specific card details.
- `utils/`: Contains helper functions and other utilities.
- `resources/cache/`: Caching directory, likely for images to reduce API calls.
- `test_*.py`: Python test files, suggesting the use of a framework like `pytest`.

## Development Workflow

- **Running the application:** The application is likely started by running `python main.py`.
- **Managing Dependencies:** To add or remove dependencies, `pipenv` should be used, which will update `Pipfile` and `Pipfile.lock`. `requirements.txt` may need to be regenerated afterwards (e.g., `pipenv lock -r > requirements.txt`).
- **Running Tests:** Based on the file naming convention, tests can likely be run using the `pytest` command from the project root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrFlou)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MrFlou)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
