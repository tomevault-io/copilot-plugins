## boobies

> > 💻 Development Environment: **Windows 11**


# Always start your response with fire emoji
# Project Knowledge and Interaction Rules

> 💻 Development Environment: **Windows 11**

## Project Overview

### Project Structure

- `client/`: Contains the **website** and **UI components**.
  - **Tech Stack**: Built with **Next.js**, **pnpm**, **TypeScript**, and **Tailwind CSS**.
- `server/`: Contains the **Node.js + Express** server responsible for handling requests from the client.
  - **Tech Stack**: Built with **Node.js**, **Express**, and **Javascript**.

## Client Rules

- When creating a **new main component**, save it under:
  ```
  client/components/[ComponentName]/[ComponentName].tsx
  ```
- Place **all child components** related to that component in the **same folder**.
- Follow a modular file structure to keep components organized, scalable, and easy to maintain.

## Server Rules

1. **File Structure & Modularity**

   - Organize code into clear folders: `routes/`, `controllers/`, `services/`, `middlewares/`, `utils/`, `interfaces` and `types/`.
   - Each file/module should follow **single responsibility** — avoid bloated logic in a single file.

2. **Routing Best Practices**

   - Group related routes under a dedicated file in `routes/`, and use an `index.js` to compose them.
   - Keep route files clean — delegate business logic to controllers or services.

3. **Controller and Service Separation**

   - Controllers should handle request/response logic.
   - Services should handle the business logic and can be reused across routes.

4. **Type Safety**

   - Define and use **custom TypeScript interfaces/types** for request bodies, responses, and data models.
   - Avoid using `any` unless absolutely necessary (and document it when you do).

5. **Middleware**

   - Use middlewares for authentication, validation, logging, and error handling.
   - Keep them modular and reusable.

6. **Error Handling**

   - Use a centralized error handler middleware.
   - Throw standardized error objects and provide meaningful error messages.
   - Avoid exposing internal stack traces to the client.

7. **Validation**

   - Validate incoming data using tools like `zod` or `Joi`, ideally in middleware before it reaches the controller.

8. **Environment Configuration**

   - Use `.env` files for environment variables.
   - Use a library like `dotenv` to load them and a `config/` folder to manage app settings cleanly.

9. **Logging & Debugging**

   - Use a logger (e.g. `winston`, `pino`) instead of `console.log` for structured logging.
   - Include request/response metadata when helpful for debugging.

10. **Code Style & Linting**
    - Follow consistent code style using tools like `ESLint` and `Prettier`.
    - Use strict TypeScript settings for better safety and code clarity.

## Agent Interaction Guidelines

1. **Familiarize with Codebase**

   - Always review the current project structure and existing code before suggesting or implementing changes.

2. **Leverage Existing Functionality**

   - Before creating new components, methods, or features, check if similar functionality already exists and can be extended or reused.

3. **Promote Reusability & Consistency**

   - Use reusable components, utility functions, and shared types/interfaces wherever possible to maintain efficiency and coherence.

4. **Follow Modular Code Practices**

   - Keep files **small and focused**.
   - Break down logic into **separate components**, **types**, and **interfaces**.
   - Ensure **one concern per file** when possible to improve readability and maintainability.

5. **Design with Purpose**

   - Follow modern UI/UX design principles: clean layouts, intuitive interactions, responsiveness, and accessibility.
   - Maintain a consistent and sleek visual style across the entire extension.

6. **Prioritize Performance & Scalability**
   - Consider browser performance and extension constraints.
   - Optimize for fast load times and minimal memory usage.

## Improvement Focus

- Continuously suggest **UI/UX improvements** in line with current trends and best practices.
- Advocate for **clean, modular, maintainable, and well-documented code**.

## Communication Protocol

- Be **concise, clear, and actionable** in all responses.
- If a task is ambiguous, **ask clarifying questions** before proceeding.
- When proposing ideas or changes, **briefly explain the rationale** behind them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MotiTheWizerd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
