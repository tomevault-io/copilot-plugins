## dialogos-a-esp

> **Dialogos a Español** is a Python-based tool designed for writers to convert manuscript dialogs from English-style formatting (using quotes) to Spanish-style formatting (using em dashes/rayas), strictly adhering to Real Academia Española (RAE) rules.

# Conversor de Diálogos a Español

## Project Overview

**Dialogos a Español** is a Python-based tool designed for writers to convert manuscript dialogs from English-style formatting (using quotes) to Spanish-style formatting (using em dashes/rayas), strictly adhering to Real Academia Española (RAE) rules.

The project offers two modes of operation:

1. **Native GUI:** Built with Tkinter (standard library) for a dependency-free experience.
2. **CLI:** A command-line interface for batch processing and automation.

### Key Features

- **Format Support:** Handles plain text (`.txt`) and OpenDocument Text (`.odt`), preserving ODT styles and metadata.
- **RAE Compliance:** Implements specific rules (D1-D5) for dialog tags, punctuation, and continuations.
- **Minimal Dependencies:** The project relies solely on the Python standard library.

## Architecture

- **Core Logic:** Located in `src/`.
  - `src/converter.py`: Main conversion logic (`DialogConverter`).
  - `src/rules.py`: Definitions of RAE dialog rules.
  - `src/odt_handler.py`: Specialized handling for `.odt` files (zipping/unzipping/XML parsing).
  - `src/main.py`: Entry point for the CLI.
- **GUI:** `gui.py` implements the Tkinter interface.
- **Examples:** `examples/` contains sample files for testing conversion.

## Building and Running

### Prerequisites

- **Python:** Version 3.11 or higher is required.
- **Dependencies:** None.

### Running the Application

**1. Native GUI (Recommended)**

```bash
./start_gui.sh
# OR
python gui.py
```

**2. Command Line (CLI)**

```bash
# Process a single file
python -m src.main path/to/file.txt

# Process a directory (recursive)
python -m src.main path/to/folder/ --recursive --filter "*.odt"
```

### Building the Executable

The project includes a script to bundle the application into a standalone Linux executable using `PyInstaller`.

```bash
./build_executable.sh
```

- **Output:** `dist/Conversor-Dialogos`
- **Requirements:** `pyinstaller` (script will ask to install if missing).

## Development Conventions

- **Code Style:** Follows standard Python conventions (PEP 8).
- **Dependencies:** Avoid adding external dependencies unless absolutely necessary. The core value proposition is being "dependency-free" (stdlib only).
- **Testing:** Currently, there are no automated unit tests (folder `tests/` is empty). Testing is done manually using the files in `examples/` and verifying the output logs.
- **ODT Handling:** When working with `.odt` files, specialized care is taken to modify `content.xml` while preserving the surrounding ZIP structure and styles.
- **Logging:** The system generates detailed `.log.txt` and `.log.json` files for every conversion to assist in debugging and reviewing changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/T4toh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
