## chemsearch

> This document provides context for the Gemini code assistant to understand the `chemsearch` project.

# Gemini Code Assistant Context

This document provides context for the Gemini code assistant to understand the `chemsearch` project.

## Project Overview

`chemsearch` is a Python-based bioinformatics tool designed to identify genomes with the potential to synthesize specific chemical compounds. It bridges the gap between chemical structures and genomic data by:

1.  **Preprocessing Biochemical Data:** It builds a database linking chemical compounds (represented by SMILES strings) to the enzymatic reactions (EC numbers) that produce them.
2.  **Querying by Chemical Structure:** It allows users to search for compounds in the database using SMILES strings or ChEBI IDs and a similarity threshold.
3.  **Identifying Promising Genomes:** It searches a collection of genomes for those that contain the identified EC numbers, providing a list of candidate organisms for synthesizing a compound of interest.

The project includes a core Python library, a command-line interface, and a web-based user interface.

## Technologies Used

*   **Backend:** Python, Flask, RDKit, Pandas, NumPy, H5Py
*   **Frontend:** HTML, CSS, JavaScript
*   **Data:** The project uses data from ModelSEED and Rhea.

## Building and Running

### Core Library

The core `chemsearch` library can be used as a Python package. The dependencies are listed in `requirements.txt` and can be installed with pip:

```bash
pip install -r requirements.txt
```

The primary workflow is demonstrated in the [example notebook](notebooks/example.ipynb).

### Web Application

The project includes a web-based user interface for interactive queries.

**1. Start the Backend Server:**

The backend is a Flask application. To run it, execute the following command from the project root:

```bash
python web-ui/backend/app.py
```

The server will start on `http://localhost:5001`.

**2. Launch the Frontend:**

Open the `web-ui/frontend/index.html` file in a web browser. The frontend will connect to the backend server to perform searches.

## Development Conventions

*   **Code Structure:** The Python source code is located in the `src/chemsearch` directory. The web application code is in the `web-ui` directory.
*   **Dependencies:** Python dependencies are managed in the `requirements.txt` file.
*   **Testing:** The `tests` directory contains tests for the project.
*   **Data:** The `data` directory contains the data used by the project.
*   **Notebooks:** The `notebooks` directory contains Jupyter notebooks with examples and analysis.
*   **Coding Style:** The code is written in Python with type hints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Robaina) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
