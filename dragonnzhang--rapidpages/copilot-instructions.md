## rapidpages

> This repository contains the source code for **Rapidpages**, a "prompt-first" IDE for building user interfaces. It allows users to describe a desired UI component in natural language, and the application uses AI to generate the corresponding code.

# Gemini Project Context: Rapidpages

## Project Overview

This repository contains the source code for **Rapidpages**, a "prompt-first" IDE for building user interfaces. It allows users to describe a desired UI component in natural language, and the application uses AI to generate the corresponding code.

The generated code is currently for **React** with **Tailwind CSS**.

The project is a full-stack web application built with a modern tech stack, largely following the principles of the T3 Stack.

- **Framework**: [Next.js](https://nextjs.org/) (React framework)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **API**: [tRPC](https://trpc.io/) for type-safe client-server communication.
- **Database/ORM**: [Prisma](https://www.prisma.io/) for database access and schema management.
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) for utility-first CSS.
- **AI**: The AI generation is powered by models from OpenAI, Anthropic, DeepSeek, and Google, using the [Vercel AI SDK](https://sdk.vercel.ai/).

The application's architecture consists of a Next.js frontend (`src/pages`, `src/components`) and a tRPC backend (`src/server/api`). The core AI logic for code generation resides in the tRPC routers.

## Building and Running

### 1. Initial Setup

**Install dependencies:**
```bash
npm install
```

**Set up environment variables:**
Copy the example environment file and fill in the required values.
```bash
cp .env.example .env
```
You will need to provide credentials for:
- **GitHub OAuth**: `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` for user authentication.
- **OpenAI API Key**: `OPENAI_API_KEY` for the AI-powered generation features.
- **Database URL**: `DATABASE_URL` for connecting to your PostgreSQL database.

### 2. Database

Push the Prisma schema to your database. This will create the necessary tables.
```bash
npm run db:push
```

### 3. Running the Application

**For development:**
This command starts the Next.js development server on `http://localhost:3000`.
```bash
npm run dev
```

**For production:**
First, build the application:
```bash
npm run build
```
Then, start the production server:
```bash
npm run start
```

## Development Conventions

### Code Style & Linting

- **Formatting**: Code is formatted with [Prettier](https://prettier.io/).
- **Linting**: [ESLint](https://eslint.org/) is used for static analysis. The configuration is in `.eslintrc.cjs`.
- **Run Linter**: Check for any linting issues with:
  ```bash
  npm run lint
  ```
- A GitHub Actions workflow in `.github/workflows/lint.yml` automatically runs the linter on pushes and pull requests to the `main` branch.

### API Development (tRPC)

- The API is built with tRPC, providing end-to-end type safety.
- The main API router is located at `src/server/api/root.ts`.
- Individual routers for different concerns (e.g., `ai`, `component`, `user`) are in `src/server/api/routers/`.
- When adding a new set of related API endpoints, create a new router file in the `routers` directory and add it to `appRouter` in `root.ts`.

### Database (Prisma)

- The database schema is defined in `prisma/schema.prisma`.
- After modifying the schema, run `npm run db:push` to apply the changes to the database. For more complex changes, you might need to create a migration.

### Path Aliases

- The project uses the `~/*` path alias for the `src/*` directory, configured in `tsconfig.json`. Use this for cleaner import paths (e.g., `import { api } from '~/utils/api';`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DragonnZhang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DragonnZhang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
