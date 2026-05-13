## parser

> This project is a Python application designed to parse structured text data, extract specific fields like user agents and cookies, and export the results into an Excel file. The application is built with a modular architecture that separates concerns into readers, parsers, and exporters. It offers both a command-line interface (CLI) for automated processing and a graphical user interface (GUI) for interactive use.

# Gemini Project: Parser

## Project Overview

This project is a Python application designed to parse structured text data, extract specific fields like user agents and cookies, and export the results into an Excel file. The application is built with a modular architecture that separates concerns into readers, parsers, and exporters. It offers both a command-line interface (CLI) for automated processing and a graphical user interface (GUI) for interactive use.

**Key Technologies:**

*   **Language:** Python
*   **UI:** Tkinter (for the GUI)
*   **CLI:** `argparse`

**Architecture:**

The application follows a pipeline architecture orchestrated by `parser_app.services.pipeline.ProcessingPipeline`. The process is as follows:

1.  **Reader:** Reads input data from a file, standard input, or the GUI. (`parser_app/readers`)
2.  **Parser:** Parses the raw text data into structured `ProfileRecord` objects based on the configured separator and field indices. (`parser_app/parsers`)
3.  **Exporter:** Exports the `ProfileRecord` objects to an Excel file. (`parser_app/exporters`)

Configuration is managed through the `AppConfig` class in `parser_app/config.py`.

## Building and Running

### Running the GUI

To run the graphical user interface, execute the `main.py` script:

```bash
python main.py
```

### Running the CLI

The command-line interface provides more flexibility for automation.

**Synopsis:**

```bash
python -m parser_app.cli [OPTIONS]
```

**Options:**

*   `-i, --input <PATH>`: Path to the input text file. If omitted, the application will use standard input or fallback to sample data.
*   `-o, --output-dir <PATH>`: Directory where the generated Excel file will be stored.
*   `--separator <SEPARATOR>`: Field separator used in the input data (default is `|`).
*   `--ua-index <INDEX>`: 1-based index of the field containing the user agent (default is `3`).
*   `--cookie-index <INDEX>`: 1-based index of the field containing cookies (default is `5`).

**Examples:**

1.  **Parse a file and save to the default `results` directory:**

    ```bash
    python -m parser_app.cli --input my_input.txt
    ```

2.  **Parse data from standard input:**

    ```bash
    cat my_input.txt | python -m parser_app.cli
    ```

3.  **Parse a file with a custom separator and output directory:**

    ```bash
    python -m parser_app.cli --input data/samples.txt --output-dir /tmp/output --separator ";"
    ```

## Development Conventions

*   **Modularity:** The application is divided into distinct modules for reading, parsing, and exporting. New functionality should follow this pattern by extending the base classes in each module.
*   **Configuration:** Application settings are centralized in the `AppConfig` class.
*   **Testing:** Tests are located in the `tests/` directory. New tests should be added with the `test_` prefix.
*   **CLI:** The CLI should remain a thin layer for argument parsing and pipeline invocation.

---
> Source: [Slots924/Parser](https://github.com/Slots924/Parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
