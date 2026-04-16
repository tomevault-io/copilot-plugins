## donamed-cliente-front

> **Donamed** is a medical platform designed to facilitate medication requests and donations. This repository contains the React-based frontend, which provides a modern, responsive interface for users to register, log in, manage their profiles, and submit/track medication requests.

# GEMINI.md - Donamed Frontend Project

## Project Overview
**Donamed** is a medical platform designed to facilitate medication requests and donations. This repository contains the React-based frontend, which provides a modern, responsive interface for users to register, log in, manage their profiles, and submit/track medication requests.

- **Primary Folder:** All source code and project configuration are located in the `lastest/` directory.
- **Main Technologies:** React 19, TypeScript, Vite, Tailwind CSS, Radix UI.
- **Routing:** React Router DOM v7 (Data Router).
- **API Client:** Axios with custom interceptors for authentication and token management.

## Tech Stack
- **Framework:** [React 19](https://react.dev/)
- **Build Tool:** [Vite](https://vitejs.dev/)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **Styling:** [Tailwind CSS](https://tailwindcss.com/)
- **UI Components:** [Radix UI](https://www.radix-ui.com/)
- **Icons:** [Lucide React](https://lucide.dev/)
- **HTTP Client:** [Axios](https://axios-http.com/)
- **State Management:** Custom React Hooks and local state.

## Getting Started

### Prerequisites
- Node.js (Latest LTS recommended)
- npm or yarn

### Installation
Navigate to the `lastest/` directory and install dependencies:
```bash
cd lastest
npm install
```

### Development
Run the development server:
```bash
npm run dev
```

### Build
Generate a production-ready build:
```bash
npm run build
```

### Linting
Check for code style and quality issues:
```bash
npm run lint
```

## Project Structure (`lastest/src/`)
- `api/`: Axios client configuration and interceptors.
- `components/`: Reusable UI components.
    - `ui/`: Base UI primitives (buttons, inputs, modals, etc.).
    - `landing/`: Components specific to the landing page.
- `config/`: Application configuration, including API endpoints.
- `hooks/`: Custom React hooks for business logic (e.g., `useAuth`, `useSolicitudes`).
- `Pages/`: Main view components organized by feature:
    - `Auth/`: Login, Register, Password Recovery.
    - `Ayuda/`: FAQ and Contact.
    - `Consultas/`: Inquiry management.
    - `Landing/`: Main landing page.
    - `MiPerfil/`: User dashboard and profile settings.
    - `Solicitudes/`: Medication request forms and history.
- `services/`: API service layers handling data fetching and mutations.
- `types/`: TypeScript interfaces and type definitions.
- `utils/`: Common utility functions and validators.

## Development Conventions
- **Naming:** Use PascalCase for components and camelCase for hooks, services, and utilities.
- **Services:** All API calls should be encapsulated in the `services/` directory and used via custom hooks in the `hooks/` directory.
- **Authentication:** Authentication tokens are stored in `localStorage`. The `apiClient` automatically attaches the `Authorization` header to requests when a token is present.
- **Styling:** Use Tailwind CSS utility classes. Avoid custom CSS files unless absolutely necessary (use `index.css` for global styles).
- **Components:** Prefer functional components and hooks. Use Radix UI primitives for accessible UI patterns.
- **API Base URL:** Configurable via the `VITE_API_URL` environment variable (defaults to `http://localhost:3000`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guelo0987) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
