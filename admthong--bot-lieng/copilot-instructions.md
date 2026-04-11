## bot-lieng

> This project, **Mezon Lieng Bot**, is a Discord-like bot designed for the **Mezon** platform. It facilitates **Liêng (Three-card Poker)** games directly within chat channels. The bot is built using **NestJS** (Node.js framework), **TypeScript**, and uses **PostgreSQL** for data persistence. It interacts with the Mezon platform using the `mezon-sdk`.

# Mezon Lieng Bot Context

## Project Overview

This project, **Mezon Lieng Bot**, is a Discord-like bot designed for the **Mezon** platform. It facilitates **Liêng (Three-card Poker)** games directly within chat channels. The bot is built using **NestJS** (Node.js framework), **TypeScript**, and uses **PostgreSQL** for data persistence. It interacts with the Mezon platform using the `mezon-sdk`.

### Key Features
*   **Liêng (Three-card Poker):** Supports multi-player games (2-8 players).
*   **Game Lifecycle:** Handles invites, joining, betting (call, raise, fold), dealing cards, and determining winners based on Liêng rules (Sáp > Liêng > Ảnh > Điểm).
*   **Currency:** Integrated token/money system for betting.
*   **Architecture:** Modular NestJS design with a command pattern for handling user inputs.

## Technical Architecture

### Tech Stack
*   **Framework:** NestJS v11 (Node.js >= 20)
*   **Language:** TypeScript
*   **Database:** PostgreSQL (via TypeORM)
*   **SDK:** `mezon-sdk` (v2.8.33)
*   **Package Manager:** Yarn (preferred) or NPM

### Directory Structure
*   `src/app.module.ts`: Root application module.
*   `src/main.ts`: Application entry point.
*   `src/bot/`: Core bot logic.
    *   `commands/`: Command implementations (e.g., `lieng/`, `admin/`, `system/`).
    *   `base/`: Abstract command classes and handlers (`GameBaseService`).
    *   `events/` & `listeners/`: Event handling for Mezon events (e.g., `ChannelMessage`).
    *   `models/`: TypeORM entities (`User`, `LiengGame`).
    *   `services/`: Business logic.
*   `src/mezon/`: Mezon SDK integration module.

## Building and Running

### Prerequisites
*   Node.js >= 20
*   PostgreSQL running locally or accessible via network.
*   `.env` file configured (see `env.example`).

### Key Commands
*   **Install Dependencies:** `yarn install` or `npm install`
*   **Development Server:** `npm run dev` or `yarn dev` (Watch mode)
*   **Production Build:** `npm run build` or `yarn build`
*   **Start Production:** `npm run start:prod` or `yarn start:prod`
*   **Testing:** `npm run test` or `yarn test`
*   **Linting:** `npm run lint` or `yarn lint`
*   **Migrations:** `npm run migration:run`

## Development Conventions

### Command Handling
*   **Prefix:** The bot listens for commands starting with `*` (e.g., `*lieng`).
*   **Implementation:** Commands are implemented as classes decorated with `@Command('name')` and extending `CommandMessage`.
*   **Registration:** The `CommandBase` and listeners route incoming messages to the appropriate command class.

### Database
*   **ORM:** TypeORM is used for database interactions.
*   **Entities:** defined in `src/bot/models/`.
*   **Migrations:** Managed via TypeORM CLI scripts.

### Code Style
*   Follows standard NestJS and TypeScript conventions.
*   Uses `prettier` and `eslint` for formatting and linting.
*   Async/await is used for asynchronous operations.

## Documentation Notes
*   **Mezon SDK:** The bot relies heavily on `mezon-sdk`. Refer to `src/mezon/` for integration details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/admthong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/admthong)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
