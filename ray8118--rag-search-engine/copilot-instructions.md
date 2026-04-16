## rag-search-engine

> This project implements a command-line interface (CLI) for a keyword search engine. Its primary purpose is to provide functionalities for building an inverted index, performing keyword searches, and calculating various term-frequency related metrics such as Term Frequency (TF), Inverse Document Frequency (IDF), TF-IDF, and BM25 IDF. The search is likely performed on a dataset of movies, using movie titles and descriptions.

# Project: `rag-search-engine`

## Project Overview

This project implements a command-line interface (CLI) for a keyword search engine. Its primary purpose is to provide functionalities for building an inverted index, performing keyword searches, and calculating various term-frequency related metrics such as Term Frequency (TF), Inverse Document Frequency (IDF), TF-IDF, and BM25 IDF. The search is likely performed on a dataset of movies, using movie titles and descriptions.

**Key Technologies:**
*   **Python:** The core programming language.
*   **NLTK (Natural Language Toolkit):** Used for natural language processing tasks, specifically Porter Stemming for tokenization.
*   **Argparse:** For parsing command-line arguments and defining the CLI structure.
*   **Pickle:** Utilized for caching the inverted index, document map, and term frequencies to improve performance.

**Architecture:**
The project follows a simple architecture with a CLI frontend (`cli/keyword_search_cli.py`) that interacts with a backend library (`cli/lib/keyword_search.py`). The backend handles the core logic of building and querying the inverted index, performing text processing (tokenization, stemming, stopword removal), and computing search-related metrics. Data and cached indexes are managed in `data/` and `cache/` directories respectively.

## Building and Running

### Prerequisites

*   Python 3.12 or higher.
*   `nltk` library.

### Installation

1.  **Install NLTK data:**
    ```bash
    python -m pip install nltk
    python -c "import nltk; nltk.download('stopwords')"
    ```

2.  **Install project dependencies:**
    ```bash
    pip install -e .
    ```
    (This assumes `pyproject.toml` is set up for editable installs, or equivalent.)

### Commands

The CLI tool `keyword_search_cli.py` supports several subcommands:

1.  **Build the Inverted Index:**
    This command processes the movie data and builds the inverted index, document map, and term frequencies, saving them to the `cache/` directory.
    ```bash
    python cli/keyword_search_cli.py build
    ```

2.  **Search Movies:**
    Searches the indexed movies for a given query.
    ```bash
    python cli/keyword_search_cli.py search "your search query"
    ```

3.  **Calculate Term Frequency (TF):
    Calculates the term frequency for a specific term within a given document ID.
    ```bash
    python cli/keyword_search_cli.py tf <doc_id> <term>
    ```

4.  **Calculate Inverse Document Frequency (IDF):**
    Calculates the IDF for a specific term across all documents.
    ```bash
    python cli/keyword_search_cli.py idf <term>
    ```

5.  **Calculate TF-IDF:**
    Calculates the TF-IDF score for a specific term within a given document ID.
    ```bash
    python cli/keyword_search_cli.py tfidf <doc_id> <term>
    ```

6.  **Calculate BM25 IDF:**
    Calculates the BM25 IDF score for a specific term.
    ```bash
    python cli/keyword_search_cli.py bm25idf <term>
    ```

## Development Conventions

*   **Language:** Python 3.12+
*   **Text Processing:** Utilizes NLTK's Porter Stemmer for text normalization during indexing and searching. Stopwords are also removed.
*   **CLI Argument Handling:** `argparse` is used for consistent and robust command-line interface design.
*   **Caching:** Inverted index data, document mappings, and term frequencies are serialized using `pickle` and stored in the `cache/` directory to avoid re-computation.
*   **Data Storage:** Raw data (e.g., movie data) is expected to reside in the `data/` directory.

## Commit Message Conventions

Commit messages should follow the Conventional Commits specification. This style provides a structured commit history, making it easier to track features, fixes, and other changes.

Each commit message should be structured as follows:

```
<type>: <description>

[optional body]

[optional footer(s)]
```

### Type

The type must be one of the following:

*   **feat**: A new feature
*   **fix**: A bug fix
*   **docs**: Documentation only changes
*   **style**: Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
*   **refactor**: A code change that neither fixes a bug nor adds a feature
*   **perf**: A code change that improves performance
*   **test**: Adding missing tests or correcting existing tests
*   **build**: Changes that affect the build system or external dependencies (example scopes: pip, docker, npm)
*   **ci**: Changes to our CI configuration files and scripts (example scopes: Circle, BrowserStack, SauceLabs)
*   **chore**: Other changes that don't modify src or test files
*   **revert**: Reverts a previous commit

### Description

A concise, imperative-mood description of the change, typically 50-72 characters.

### Body (Optional)

A more detailed explanatory text, wrapped to about 72 characters. It should explain the *why* and *how* of the change.

### Example

```
feat: Implement BM25 IDF calculation

This commit introduces the BM25 Inverse Document Frequency (IDF) calculation.

Key changes include:
- Added `bm25idf` to `lib/keyword_search.py` for calculating BM25 IDF.
- Implemented `getbm25_idf` method in `InvertedIndex` class within `lib/keyword_search.py`.
- Added `bm25_idf_command` function in `lib/keyword_search.py` to expose the BM25 IDF calculation.
- Updated `cli/keyword_search_cli.py` to include `bm25idf` in the main execution flow and added a new `bm25idf` sub-command to the CLI.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RaY8118) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
