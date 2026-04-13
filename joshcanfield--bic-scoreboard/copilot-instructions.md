## bic-scoreboard

> This project is a hockey scoreboard application for the Bremerton Ice Arena. It is written in Java 21 and built with Gradle.

# Project Overview

This project is a hockey scoreboard application for the Bremerton Ice Arena. It is written in Java 21 and built with Gradle.

The application consists of a server that manages the game state and communicates with both hardware and browser clients.

- **Jetty + RESTEasy:** Expose a REST API on port `8080` for querying and updating the game clock, scores, penalties, and buzzer.
- **Native WebSocket server:** Pushes real-time score and clock updates to connected web clients on port `8082`.
- **jSerialComm:** Connects to the physical scoreboard over a serial port so on-ice displays mirror the server state.
- **Dagger:** Wires these components together and produces singletons for the scoreboard state and services.

For more detailed information on project structure, build commands, coding style, and other guidelines, please refer to [AGENTS.md](AGENTS.md).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshcanfield)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshcanfield)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
