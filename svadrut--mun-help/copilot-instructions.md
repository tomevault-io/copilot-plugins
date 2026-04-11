## mun-help

> This project is a Next.js application that provides a platform for Model United Nations (MUN) training. It allows admins to create lessons with rich text content and grading guidelines, and students to view these lessons and complete associated activities. The application uses a PostgreSQL database with Drizzle ORM for data persistence, Clerk for authentication, and Tailwind CSS for styling.

# GEMINI.md

## Project Overview

This project is a Next.js application that provides a platform for Model United Nations (MUN) training. It allows admins to create lessons with rich text content and grading guidelines, and students to view these lessons and complete associated activities. The application uses a PostgreSQL database with Drizzle ORM for data persistence, Clerk for authentication, and Tailwind CSS for styling.

The data model is well-defined and includes tables for schools, users, memberships (with roles like "admin" and "student"), lessons, submissions, and grades. This structure supports a multi-tenant setup where each school has its own set of users and content.

The application uses ProseMirror for rich text editing, and the lesson content is stored as JSON. A custom renderer converts this JSON to HTML for display, and it supports a slide-based presentation format using a `<!-- Slide -->` delimiter.

## Building and Running

### Prerequisites

- Node.js and npm (or yarn/pnpm/bun)
- A PostgreSQL database

### Installation

1.  Clone the repository.
2.  Install the dependencies:
    ```bash
    npm install
    ```
3.  Create a `.env` file in the root of the project and add the following environment variables:
    ```
    DATABASE_URL="your-postgresql-database-url"
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="your-clerk-publishable-key"
    CLERK_SECRET_KEY="your-clerk-secret-key"
    ```

### Running the Application

To run the application in development mode, use the following command:

```bash
npm run dev
```

This will start the development server at `http://localhost:3000`.

### Building for Production

To build the application for production, use the following command:

```bash
npm run build
```

This will create an optimized build of the application in the `.next` directory.

### Running in Production

To run the application in production, use the following command:

```bash
npm run start
```

This will start the production server.

## Development Conventions

### Code Style

The project uses ESLint for linting. To run the linter, use the following command:

```bash
npm run lint
```

The project also uses Prettier for code formatting, which is likely integrated with the development environment.

### Database Migrations

The project uses Drizzle Kit for database migrations. To generate a new migration, you can use a command like this:

```bash
npx drizzle-kit generate
```

To apply migrations, you can use a command like this:

```bash
npx drizzle-kit migrate
```
**Note:** The exact commands for drizzle-kit may vary based on the project's setup.

### Testing

There are no explicit testing practices evident from the file structure. It's recommended to add a testing framework like Jest or Vitest to the project.

### Contribution Guidelines

There are no explicit contribution guidelines in the repository. It's recommended to add a `CONTRIBUTING.md` file to document the process for contributing to the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Svadrut)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Svadrut)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
