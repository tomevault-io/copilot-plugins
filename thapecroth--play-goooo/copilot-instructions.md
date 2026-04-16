## play-goooo

> This document provides a comprehensive guide for developers working on the Go Game project. It covers the project's architecture, development workflow, and key components.

# GEMINI.md

This document provides a comprehensive guide for developers working on the Go Game project. It covers the project's architecture, development workflow, and key components.

## Project Overview

This project is a web-based implementation of the game of Go, featuring multiple AI opponents. The application allows users to play Go against different AI models, including a classic rule-based AI, a Deep Q-Network (DQN) AI, and an AlphaGo-style AI.

The project has a dual-backend architecture:
- A **Node.js server** handles the main web application and the classic AI.
- A **Python server** provides the more advanced AI functionalities (DQN and AlphaGo).

## Technologies

- **Frontend:** HTML, CSS, Vanilla JavaScript
- **Backend (Web Server):** Node.js, Express, Socket.IO
- **Backend (AI Server):** Python, FastAPI, python-socketio
- **AI & Machine Learning:** PyTorch, NumPy, Numba
- **Package Management:** npm, pip

## Project Structure

```
/
├───.gitignore
├───alpha_go.py           # AlphaGo (MCTS + NN) AI implementation
├───benchmark_ai.py       # Script to benchmark AI performance
├───mcts_go.py            # Monte Carlo Tree Search (MCTS) implementation
├───optimized_go.py       # Optimized Go game logic with NumPy and Numba
├───package.json          # Node.js dependencies and scripts
├───requirements.txt      # Python dependencies
├───self_play.py          # Script for training AlphaGo AI with self-play
├───server.js             # Node.js web server (main entry point)
├───server.py             # Python AI server
├───train_dqn.py          # Script for training the DQN model
├───game/
│   ├───GoAI.js           # Classic rule-based AI for the Node.js server
│   └───GoGame.js         # Core Go game logic in JavaScript
├───models/               # Directory for trained AI models
├───public/               # Frontend static files (HTML, CSS, JS)
└───tests/                # Test files for the project
```

## Getting Started

### Prerequisites

- Node.js and npm
- Python 3.x and pip
- `pyenv` (recommended for managing Python environments)

### Installation

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2.  **Install Node.js dependencies:**
    ```bash
    npm install
    ```

3.  **Set up Python environment and install dependencies:**
    ```bash
    # **IMPORTANT**: This project uses a specific Python version managed by pyenv.
    # Activate the pyenv environment before running any Python commands.
    pyenv activate play-gooo

    # Install Python packages
    pip install -r requirements.txt
    ```

## Development Commands

**IMPORTANT**: Always activate the `pyenv` environment before running any Python commands:
`pyenv activate play-gooo`

### Running the Application

You can run the application with either the Node.js backend (for the classic AI) or the Python backend (for advanced AIs).

-   **Node.js Server (Default):**
    ```bash
    # For production
    npm start

    # For development with auto-reloading
    npm run dev
    ```

-   **Python Server (for DQN and AlphaGo AI):**
    ```bash
    # Make sure your pyenv environment is activated
    npm run start:python
    ```

    The application will be available at `http://localhost:3000`.

### Training AI Models

-   **Train the DQN Model:**
    ```bash
    # Activate your Python virtual environment first
    python train_dqn.py --board-size 9 --depth 6 --episodes 10000
    ```
    You can customize training parameters like `--board-size`, `--depth`, `--episodes`, `--learning-rate`, etc.

-   **Train the AlphaGo Model:**
    ```bash
    # Activate your Python virtual environment first
    python self_play.py
    ```

## AI Models

The application supports three types of AI opponents, which can be selected from the UI:

1.  **Classic AI:** A traditional rule-based AI with two algorithm options:
    -   **Minimax:** A standard minimax algorithm with alpha-beta pruning.
    -   **MCTS:** A Monte Carlo Tree Search algorithm.

2.  **DQN AI:** A Deep Q-Network that has been trained to play Go. This requires the Python server to be running.

3.  **AlphaGo AI:** An AI based on the AlphaGo architecture, combining MCTS with a policy and value network. This also requires the Python server.

Trained models are saved in the `models/` directory.

## Key Features

-   Play Go on 9x9, 13x13, and 19x19 boards.
-   Real-time gameplay with Socket.IO.
-   Multiple AI opponents with configurable difficulty.
-   DQN Q-value visualization to see the AI's "thoughts".
-   Optimized game logic in Python using NumPy and Numba for high performance.
-   Scripts for training your own AI models.

## Testing

The project includes tests for the optimized Go game logic. To run the tests:

```bash
python -m unittest tests/test_optimized_go.py
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thapecroth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
