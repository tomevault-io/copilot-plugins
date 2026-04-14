## cluster-sentinel

> This project, "The Cluster Sentinel," is a lightweight, distributed server monitoring system written in Python. It consists of three main components:

# GEMINI.md

## Project Overview

This project, "The Cluster Sentinel," is a lightweight, distributed server monitoring system written in Python. It consists of three main components:

1.  **Agent (`agent.py`):** A client-side script that runs on a machine to be monitored. It collects system health data (CPU, RAM) and sends it to the master server.
2.  **Collector (`master_server.py`):** A master server that listens for data from one or more agents. It receives the data and writes it to a shared memory block for other processes to access.
3.  **Dashboard (`dashboard.py` or integrated into `master_server.py`):** A simple web server that reads the monitoring data from shared memory and displays it on a real-time web dashboard.

The system uses TCP sockets for communication between the agent and the collector, and the `multiprocessing` library with shared memory (`multiprocessing.shared_memory`) and locks (`multiprocessing.Lock`) for inter-process communication (IPC) between the collector and the dashboard. Data is serialized using `pickle` before being sent over the network.

## Building and Running

There are no external dependencies mentioned, so the project should run with a standard Python installation. The project is designed to be run in separate pieces, which are then integrated.

### Running the Components

1.  **Run the Collector (Master Server):**
    ```bash
    python master_server.py
    ```
    This will start the socket server to listen for agent data and manage the shared memory.

2.  **Run the Agent:**
    ```bash
    python agent.py
    ```
    This will start collecting and sending system stats to the master server. You can run this on the same or different machines.

3.  **View the Dashboard:**
    Open a web browser and navigate to `http://localhost:8080` (or the IP address of the machine running the master server). The page should display the real-time CPU and RAM stats and auto-refresh every 2 seconds.

## Development Conventions

The project is structured as a set of small, focused Python scripts. The `project_plan.md` file provides a clear, phased approach to development, indicating an iterative and modular development style.

*   **Modularity:** The project is broken down into logical components (agent, collector, dashboard) that can be developed and tested independently.
*   **Inter-process Communication:** The use of `multiprocessing` with shared memory and locks is a key architectural pattern for communication between the data collection and web server processes.
*   **Networking:** The project uses low-level socket programming for custom TCP communication and a basic HTTP server implementation from scratch.
*   **Code Style:** The provided file names and the project plan suggest a preference for clear, descriptive names (e.g., `get_system_stats()`, `master_server.py`).
*   **Testing:** The project plan includes "Checkpoints" at each phase, suggesting a manual, iterative testing process. For example, using a `test_reader.py` script to verify the shared memory is working correctly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YuHan-J-Nam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YuHan-J-Nam)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
