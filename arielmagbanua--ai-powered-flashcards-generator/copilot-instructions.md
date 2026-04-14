## ai-powered-flashcards-generator

> This project is a Firebase-based backend system designed to generate flashcards using AI. It utilizes Firebase Cloud Functions (v2) written in TypeScript to handle logic, likely intended for integration with AI models (such as Gemini) to automate flashcard creation.

# AI-Powered Flashcards Generator

This project is a Firebase-based backend system designed to generate flashcards using AI. It utilizes Firebase Cloud Functions (v2) written in TypeScript to handle logic, likely intended for integration with AI models (such as Gemini) to automate flashcard creation.

## Project Structure

- `functions/`: Contains the Firebase Cloud Functions source code.
  - `src/index.ts`: The main entry point for Cloud Functions.
  - `package.json`: Dependencies and scripts for the functions environment.
  - `tsconfig.json`: TypeScript configuration.

## Key Technologies

- **Firebase Cloud Functions (v2)**: Serverless backend logic.
- **TypeScript**: Typed JavaScript for better development experience and reliability.
- **Node.js 24**: The target runtime for Cloud Functions.
- **ESLint**: Linting using the Google configuration.

## Getting Started

### Prerequisites

- [Node.js 24+](https://nodejs.org/)
- [Firebase CLI](https://firebase.google.com/docs/cli) (`npm install -g firebase-tools`)

### Installation

Navigate to the `functions/` directory and install dependencies:

```bash
cd functions
npm install
```

### Local Development

To run the functions locally using the Firebase Emulator Suite:

```bash
cd functions
npm run serve
```

This will compile the TypeScript code and start the emulator.

### Deployment

To deploy the functions to your Firebase project:

```bash
cd functions
npm run deploy
```

## Development Commands

All commands should be run within the `functions/` directory:

- `npm run build`: Compiles TypeScript to JavaScript (output in `lib/`).
- `npm run lint`: Runs ESLint to check for code style issues.
- `npm run shell`: Starts a local interactive shell for testing functions.
- `npm run logs`: Streams logs from the deployed functions.

## Coding Standards

- **Style**: Adheres to the [Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html).
- **Triggers**: Prefers Firebase Functions v2 `onRequest` and other v2 triggers for better performance and flexibility.
- **Concurrency**: Global options are set to limit instances (defaulting to 10) to manage costs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arielmagbanua)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/arielmagbanua)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
