## typing-game-hono

> This document provides a comprehensive overview of the `typing-game-hono` project, designed to be used by the Gemini Code Assistant.

# Gemini Code Assistant Project Overview: typing-game-hono

This document provides a comprehensive overview of the `typing-game-hono` project, designed to be used by the Gemini Code Assistant.

## Project Description

`typing-game-hono` is an interactive web-based typing game built with modern web technologies. The game challenges players to type words or phrases accurately and quickly. It features a user-friendly interface, real-time feedback, and engaging gameplay mechanics. The project is structured as a full-stack application with a Next.js frontend and a Hono-based backend, ensuring a seamless and performant user experience.

## Key Technologies

The project leverages a range of modern technologies to deliver a robust and scalable application:

- **Frontend:**
  - **Next.js:** A React framework for building server-side rendered and statically generated web applications.
  - **React:** A JavaScript library for building user interfaces.
  - **Tailwind CSS:** A utility-first CSS framework for creating custom designs with ease.
  - **Shadcn/UI:** A collection of reusable UI components for building accessible and visually appealing interfaces.
  - **TypeScript:** A statically typed superset of JavaScript that enhances code quality and maintainability.

- **Backend:**
  - **Hono:** A lightweight and fast web framework for building APIs and web applications.
  - **Upstash Redis:** A serverless Redis database for managing game state and user data.
  - **Nodemailer:** A library for sending emails, used for features like contact forms or user notifications.

- **Development and Tooling:**
  - **ESLint:** A pluggable linter for identifying and fixing problems in JavaScript and TypeScript code.
  - **Prettier:** An opinionated code formatter that ensures consistent code style across the project.
  - **Bun:** A fast JavaScript runtime and package manager used for development and dependency management.

## Project Structure

The project follows a standard Next.js application structure, with key directories and files organized as follows:

- **`app/`**: The main application directory, containing the core components, pages, and logic of the application.
  - **`api/`**: Backend API routes built with Hono.
  - **`components/`**: Reusable React components used throughout the application.
  - **`context/`**: React context providers for managing global state.
  - **`lib/`**: Utility functions and helper modules.
  - **`styles/`**: Global styles and CSS configurations.
- **`public/`**: Static assets such as images, fonts, and sound files.
- **`constants/`**: Application-wide constants and configuration data.
- **`types/`**: TypeScript type definitions and interfaces.
- **`package.json`**: Project metadata, dependencies, and scripts.
- **`next.config.js`**: Configuration file for Next.js.
- **`tsconfig.json`**: TypeScript configuration file.

## Available Scripts

The following scripts are available in the `package.json` file to streamline development and maintenance tasks:

- **`dev`**: Starts the development server with hot-reloading enabled.
- **`build`**: Compiles the application for production.
- **`start`**: Starts the production server.
- **`lint`**: Lints the codebase using ESLint to identify and fix code quality issues.
- **`format`**: Formats the codebase using Prettier to ensure consistent code style.
- **`format:check`**: Checks the codebase for formatting issues without making any changes.

## Getting Started

To get started with the project, follow these steps:

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   ```
2. **Install dependencies:**
   ```bash
   bun install
   ```
3. **Run the development server:**
   ```bash
   bun run dev
   ```
4. **Open your browser:**
   Navigate to `http://localhost:3000` to view the application.

This `GEMINI.md` file provides a solid foundation for understanding and working with the `typing-game-hono` project. For more detailed information, refer to the official documentation of the technologies used.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shinguakira)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shinguakira)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
