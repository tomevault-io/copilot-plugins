## fortressos

> - Use English for all code and documentation

# General Code Style & Formatting
- Use English for all code and documentation
- Adhere to the Airbnb JavaScript and React/JSX Style Guide
- Use PascalCase for React component file names (e.g., UserCard.tsx)
- Prefer named exports for components
- Declare types for all variables and functions (parameters and return values)
- Avoid using 'any'; define necessary types instead
- Document public classes and methods using JSDoc
- Avoid blank lines within functions
- Limit to one export per file
- Utilize Prettier for consistent code formatting
- Configure ESLint with Airbnb rules and integrate with Prettier

# Naming Conventions
- Use PascalCase for classes
- Use camelCase for variables, functions, and methods
- Use kebab-case for file and directory names
- Use UPPERCASE for environment variables
- Avoid magic numbers; define constants instead

# Project Structure & Architecture
- Follow Next.js App Router conventions
- Distinguish between server and client components appropriately
- Organize components within the 'app/' directory
- Implement dynamic routing using file-based routing

# Styling & UI
- Employ Tailwind CSS for styling
- Utilize Shadcn UI components for consistent design
- Ensure responsive design and accessibility compliance

# Data Fetching & Forms
- Use TanStack Query (react-query) for frontend data fetching
- Implement React Hook Form for form handling
- Apply Zod for schema validation

# State Management & Logic
- Manage global state using React Context
- Avoid external state management libraries unless necessary

# Functions & Logic
- Keep functions short and single-purpose (less than 20 lines)
- Avoid deeply nested blocks by:
  - Using early returns
  - Extracting logic into utility functions
- Use higher-order functions (map, filter, reduce) to simplify logic
- Use arrow functions for simple cases (less than 3 instructions); use named functions otherwise
- Use default parameter values instead of null/undefined checks
- Use RO-RO (Receive Object, Return Object) for functions with multiple parameters

# Data Handling
- Encapsulate data in composite types; avoid excessive use of primitive types
- Avoid placing validation inside functions; use classes with internal validation instead
- Prefer immutability for data:
  - Use 'readonly' for immutable properties
  - Use 'as const' for literals that never change

# Backend & Database
- Access the database using Prisma ORM
- Define and manage the database schema with Prisma's 'schema.prisma' file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Olaw2jr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
