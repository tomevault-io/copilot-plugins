## imperium

> Imperium is a distributed resource trading simulation built in Elixir. It models a space trading game where autonomous ships, each running as an independent Elixir process, navigate a sector map, gather resources, and trade with each other. The system is designed to explore concurrent systems, the actor model, and OTP patterns.

# Imperium: A Distributed Space Trading Simulation

## Project Overview

Imperium is a distributed resource trading simulation built in Elixir. It models a space trading game where autonomous ships, each running as an independent Elixir process, navigate a sector map, gather resources, and trade with each other. The system is designed to explore concurrent systems, the actor model, and OTP patterns.

The core of the application revolves around the following concepts:

*   **Ships:** Each ship is an independent process (likely a `GenServer`) with its own state, including position, inventory, and AI state.
*   **SectorMap:** A central coordinator that tracks the positions of all ships.
*   **FlightControl:** A `GenServer` that manages ETS tables for ship locations and sectors, ensuring data consistency.
*   **Message Passing:** Ships and other components communicate via message passing, avoiding shared state.
*   **ETS Tables:** Used for efficient storage and retrieval of ship locations and sector data.

The project is in its early stages, with a focus on implementing the core mechanics of ship movement and registration. Future goals include adding resource gathering, trading, and a network protocol layer to allow external clients to interact with the simulation.

## Building and Running

This is a standard Elixir project. The following `mix` commands are used for common development tasks:

*   **Install dependencies:**
    ```sh
    mix deps.get
    ```

*   **Compile the project:**
    ```sh

    mix compile
    ```

*   **Run tests:**
    ```sh
    mix test
    ```

*   **Run tests with coverage:**
    ```sh
    mix test --cover
    ```

*   **Run the application in an interactive shell:**
    ```sh
    iex -S mix
    ```

*   **Format the code:**
    ```sh
    mix format
    ```

## Development Conventions

*   **Testing:** The project uses `ExUnit` for testing. Tests are located in the `test` directory and generally mirror the structure of the `lib` directory.
    *   Tests that require shared resources like ETS tables use `setup` blocks to create them and `on_exit` callbacks for cleanup.
    *   Helper functions are used within test modules to simplify test setup and assertions.
    *   Tests are written to be asynchronous (`async: true`) where possible.

*   **Code Style:** The project uses the standard Elixir code formatter (`mix format`). The formatting rules are defined in the `.formatter.exs` file.

*   **Supervision:** The application is being updated to use a supervision tree. The main application module is `Imperium`, which starts a `FlightControl` supervisor. This supervisor will be responsible for managing the lifecycle of the core application processes, such as the `GenServer` that manages the ETS tables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linuxunil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/linuxunil)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
