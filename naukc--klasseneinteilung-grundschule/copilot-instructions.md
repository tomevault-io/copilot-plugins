## klasseneinteilung-grundschule

> The "Klasseneinteilung_Grundschule" project is a Python application designed to assist primary schools in the automated allocation of students into classes. It employs a Simulated Annealing algorithm to optimize class divisions based on various criteria, including gender balance, migration background, "Auffälligkeit" (a score indicating special needs or behavioral patterns), friendship wishes, and separation requests. The application offers both a command-line interface (CLI) and a graphical user interface (GUI) for user interaction.

# Project: Klasseneinteilung_Grundschule

## Project Overview
The "Klasseneinteilung_Grundschule" project is a Python application designed to assist primary schools in the automated allocation of students into classes. It employs a Simulated Annealing algorithm to optimize class divisions based on various criteria, including gender balance, migration background, "Auffälligkeit" (a score indicating special needs or behavioral patterns), friendship wishes, and separation requests. The application offers both a command-line interface (CLI) and a graphical user interface (GUI) for user interaction.

## Key Components

*   **`algorithmus.py`**: Contains the core logic for class allocation. This includes functions for generating random initial class divisions (`erstelle_zufaellige_einteilung`), evaluating the quality of a division based on predefined rules (`bewerte_einteilung`), and optimizing the class divisions using the Simulated Annealing metaheuristic (`optimiere_einteilung`).
*   **`config.py`**: Stores all configurable parameters for the optimization algorithm. This includes penalty points for imbalances (e.g., gender, migration background, "Auffälligkeit"), positive points for fulfilling friendship wishes, and parameters controlling the Simulated Annealing process (e.g., number of iterations, starting temperature, cooling rate).
*   **`datenlader.py`**: Responsible for loading student data. It reads student information from an Excel file (`.xlsx`) using `pandas`. The expected format includes a 'Schüler-ID' as the index and columns such as 'Name', 'Vorname', 'Geschlecht', 'Migrationshintergrund / 2. Staatsangehörigkeit', 'Auffaelligkeit_Score', 'Wunsch_X' (for friendship wishes), and 'Trennen_Von' (for separation requests).
*   **`gui.py`**: Implements the graphical user interface using the `tkinter` library. It allows users to easily select an Excel file containing student data, specify the desired number of classes, initiate the class allocation process, and save the optimized class divisions to a new Excel file.
*   **`main.py`**: Provides a command-line interface for the class allocation tool. It loads student data, performs the optimization, prints the results to the console, and automatically saves the final allocation to an Excel file.
*   **`requirements.txt`**: Lists all Python package dependencies required for the project, including `pandas`, `numpy`, `openpyxl`, and `tkintertable`.

## Building and Running

### Dependencies
The project requires the following Python packages, which can be installed using `pip`:

```bash
pip install -r requirements.txt
```

### Running the Application

#### Graphical User Interface (GUI)
To run the application with the graphical user interface:

```bash
python gui.py
```

#### Command-Line Interface (CLI)
To run the application from the command line:

```bash
python main.py
```

## Development Conventions

*   **Language**: The project is developed in Python.
*   **Localization**: Code comments, variable names, and function names are predominantly in German.
*   **Data Handling**: `pandas` DataFrames are extensively used for efficient data manipulation and analysis.

## Current Development Status (as of November 3, 2025)

This section summarizes the latest significant changes and the current state of the repository.

### Branch `main` Status
The `main` branch currently reflects all features and fixes developed in the `statistik` branch. It is the new stable baseline for the project.

### Key Features Implemented:
*   **Enhanced Statistics Display:** The GUI now displays comprehensive statistics for class allocation, including gender balance, migration background, "Auffälligkeit" sum, fulfilled wishes, and **unfulfilled wishes count per class**.
*   **Detailed Unfulfilled Wishes:** The GUI also iteratively lists individual unfulfilled wishes, providing details such as student name, ID, assigned class, wished student's name, ID, total wishes for the student, and total unfulfilled wishes for the student.
*   **Robust Excel Export:** The Excel export functionality has been significantly improved to handle statistics and unfulfilled wishes reliably.
    *   The main "Auswertung" sheet contains class statistics.
    *   A new sheet named "Wuensche" (previously "Nicht erfüllte Wünsche (Details)") is included, detailing individual unfulfilled wishes with clear, atomic columns for better Excel compatibility.
    *   Class lists are exported to separate sheets (e.g., "Klasse_A", "Klasse_B").
*   **Alphabetical Class Naming:** Classes are now named alphabetically (A, B, C, etc.) instead of numerically (1, 2, 3). This applies to both GUI display and Excel exports.
*   **Configurable Iterations:** The `OPT_ITERATIONEN` in `config.py` is set to `10000` for production use.
*   **Default Number of Classes:** The default `ANZAHL_KLASSEN` in `config.py` is set to `5`.
*   **Data Cleaning:** The `Geschlecht` column in `datenlader.py` is now standardized (converted to lowercase and stripped of whitespace) during data loading to ensure accurate gender statistics.
*   **Comprehensive Unit Tests:** New unit tests (`tests/test_utils.py`) have been added to verify the correctness of statistics calculations and utility functions. All existing and new tests are passing.

### Next Steps / Future Considerations:
*   Further refinement of the optimization algorithm.
*   User feedback on the new statistics and Excel export format.
*   Consider adding more configurable parameters or criteria for class allocation.

### Agent Mandates:
*   **Dynamic README.md Updates:** The `README.md` file should be dynamically updated to reflect new features or changes in functionality. This ensures that the project documentation remains current and accurate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naukc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/naukc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
