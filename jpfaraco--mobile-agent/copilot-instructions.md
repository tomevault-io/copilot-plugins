## mobile-agent

> This document provides a comprehensive overview of the Autonomous Mobile Agent project, its architecture, and instructions for setup and execution.

# GEMINI.md: Autonomous Mobile Agent

This document provides a comprehensive overview of the Autonomous Mobile Agent project, its architecture, and instructions for setup and execution.

## Project Overview

This project implements an autonomous agent capable of navigating and interacting with Android applications to accomplish a predefined "mission." The agent uses a combination of computer vision and UI analysis to understand the device's screen and decide on the next course of action.

The core of the agent is powered by a large language model (specifically, `gpt-4o`), which receives the current screen's screenshot, the UI's XML layout, the overall mission, and a history of past actions. Based on this context, the model determines the next action to take.

### Key Technologies

*   **Python 3:** The primary programming language.
*   **OpenAI API (`gpt-4o`):** The "brain" of the agent, responsible for decision-making.
*   **Pure Python ADB (`ppadb`):** For communication with the Android device via the Android Debug Bridge (ADB).
*   **Pillow:** For handling and processing screenshots.
*   **python-dotenv:** For managing environment variables, particularly the OpenAI API key.
*   **fpdf2:** For generating PDF reports.

### Architecture

The project's logic is consolidated in `agent.py`, which is structured into four main logical components, separated by comments:

1.  **Vision Module:** Responsible for analyzing the device's screen. It encodes the screenshot, prepares the prompt for the language model, and sends the request to the OpenAI API.
2.  **Interaction Module:** Handles all direct communication with the Android device. This includes taking screenshots, retrieving the UI XML layout, and executing actions.
3.  **Reporting Module:** Generates a PDF report of the agent's run, including the mission, steps taken, screenshots, and the agent's reasoning.
4.  **Agent Core:** The central orchestrator. It manages the main loop of the agent, which consists of perceiving the environment, thinking about the next action, and acting on the environment.

### Memory

The agent has a memory mechanism to avoid getting stuck in loops. It keeps a history of the actions it has taken on each screen it has visited. When it revisits a screen, it is reminded of its previous actions on that screen, which helps it to make a better decision and avoid repeating failed attempts.

### Agent Actions

The agent can perform the following actions:

*   **TAP:** Taps on a specific element on the screen, identified by its bounds in the UI XML.
*   **GO_BACK:** Executes the system's "back" command, which is equivalent to pressing the back button.
*   **SCROLL:** Scrolls the screen down or up to discover more content.

### Reporting

At the end of each run, the agent generates a PDF report that summarizes the mission. The report includes:
*   The mission statement.
*   A step-by-step log of the agent's actions.
*   The screenshot for each step.
*   The agent's reflection and thought process for each step.
*   The final status of the mission (successful or failed).

The reports are saved in the `reports` directory.

## Building and Running

To run the agent, follow these steps:

1.  **Prerequisites:**
    *   Python 3.
    *   An Android device with USB debugging enabled.
    *   ADB (Android Debug Bridge) installed and running on your computer.

2.  **Setup:**
    *   Clone the repository.
    *   Install the required Python packages:
        ```bash
        pip install -r requirements.txt
        ```
    *   Create a `.env` file in the root of the project and add your OpenAI API key:
        ```
        OPENAI_API_KEY="your-api-key-here"
        ```

3.  **Running the Agent:**
    *   Connect your Android device to your computer.
    *   Ensure the ADB server is running and your device is recognized (`adb devices`).
    *   Execute the `agent.py` script with a mission passed as an argument:
        ```bash
        python agent.py "Your mission description here"
        ```
    *   You can also optionally specify the maximum number of steps the agent can take:
        ```bash
        python agent.py "Your mission description here" --max-steps 20
        ```

The agent will then start running, attempting to complete the mission provided via the command line.

## Development Conventions

*   **Modular Design:** The code is organized into logical modules within `agent.py`. Future development should maintain this separation of concerns.
*   **Clear Naming:** Functions and variables are named descriptively to enhance readability.
*   **Environment Variables:** Sensitive information, such as API keys, should be stored in a `.env` file and not hardcoded in the source.
*   **Error Handling:** The code includes basic error handling for common issues like missing API keys or device connection failures.
*   **Extensibility:** The agent's "mission" is configurable via a command-line argument.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpfaraco)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jpfaraco)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
