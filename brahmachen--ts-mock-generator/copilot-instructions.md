## ts-mock-generator

> This document provides a comprehensive overview of the `ts-mock-generator` VS Code extension, detailing its purpose, technology stack, structure, and core functionalities.


# GEMINI Analysis of ts-mock-generator

This document provides a comprehensive overview of the `ts-mock-generator` VS Code extension, detailing its purpose, technology stack, structure, and core functionalities.

## Project Introduction

`ts-mock-generator` is a Visual Studio Code extension designed to streamline the process of creating mock data for TypeScript projects. It leverages AI (specifically the DeepSeek API) to generate realistic mock data and JSON schemas directly from TypeScript interfaces and type definitions. This tool is aimed at developers who need to quickly produce test data, prototypes, or mock API responses.

The extension integrates seamlessly into the VS Code editor, providing context menu commands for one-click generation of mock files.

## Technology Stack

- **Language**: TypeScript
- **Framework**: Node.js with the VS Code Extension API
- **Key Dependencies**:
    - `@faker-js/faker`: For generating realistic-looking fake data.
    - `openai`: To interact with the DeepSeek AI API.
    - `ts-json-schema-generator`: For creating JSON schemas from TypeScript types.
- **Dev Tools**:
    - `eslint`: For code linting.
    - `prettier`: For code formatting.
    - `typescript`: The TypeScript compiler.

## Project Structure

```
/
├─── .vscode/         # VS Code launch and task configurations
├─── src/             # Main source code for the extension
│   ├─── extension.ts # Main entry point of the extension
│   ├─── genCode.ts   # Logic for generating mock code (Faker.js)
│   ├─── genSchema.ts # Logic for generating JSON schemas
│   └─── ...
├─── examples/        # Example usage files
├─── out/             # Compiled JavaScript output
├─── package.json     # Project metadata, dependencies, and scripts
├─── README.md        # Project documentation
└─── tsconfig.json    # TypeScript compiler options
```

## Available Commands

The following scripts are defined in `package.json` and can be run using `npm run <script_name>`:

- `npm run vscode:prepublish`: Compiles the project before publishing.
- `npm run compile`: Compiles TypeScript to JavaScript.
- `npm run watch`: Watches for changes and recompiles automatically.
- `npm run format`: Formats the code using Prettier.
- `npm run lint`: Lints the code using ESLint.

## Core Functionalities

Based on the source code and documentation, the extension provides the following key features:

1.  **AI-Powered Mock Data Generation**: Utilizes the DeepSeek AI to understand TypeScript interfaces and generate relevant mock data.
2.  **Faker.js Code Generation**: Creates a `.mock.js` file containing `Faker.js` code that can be executed to produce mock data. This is triggered by the `TS -> Faker Mock` command.
3.  **Direct JSON Mock Generation**: Generates a `.mock.json` file with the final mock data object. This is triggered by the `TS -> JSON Mock` command.
4.  **JSON Schema Generation**: Can generate a `.schema.json` file from a TypeScript interface.
5.  **Run Mock Scripts**: Provides a command (`Run MockJS`) to execute the generated `.mock.js` files and view the output directly within VS Code.
6.  **Context Menu Integration**: All major commands are accessible via the right-click context menu in the editor, with visibility controlled by the file type (`.ts`, `.tsx`, `.schema.json`, `.mock.js`).
7.  **Configurable AI Settings**: Users can configure their DeepSeek API key, endpoint URL, model, and temperature through the VS Code settings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brahmachen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brahmachen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
