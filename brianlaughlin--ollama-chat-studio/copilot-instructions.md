## ollama-chat-studio

> This project, "Ollama Chat Studio," is a Python-based web application that provides a graphical user interface (GUI) for interacting with local AI models through the Ollama API. The application is built using the Streamlit framework, which allows for the rapid development of interactive web apps.

# GEMINI.md

## Project Overview

This project, "Ollama Chat Studio," is a Python-based web application that provides a graphical user interface (GUI) for interacting with local AI models through the Ollama API. The application is built using the Streamlit framework, which allows for the rapid development of interactive web apps.

The primary purpose of this application is to offer a feature-rich and user-friendly chat interface for Ollama models, including features like model comparison, theme switching, and advanced parameter tuning.

**Key Technologies:**

*   **Backend:** Python
*   **Web Framework:** Streamlit
*   **API Interaction:** `requests` library to communicate with the Ollama API.

**Architecture:**

The application is a single-script Streamlit application (`ollama-chat-app.py`). It manages the application state, UI components, and interaction with the Ollama API within this single file. The application state is managed using Streamlit's session state feature.

## Building and Running

### Prerequisites

*   Python 3.8 or higher
*   Ollama installed and running

### Installation

1.  **Install Python dependencies:**

    ```bash
    pip install streamlit requests
    ```

2.  **Run the application:**

    On Windows, you can use the provided batch file:

    ```bash
    run_streamlit.bat
    ```

    Alternatively, you can run the application directly using Streamlit:

    ```bash
    streamlit run ollama-chat-app.py
    ```

## Development Conventions

### Coding Style

The code follows general Python conventions (PEP 8), although there are no explicit linting or formatting configurations found in the project. The code is well-structured with functions for specific tasks like fetching models, streaming responses, and exporting conversations.

### Testing

There are no automated tests in this project. Testing appears to be done manually by running the application and interacting with the UI.

### Contribution

The `README.md` file mentions that contributions are welcome and suggests opening issues or submitting pull requests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brianlaughlin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brianlaughlin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
