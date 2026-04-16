## supreme-waffle

> This file provides instructional context about the project for future interactions with the Gemini AI agent.

# GEMINI Project Context: VFD Control Dashboard

This file provides instructional context about the project for future interactions with the Gemini AI agent.

## Project Overview

This is a full-stack IoT project for remotely controlling and monitoring a Mitsubishi FR-E700 VFD (Variable Frequency Drive).

The project has two main components:

1.  **Frontend:** A responsive web dashboard built with **React**, **Vite**, and **TypeScript**. It uses **Tailwind CSS** and **shadcn/ui** for styling and components. It communicates with the hardware device in real-time via an **MQTT** broker.
2.  **Firmware:** An **ESP32** microcontroller runs firmware written in C++/Arduino (`.ino`). This device interfaces directly with the VFD via the Modbus RTU protocol (over RS-485) and connects to the internet using a SIM800C GPRS module to communicate with the MQTT broker.

The overall architecture allows a user to send commands (e.g., start motor, set frequency) from the web UI, which are relayed through the MQTT broker to the ESP32, which in turn controls the VFD. The ESP32 also sends status data back to the UI.

## Building and Running (Frontend)

The frontend is a standard Vite-powered React application. Key commands are defined in `package.json`.

- **Install dependencies:**
  ```bash
  npm install
  ```
- **Run development server:**
  ```bash
  npm run dev
  ```
- **Build for production:**
  ```bash
  npm run build
  ```
- **Lint the code:**
  ```bash
  npm run lint
  ```
- **Preview the production build:**
  ```bash
  npm run preview
  ```

## Development Conventions

- **Structure:** The React source code is located in the `src/` directory. It follows a component-based architecture.
  - `src/app/`: Contains top-level pages for different routes (dashboard, settings, etc.).
  - `src/components/`: Contains reusable React components.
  - `src/components/ui/`: Contains base UI components from the shadcn/ui library.
  - `src/lib/`: Contains utility functions.
- **Styling:** The project uses Shadcn for styling, Use MCP server for building anything.
- **Configuration:**
  - Frontend configuration (MQTT credentials) is managed via a `.env` file in the project root. See `README.md` for required variables.
  - Firmware configuration (GPRS & MQTT settings) is managed in the `config.h` file.
- **Linting & Formatting:** The project is configured with ESLint (`eslint.config.js`) and TypeScript (`tsconfig.json`) to enforce code quality and consistency.
- **Path Aliases:** A `@` alias is configured in `vite.config.ts` to point to the `src/` directory for cleaner import paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imkenough) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
