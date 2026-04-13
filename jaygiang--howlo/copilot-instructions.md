## howlo

> - Start server: `npm start`

# Bingo App - Development Guide

## Build & Run Commands
- Start server: `npm start` 
- Run in dev mode: `nodemon server.js`
- Install dependencies: `npm install`

## Code Style Guidelines
- **Formatting**: Use 2-space indentation
- **Naming**: 
  - camelCase for variables and functions
  - PascalCase for models
- **Error Handling**: Always use try/catch for async operations with descriptive console.error messages
- **Imports**: Group imports in order: core modules, external modules, internal modules
- **Organization**: Maintain route handlers in server.js, database models in separate files
- **Environment Variables**: Access with process.env, configure with dotenv
- **MongoDB**: Use async/await with mongoose, handle connection errors explicitly
- **Slack API**: Use the WebClient from @slack/web-api for all Slack interactions

## Project Structure
- **server.js**: Main application entry point
- **Accomplishment.js**: Mongoose schema for bingo accomplishments
- **.env**: Required environment variables: MONGODB_URI, SLACK_BOT_TOKEN, SECRET_KEY, APP_BASE_URL

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaygiang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jaygiang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
