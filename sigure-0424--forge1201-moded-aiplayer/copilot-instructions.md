## forge1201-moded-aiplayer

> This document serves as the primary instructional context for Gemini CLI when operating within the **Minecraft Forge 1.20.1 AI Player System (Forge1201AIP)** project.

# GEMINI.md - Project Instructional Context

This document serves as the primary instructional context for Gemini CLI when operating within the **Minecraft Forge 1.20.1 AI Player System (Forge1201AIP)** project.

## Project Overview
An autonomous AI player framework utilizing **Mineflayer** and **Large Language Models (LLMs)** tailored for **Minecraft 1.20.1 Forge** environments. It overcomes protocol incompatibilities (FML3 Handshake) and dynamic registry shifts common in modded servers.

### Architecture
- **Agent Manager (`src/agent_manager.js`)**: Orchestrator that manages bot lifecycles via child processes. It handles LLM interaction, message queuing, and recovery.
- **Bot Actuator (`src/bot_actuator.js`)**: The isolated child process running the Mineflayer instance. Implements action execution (combat, collection, crafting) and protocol bypasses.
- **Protocol Bridge (`src/forge_handshake_state_machine.js`)**: Intercepts and spoofs FML3 handshake packets.
- **Registry Injector (`src/dynamic_registry_injector.js`)**: Maps modded block/item IDs to valid internal definitions to maintain physics and pathfinding stability.

## Building and Running

### Prerequisites
- Node.js (>= 18 recommended for `fetch` support).
- A Minecraft Forge 1.20.1 server.
- An LLM provider (Ollama or OpenAI-compatible API).

### Key Commands
- **Install Dependencies**: `npm install`
- **Start the System**: `node index.js`
- **Run Test Suite**: `npm test`
- **Run E2E Integration Test**: `npm run test:e2e`

### Environment Variables (.env)
- `MC_HOST`: Server hostname.
- `MC_PORT`: Server port.
- `BOT_NAME`: Username for the bot.
- `OLLAMA_URL`: API endpoint for the LLM (e.g., `http://localhost:11434/api/generate`).
- `OLLAMA_API_KEY`: API key for the provider.

## Development Conventions

### 1. Stability First
- **Process Isolation**: Do NOT move heavy logic (pathfinding, physics) into the `AgentManager`. Keep it in the `BotActuator` to prevent main-loop blocks.
- **Protocol Bypasses**: If a new mod causes "unexpected tag end" or "stream desync" errors, add the problematic packet names to the `bypass` array in `src/bot_actuator.js`.

### 2. LLM Interaction
- **Action Format**: The LLM must respond with a JSON array of actions. See the prompt in `src/agent_manager.js` for supported schemas.
- **Sanitization**: Use `sanitizeLLMAction` in `AgentManager` to handle minor LLM formatting errors.

### 3. Registry & Physics
- **Vanilla-First Mapping**: When modded blocks are encountered, they are mapped to "stone" or "air" templates by default to ensure the physics engine doesn't crash.
- **Dictionary Updates**: Add specific block properties to `data/sample/configs/mod_blocks_dictionary.json` if a modded block requires specific hardness or transparency settings.

### 4. Testing
- **Mocking**: Use `tests/mock_forge_server.js` for protocol-level testing without a real Forge server.
- **Unit Tests**: Ensure logic changes are verified in `tests/test_fml3_handshake_logic_pure.js` or `tests/test_dynamic_registry_injector.js`.

## Deployment & Production
- The system is designed to run within a **Docker** environment as defined in `docker/compose.yaml`.
- Ensure `data/sample/configs` is mounted for proper RAG (Retrieval-Augmented Generation) of server constraints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sigure-0424)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sigure-0424)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
