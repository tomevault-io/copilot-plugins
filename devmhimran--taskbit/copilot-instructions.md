## taskbit

> This is a Next.js application using a Role-Based Access Control (RBAC) system organized via Route Groups.

# Project Context & AI Rules: Antigravity Agent

## 🏗️ Architecture Overview
This is a Next.js application using a Role-Based Access Control (RBAC) system organized via Route Groups.

- **(public):** Sign-in and Forgot Password.
- **(private):** Layout-driven access for `(admin)`, `(client)`, and `(user)`.
    - **Roles (Admin):** `ADMIN`, `CO_ADMIN`.
    - **Roles (User):** `USER`, `EMPLOYEE`.

## 📁 Directory Mapping
- `app/`: Contains the route groups.
- `components/ui/`: Raw Shadcn UI components.
- `components/forms/`: Centralized form logic.
- `components/filters/`: Reusable filter components.
- `components/skeletons/`: Loading states.
- `hooks/`: Tanstack Query hooks (GET, POST, PATCH, DELETE).
- `server/`: Next.js Server Actions (Internal API logic).
- `lib/`: Email logic, Error Handlers, and React Query config.
- `utils/`: Pure utility functions.

## 📡 Data Patterns (Strict)
### Server Action Response Format
All data fetching actions must return this structure:
{
  "data": "values",
  "meta": {
    "count": "number",
    "page": "number",
    "limit": "number",
    "totalPages": "number"
  }
}

### Data Fetching Rule
- Use **Tanstack Query** via the `hooks/` folder.
- Do not call Server Actions directly in components; always wrap them in a hook for state management (loading/error).

## 🛠️ Admin Module Specifications
When working in `app/(private)/(admin)`, follow these page logic rules:
- **Dashboard:** Must include Payment Overview, Expenses, and "Business Health Alert".
- **Tasks:** Card-based view. Filters: Payment Status, Task Status, Paper Type, Delivery Date, Create Date. Must support Export.
- **Expenses/Salaries:** Table-based view. Statistics must be displayed at the top. Filtering requires Month/Year and specific Date support.
- **Management (Users/Clients):** Table view with Search + Role/Status filters.

## 🤖 Interaction Rules for Agent
1. **Modification Rule:** If modifying a Shadcn component, do NOT edit it in `ui/`. Move it to the general `components/` folder first.
2. **Naming Convention:** Use PascalCase for components and kebab-case for utility files.
3. **Skeleton Rule:** Every new page in `(private)` must have a corresponding skeleton in `components/skeletons/`.
4. **Error Handling:** Use the global error handler in `lib/error-handler.ts` for all server actions.

---
> Source: [devmhimran/taskbit](https://github.com/devmhimran/taskbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
