## prompthub

> - Start server: `npm start` or watch mode: `npm run server`

# PromptHub Development Guide

## Commands
- Start server: `npm start` or watch mode: `npm run server`
- Start client: `cd client && npm start`
- Run concurrently: `npm run dev`
- Run client tests: `cd client && npm test`
- Run single test: `cd client && npm test -- -t "test name"`
- Build client: `cd client && npm run build`
- Create production build: `cd client && npm run build && npm start`

## Code Quality Tools
- Run server-side linting: `npm run lint`
- Fix server-side lint issues: `npm run lint:fix`
- Format server-side code: `npm run format`
- Run client-side linting: `cd client && npm run lint`
- Fix client-side lint issues: `cd client && npm run lint:fix`
- Format client-side code: `cd client && npm run format`
- Lint entire project: `npm run lint:all`
- Format entire project: `npm run format:all`

## Code Style Guidelines
- **Imports**: Group by type (React, libraries, components, styles)
- **Formatting**: 2-space indentation, semicolons, max 100 chars per line
- **Naming**: camelCase (variables/functions), PascalCase (components/classes), UPPERCASE (constants)
- **Components**: One component per file, use functional components with hooks
- **State Management**: Use React hooks (useState, useContext)
- **API Calls**: Centralize in dedicated service files with proper error handling
- **Error Handling**: Use try/catch with specific error messages
- **Mongoose**: Define schemas with validation, use timestamps
- **Authentication**: JWT tokens with proper expiration
- **Localization**: Support Japanese and English (comments in both languages)
- **Testing**: Component/unit tests with React Testing Library
- **Environment**: Use .env for configuration (see .env.example)
- **Linting**: Always run linter before committing code (`npm run lint:all`)
- **Formatting**: Ensure code is properly formatted (`npm run format:all`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hrdmtr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hrdmtr)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
