## educare

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is EduCare - an Educational AI Assistant application - a React-based web app designed for Lumina Foundation to create, manage, and chat with AI assistants for educational support, powered by Google's Gemini API. The app features advanced RAG (Retrieval-Augmented Generation) capabilities with support for PDF, DOCX, and MD files, Turso DB for cloud persistence, and comprehensive assistant sharing functionality with QR codes.

## Development Commands

- **Start development server**: `pnpm run dev` (runs on Vite)
- **Build for production**: `pnpm run build`
- **Preview production build**: `pnpm run preview`
- **Install dependencies**: `pnpm install`

### Linting & Formatting

- **Lint code**: `pnpm run lint`
- **Auto-fix linting issues**: `pnpm run lint:fix`
- **Format code**: `pnpm run format`
- **Check formatting**: `pnpm run format:check`
- **Type checking**: `pnpm run typecheck`
- **Run all quality checks**: `pnpm run quality`

### Testing

- **Run tests**: `pnpm run test`
- **Run tests in watch mode**: `pnpm run test:watch`
- **Run tests with UI**: `pnpm run test:ui`
- **Generate coverage report**: `pnpm run test:coverage`

**IMPORTANT**: For all testing-related tasks including writing tests, test automation, TDD workflow, and test coverage analysis, always delegate to the `test-automator` agent using the Task tool. This agent is specialized for comprehensive test suite creation with Vitest and React Testing Library following TDD methodology.

## Architecture

### Core Components

- **App.tsx**: Main application orchestrating assistants, sessions, and view modes (chat, edit, settings)
- **types.ts**: Core TypeScript interfaces defining Assistant, ChatSession, ChatMessage, and RagChunk
- **components/**: React components for UI (AssistantEditor, ChatWindow, Icons)
- **services/**: Business logic and external integrations

### Data Layer

- **services/db.ts**: Database operations interface for assistants and chat sessions
- **services/tursoService.ts**: Turso DB integration for cloud-based data persistence
- **services/geminiService.ts**: Google Gemini AI integration with streaming chat capabilities
- **services/embeddingService.ts**: Vector embeddings for RAG functionality using HuggingFace transformers
- **services/fileProcessingService.ts**: Document processing for PDF, DOCX, and MD files
- **services/sharingService.ts**: Assistant sharing with secure links and QR code generation

### Key Architecture Patterns

- **State Management**: React hooks with local component state, no external state library
- **Data Flow**: Turso DB → React state → UI components
- **AI Integration**: Streaming responses from Gemini with token counting and chat history truncation (max 20 messages)
- **RAG Implementation**: Multi-format file processing (PDF/DOCX/MD), text chunking, vector embeddings, and similarity search for context injection
- **Sharing System**: Secure assistant sharing with QR codes and public/private link management
- **Performance**: Preloaded embedding models and optimized UI rendering

### Environment Configuration

- **GEMINI_API_KEY**: Required in `.env.local` for AI functionality (user-configurable in UI)
- **TURSO_DATABASE_URL**: Turso database connection URL
- **TURSO_AUTH_TOKEN**: Turso database authentication token
- **Vite Configuration**: Exposes environment variables for both development and production

### Data Models

- **Assistant**: Has id, name, description, systemPrompt, ragChunks (for context), isShared and timestamps
- **ChatSession**: Belongs to assistant, contains message history, token counts, and session metadata
- **RagChunk**: Enhanced with file metadata, chunk type, and vector embeddings
- **ShareData**: Public sharing configuration with QR codes and access controls
- **RAG Integration**: Multi-format file processing, intelligent chunking, vectorization, and contextual retrieval

### Build System

- **TypeScript + React**: Modern React 19.1.1 with TypeScript, using Vite for bundling
- **Dependencies**: Core libraries include @google/genai, @huggingface/transformers, @libsql/client, mammoth, pdfjs-dist, qrcode
- **Development Tools**: ESLint, Prettier, Vitest, Husky, lint-staged for quality assurance
- **Database Scripts**: Custom Turso DB management and migration scripts
- **Path Aliases**: `@/*` maps to project root for imports

## Testing Structure

### Test Files Location

- Service tests: `services/*.test.ts`
- Component tests: `components/*.test.tsx`
- Test utilities: `src/test/`

### Test Conventions

- Use Vitest for unit and integration testing
- Use @testing-library/react for component testing
- Mock external APIs and IndexedDB in tests
- Follow AAA pattern: Arrange, Act, Assert
- Test file naming: `*.test.{ts,tsx}` or `*.spec.{ts,tsx}`

### Quality Gates

- All code must pass ESLint checks
- All code must be formatted with Prettier
- TypeScript compilation must succeed
- All tests must pass
- Pre-commit hooks enforce these standards

**IMPORTANT**: For all linting, syntax checking, type checking, and code quality issues, always delegate to the `linting-specialist` agent using the Task tool. This agent is specialized for automated detection and resolution of ESLint violations, TypeScript errors, formatting issues, and code quality problems.

## Important Notes

- The app requires a valid Gemini API key to function for AI features (configurable in UI)
- Data persists in Turso DB for cross-device synchronization
- Assistant sharing uses secure public links with QR codes
- RAG supports PDF, DOCX, and MD file processing with intelligent chunking
- Chat sessions have token counting and automatic history truncation
- This project uses pnpm as the package manager
- Pre-commit hooks automatically lint and format code before commits
- Mobile-responsive design optimized for all device sizes
- Performance optimizations include preloaded embedding models

### Recent Architectural Changes

- **Database Migration**: Moved from IndexedDB + Google Sheets to Turso DB for better scalability
- **Enhanced File Processing**: Added support for PDF and DOCX documents alongside markdown
- **Improved Sharing**: Dedicated sharing modal with QR code generation
- **UI/UX Overhaul**: Complete redesign with mobile responsiveness and custom styling
- **Development Infrastructure**: Added comprehensive linting, testing, and formatting pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/killkli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
