## theeasyapp

> This is a React project bootstrapped with `bun-react-tailwind-shadcn-template`. It uses [Bun](https://bun.sh/) as the JavaScript runtime, package manager, and bundler. The frontend is built with [React](https://react.dev/) and styled with [Tailwind CSS](https://tailwindcss.com/) and [shadcn/ui](https://ui.shadcn.com/).

## Project Overview

This is a React project bootstrapped with `bun-react-tailwind-shadcn-template`. It uses [Bun](https://bun.sh/) as the JavaScript runtime, package manager, and bundler. The frontend is built with [React](https://react.dev/) and styled with [Tailwind CSS](https://tailwindcss.com/) and [shadcn/ui](https://ui.shadcn.com/).

The project includes a simple backend API created with Bun's built-in `serve` function. The API has a few example endpoints in `src/index.tsx`.

The frontend application is rendered from `src/frontend.tsx`, which mounts the main `App` component defined in `src/App.tsx`.

## Building and Running

### Development

To start the development server with hot reloading:

```bash
bun dev
```

The server will be available at `http://localhost:3000`.

### Production

To build the project for production:

```bash
bun build
```

This will create a `dist` directory with the optimized production assets.

To run the production server:

```bash
bun start
```

## Development Conventions

- **Package Management:** This project uses [Bun](https://bun.sh/) for package management. Use `bun install` to install dependencies.
- **Styling:** [Tailwind CSS](https://tailwindcss.com/) is used for styling. Utility classes are preferred.
- **Components:** [shadcn/ui](https://ui.shadcn.com/) components are used. These are unstyled components that can be customized with Tailwind CSS.
- **API:** The backend API is defined in `src/index.tsx`. New routes can be added to the `routes` object.
- **TypeScript:** The project uses TypeScript with strict mode enabled. Path aliases are configured in `tsconfig.json` to allow for absolute imports from the `src` directory (e.g., `import { Button } from "@/components/ui/button";`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uwenayoallain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/uwenayoallain)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
