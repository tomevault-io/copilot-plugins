## darkwar-bot

> This project is a Go-based automation bot for the game "Dark War Survival". It uses GUI automation to perform repetitive tasks, helping the user to progress in the game by automatically helping allies and joining specific events.

# Project Overview

This project is a Go-based automation bot for the game "Dark War Survival". It uses GUI automation to perform repetitive tasks, helping the user to progress in the game by automatically helping allies and joining specific events.

**Main Technologies:**

*   **Go:** The core language for the bot's logic.
*   **robotgo:** A library for cross-platform GUI automation, used to control the mouse, simulate clicks, and capture the screen.
*   **gosseract:** A wrapper for the Tesseract OCR engine, used to read text from the screen (timers, button labels, event titles).
*   **imgo, resize:** Libraries for image processing, used to find icons, manipulate screenshots, and prepare images for OCR.

**Architecture:**

The bot follows a two-phase execution flow:

1.  **Preparation Phase (Sequential):**
    *   **Screen Verification:** detecting if the game is in the "Shelter" or "World" view and navigating to the "World" view if necessary.
    *   **Event Analysis:** Scans the list of available events to determine the number of pending meetings for specific types (e.g., "Zombi Gigante", "Caza con Victor").
    *   **Configuration Setup:** Navigates through game menus to disable the in-game "Automatic Meetings" feature to prevent conflicts with the bot's logic.

2.  **Execution Phase (Concurrent):**
    Once preparation is complete, the bot runs two main tasks concurrently as goroutines:
    *   **Helping Allies:** Repeatedly searches for an "ally help" icon on the screen and clicks it.
    *   **Joining Meetings:** Monitors the screen for a "meeting" icon. When found:
        *   It uses OCR to validate the meeting timer.
        *   Pauses the "Helping Allies" task to avoid mouse interference.
        *   Searches for and clicks the "Green Plus" button to join, iterating through available event cards.
        *   Validates and clicks the "Partir" (Depart) button, checking multiple potential screen positions and verifying if the button is enabled (green) or disabled (gray) before clicking.

# Building and Running

**Prerequisites:**

*   Go programming language.
*   Tesseract OCR engine installed and available in the system's PATH.

**Dependencies:**

The project uses the following Go modules:

*   `github.com/go-vgo/robotgo`
*   `github.com/nfnt/resize`
*   `github.com/otiai10/gosseract`
*   `github.com/vcaesar/imgo`

To install the dependencies, run:

```sh
go get github.com/go-vgo/robotgo
go get github.com/nfnt/resize
go get github.com/otiai10/gosseract
go get github.com/vcaesar/imgo
```

**Running the bot:**

To run the bot, execute the following command in the project's root directory:

```sh
go run main.go
```

The bot will start after a 5-second delay. To stop the bot, press the `ENTER` key in the terminal.

# Development Conventions

*   **Configuration:** The bot's behavior is extensively configured through constants at the beginning of `main.go`. This includes:
    *   **General:** Debug mode, file paths, image tolerances.
    *   **Search Areas:** specific `image.Rect` definitions for icons, OCR regions, and button locations.
    *   **UI Layout:** Constants defining the dimensions and spacing of event cards (`CARD_HEIGHT`, `CARD_SPACING`) for list iteration.
*   **Concurrency & Synchronization:**
    *   `sync.WaitGroup` is used to manage the lifetime of the main goroutines.
    *   A boolean channel (`pausarAyudaChan`) coordinates the tasks, allowing the high-priority "Meeting" task to pause the "Ally Help" task during complex click sequences.
*   **Image-based Recognition:**
    *   Uses `imgo` and `robotgo` to find icons (`ayuda_icono.png`, `reunion_icono.png`, `mas_verde_icono.png`) with configurable color and pixel tolerances (`TOLERANCIA_COLOR`, `TOLERANCIA_PIXEL`).
*   **Advanced OCR Strategy:**
    *   Text recognition is critical for reading timers and identifying buttons.
    *   **Preprocessing Pipeline:** Images captured for OCR undergo a rigorous pipeline to maximize accuracy: `Resize (Bicubic)` -> `Grayscale` -> `Threshold Binarization` -> `Inversion` -> `Padding`.
    *   **Button Validation:** The bot uses OCR to find the "Partir" text but also checks the pixel color at the button's origin to determine if it is enabled (Green) or disabled (Gray).
*   **Navigation Logic:**
    *   The bot assumes a specific screen resolution (referenced as `ANCHO_PANTALLA = 3840` in constants) and uses absolute coordinates for some menu navigation steps.
    *   It includes robust checks (like `irAlMundo`) to ensure the game state is correct before performing actions.
*   **Debugging:**
    *   `DEBUG_MODE = true` enables verbose console output and saves intermediate image processing steps (e.g., `identificacion_pantalla_ocr.png`, `boton_partir_intento_X.png`) to help diagnose recognition failures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ismafc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ismafc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
