## lodestone

> - `npm run dev` - Start development server

# Lodestone Development Guide

## Build Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production (runs TypeScript compile and Vite build)
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## Code Style Guidelines

- **Imports**: Group imports by type (React, libraries, local components)
- **Naming**: Use PascalCase for components, camelCase for variables/functions
- **TypeScript**: Use strict typing, define interfaces for props, avoid `any`
- **Components**: Use functional components with hooks, avoid class components
- **Error Handling**: Use try/catch with clear error messages
- **Comments**: Document complex logic, avoid unnecessary comments
- **State Management**: Prefer React hooks over Redux/context when possible
- **Formatting**: Indent with tabs, use trailing commas in objects/arrays
- **File Structure**: Group related files in folders, use index files for exports

## Project Context

This project is an AI-human collaborative writing and critical thinking tool using React, TypeScript, Tailwind, Remirror, and Dexie. The tool enables users to create text sessions, analyse them with language models, and edit/customize text annotations with various labels.

---
> Source: [MaggieAppleton/lodestone](https://github.com/MaggieAppleton/lodestone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
