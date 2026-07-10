## wallmart-clone

> This is a [Next.js](https://nextjs.org/) project, bootstrapped with `create-next-app`. It appears to be a web application, likely a clone of Walmart's website given the project name "wallmart-clone".

# GEMINI.md

## Project Overview

This is a [Next.js](https://nextjs.org/) project, bootstrapped with `create-next-app`. It appears to be a web application, likely a clone of Walmart's website given the project name "wallmart-clone".

The project uses:
- **Framework:** Next.js 16
- **Language:** TypeScript
- **UI Library:** React 19
- **Styling:** Tailwind CSS
- **Linting/Formatting:** Biome

The application structure follows the Next.js App Router paradigm, with the main layout defined in `app/layout.tsx` and the main page in `app/page.tsx`. The project also utilizes the Geist font.

## Building and Running

You can use `pnpm` (as indicated by `pnpm-lock.yaml`), `npm`, or `yarn` to manage dependencies and run scripts.

### Development

To run the development server:

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to view the application.

### Building for Production

To create a production build:

```bash
pnpm build
```

### Running in Production

To start the production server:

```bash
pnpm start
```

## Development Conventions

### Linting and Formatting

This project uses [Biome](https://biomejs.dev/) for linting and formatting.

- To check for linting errors, run:
  ```bash
  pnpm lint
  ```
- To format the code, run:
  ```bash
  pnpm format
  ```

The configuration in `biome.json` specifies a 2-space indent style and includes recommended rules for React and Next.js.

### TypeScript

The project is configured with a strict TypeScript setup. Path aliases are configured to allow imports starting with `@/*`, which maps to the project's root directory.

---
> Source: [pestov-v/wallmart-clone](https://github.com/pestov-v/wallmart-clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
