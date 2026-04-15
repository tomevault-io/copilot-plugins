## electron-course

> This document provides a guide for developers to understand, run, and contribute to the `electron-course` project.

# Gemini Code-Guide: `electron-course`

This document provides a guide for developers to understand, run, and contribute to the `electron-course` project.

## Project Overview

This is a desktop application built with **Electron**, **React**, and **TypeScript**. It functions as a host application for devices communicating over a serial port, specifically designed to parse and display data from currency counting machines.

### Core Features

-   **Serial Protocol Parsing**: Extensible framework for handling serial data.
-   **Real-time Data Display**: Visualizes data from the serial port.
-   **File Management & Reporting**: Exports data to Excel and PDF formats.
-   **Modern UI**: Built with React, supporting internationalization.

## Technology Stack

-   **Framework**: [Electron](https://www.electronjs.org/)
-   **Frontend**: [React](https://reactjs.org/)
-   **Language**: [TypeScript](https://www.typescriptlang.org/)
-   **Build Tool**: [Vite](https://vitejs.dev/)
-   **Testing**: [Playwright](https://playwright.dev/) for E2E tests, [Vitest](https://vitest.dev/) for unit tests.
-   **Installer**: [electron-builder](https://www.electron.build/)

## Development Setup & Commands

### 1. Install Dependencies

First, install the necessary `npm` packages.

```bash
npm install
```

### 2. Run in Development Mode

This command starts the application in development mode with hot-reloading enabled. It runs both the React frontend and the Electron main process in parallel.

```bash
npm run dev
```

### 3. Build for Production

To create a production-ready build, use the following command. This will transpile the TypeScript code and bundle the application.

```bash
npm run build
```

### 4. Run Tests

The project has both unit and end-to-end (E2E) tests.

-   **Run all unit tests:**
    ```bash
    npm run test:unit
    ```
-   **Run E2E tests:**
    ```bash
    npm run test:e2e
    ```

### 5. Create a Distributable Package

To build the application and package it for distribution (e.g., as an `.exe` or `.dmg` file), use one of the following commands:

-   **For Windows:**
    ```bash
    npm run dist:win
    ```
-   **For macOS:**
    ```bash
    npm run dist:mac
    ```
-   **For Linux:**
    ```bash
    npm run dist:linux
    ```

## Project Structure

-   `src/electron/`: Contains the Electron main process code. This is where you'll find logic for window management, file system access, and IPC communication.
-   `src/ui/`: Contains the React frontend code, including all components, hooks, and styles.
-   `electron-builder.json`: Configuration file for `electron-builder`, which defines how the application is packaged.
-   `vite.config.ts`: Configuration file for Vite.

This guide should help you get started. Let me know if you have any other questions!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yedouuu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yedouuu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
