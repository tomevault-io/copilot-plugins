## codepad

> This is a collaborative code editor application built with React frontend and Node.js backend.

# CodeCrush

## Project Overview
This is a collaborative code editor application built with React frontend and Node.js backend.

## Package Management
- This project uses **yarn** as the package manager
- Always use `yarn` commands instead of `npm`
- Use `yarn add` for dependencies and `yarn add -D` for dev dependencies

## Development Setup
- The client and server are served from the same port
- To start the application: `yarn build && yarn start`
- The backend serves the built React app as static files
- WebSocket connections are used for real-time collaboration

## Project Structure
- `/client/` - React frontend application
- `/server/` - Node.js backend with Socket.IO
- `/lsp-gateway/` - Containerized LSP gateway for Kotlin/Java language servers
- Both client and server have their own package.json files

## Key Technologies
- Frontend: React, TypeScript, Monaco Editor, Socket.IO client
- Backend: Node.js, Express, Socket.IO, WebSocket (for LSP)
- Database: PostgreSQL with migrations
- Code Execution: Piston API integration
- Language Support: Enhanced Kotlin and Java support with LSP gateway integration

## Development Guidelines
- Use TypeScript for type safety
- Follow React hooks patterns
- Implement real-time features using Socket.IO
- Handle authentication with Google OAuth
- Use Monaco Editor for code editing with language-specific features

## Git Workflow
- **DO NOT commit anything to git automatically**
- The user will handle all git operations manually
- Do not run git commands or suggest git commits
- Focus on code changes and functionality only

## File Naming Conventions
- React components: PascalCase (e.g., `Room.tsx`, `AuthContext.tsx`)
- Utility files: camelCase (e.g., `monaco-setup.ts`)
- Configuration files: lowercase with hyphens (e.g., `package.json`)

## Code Style
- Use functional components with hooks
- Prefer const/let over var
- Use async/await over promises when possible
- Handle errors gracefully with try/catch blocks
- Add proper TypeScript types for all functions and components

## Testing and Building
- Run tests with `yarn test`
- Build for production with `yarn build`
- Start production server with `yarn start`
- Development mode: separate client and server processes

## Special Features
- Real-time collaborative editing
- Multiple programming language support
- Code execution via Piston API
- Session playback functionality
- Enhanced Kotlin and Java language support with LSP
- User authentication and room management 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andygale) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
