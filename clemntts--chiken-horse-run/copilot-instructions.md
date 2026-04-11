## chiken-horse-run

> This project is a 2D multiplayer platformer game, inspired by "Ultimate Chicken Horse," where players collaboratively build levels before racing through them.

# GEMINI Project Context: Party-Platformer-Builder

## Project Overview

This project is a 2D multiplayer platformer game, inspired by "Ultimate Chicken Horse," where players collaboratively build levels before racing through them.

It's a full-stack TypeScript project structured as a monorepo:
-   `client/`: A Vite-powered React application that uses the **Excalibur.js** game engine to render the game.
-   `server/`: A Node.js backend using **Express** and **Colyseus.io** for real-time multiplayer state synchronization and communication via WebSockets.
-   `shared/`: Intended for code shared between the client and server (e.g., game state schemas, constants).

The project follows a "Single-Port" architecture: in production, the Express server handles both the Colyseus WebSocket connections and serves the static build of the React client.

## Building and Running

### Development

For development, run the client and server in separate terminals.

**1. Run the Server:**
```bash
npm run dev:server
```
*Note: The `dev:server` script was missing. I have added it to `server/package.json` to run `ts-node-dev src/index.ts`.*

**2. Run the Client:**
```bash
npm run dev:client
```

### Production

To simulate a production environment:
```bash
npm start
```
This command first builds the client application into `client/dist`, then starts the server, which serves the built files.

### Sharing for Multiplayer Testing

The project is pre-configured to use `ngrok` to easily expose the local server to the internet for multiplayer testing.
```bash
npm run share
```
This will start an ngrok tunnel to port `2567`, which is the default port for the game server.

## Key Files

-   `PROJECT_EXPLAIN.md`: The high-level vision and technical guidelines for the project.
-   `package.json` (root): Contains the main scripts for running and building the client and server together.
-   `server/src/index.ts`: The entry point for the backend server. It initializes Express and the Colyseus game server.
-   `server/src/GameRoom.ts`: (Assumed) The core of the game logic on the server, defining the Colyseus room and state.
-   `client/src/main.tsx`: The entry point for the React application.
-   `client/src/App.tsx`: The main React component, likely responsible for setting up the Excalibur game.
-   `client/vite.config.ts`: Vite configuration for the client-side development server and build process.

## Development Conventions

-   **Language:** Strict TypeScript is used across the entire stack.
-   **Networking:** The client is configured to connect to the server using `window.location` to ensure it works seamlessly when deployed or shared via ngrok.
-   **Focus:** The development priority is on implementing core game mechanics (grid system, player movement, traps) before visual polish.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ClemNTTS)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ClemNTTS)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
