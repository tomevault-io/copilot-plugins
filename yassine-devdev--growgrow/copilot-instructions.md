## growgrow

> "instruction": "Rewrite every file in this full-stack project—frontend and backend—to be fully executable in a real-world production environment. Eliminate all use of null, undefined, any, unknown, void, scaffold, placeholder, mock, or incomplete constructs. Ensure all modules, imports, functions, and folders are defined, real, and operational. The goal is not just to remove errors, but to ensure the entire system runs with complete logic, secure flows, and production-grade architecture.",

{
  "instruction": "Rewrite every file in this full-stack project—frontend and backend—to be fully executable in a real-world production environment. Eliminate all use of null, undefined, any, unknown, void, scaffold, placeholder, mock, or incomplete constructs. Ensure all modules, imports, functions, and folders are defined, real, and operational. The goal is not just to remove errors, but to ensure the entire system runs with complete logic, secure flows, and production-grade architecture.",
  "goal": "Transform the entire codebase into a production-ready system with real authentication, secure password handling, complete logout flows, consistent TypeScript configurations, and fully documented modules. Resolve all known issues and ensure the system is deployable without ceremony or scaffolding.",
  "issuesToResolve": [
    "Replace mock authentication with a secure, real-world auth system (e.g., JWT, OAuth, or session-based)",
    "Implement password hashing and validation using a secure library like bcrypt",
    "Complete logout functionality with proper token/session invalidation",
    "Fix all TypeScript type errors and enforce strict typing across the codebase",
    "Remove hardcoded URLs and replace with environment-based config variables",
    "Increase backend test coverage with real unit and integration tests",
    "Unify TypeScript configurations across frontend and backend",
    "Remove redundant dependencies and optimize package.json",
    "Add clear, complete documentation for all modules, services, and components"
  ],
  "projectScope": {
    "frontend": {
      "framework": "React + Vite",
      "language": "TypeScript",
      "goals": [
        "Real authentication UI with secure form validation",
        "No dummy components or placeholder screens",
        "Environment-based API calls with fallback logic",
        "Consistent styling and layout across all pages"
      ]
    },
    "backend": {
      "framework": "Node.js + Express + Prisma",
      "language": "TypeScript",
      "goals": [
        "Secure API endpoints with real auth middleware",
        "Validated request/response schemas",
        "Centralized config and model routing",
        "Real database models and seed logic"
      ]
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yassine-devdev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yassine-devdev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
