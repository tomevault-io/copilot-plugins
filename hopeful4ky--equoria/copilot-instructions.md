## equoria

> Defines the core frameworks used in the Equoria project, including integration patterns and implementation structure.


## 🧩 React Native

- React Native is used to build Equoria’s cross-platform mobile app for iOS and Android.
- All components are written in JSX using function components and React Hooks.
- UI elements are styled using Tailwind CSS via compatible libraries like `tailwind-rn` or `nativewind`.
- React Navigation is used to manage tabbed layout and in-app screen transitions.

**File Structure Convention:**
```
frontend/
├── components/
├── screens/
├── navigation/
└── App.js
```

> React Native components are organized by view (e.g., StableScreen, HorseProfile, Marketplace).

---

## 🧩 Express.js

- Express.js is used as the backend web framework, running atop Node.js.
- Responsible for routing API calls and connecting to PostgreSQL.
- Implements RESTful endpoints under `/api/`
- Utilizes middleware for:
  - Authentication (JWT)
  - Error handling
  - Rate limiting
  - Request validation
- Responds with JSON to be consumed by React Native frontend.

**Recommended Folder Structure:**
```
backend/
├── controllers/
├── routes/
├── middleware/
├── services/
└── app.js
```

---

## 🧩 Tailwind CSS

- Utility-first CSS framework used throughout the frontend.
- Used with compatible solutions for React Native like `nativewind`, `tailwind-rn`, or custom implementations.
- Encouraged pattern: layout components (wrappers/containers) + styled primitives (buttons, cards, text blocks)
- Styling is managed in `tailwind.config.js` with mobile-first design breakpoints.

> Tailwind allows for rapid, consistent visual development across all views.

---

## 🧩 Node.js

- Core runtime for all backend services.
- Runs the Express.js app server and manages connections to PostgreSQL.
- Enables filesystem access, server logic, route middleware, and CLI scripts.
- Supports CommonJS or ES module syntax (`require` or `import`) depending on configuration.

> Node serves as the backbone for data flow between mobile client and database.

---

## 🧩 PostgreSQL

- Primary relational database used for all persistent game data.
- Advanced use of `JSONB` fields to store flexible, structured data for:
  - Horse genetics
  - Breed profiles
  - Dynamic rider traits
- Managed and queried via `pg` Node.js module or query builders (optionally with Prisma).

**Conventions:**
- Database schema lives in `/schema.sql`
- Test database configured with `.env.test`

> Indexed, relational architecture ensures scalable and performant gameplay logic.

---

## 🧩 GitHub Actions (CI/CD)

- Handles continuous integration and testing pipeline.
- All commits and PRs must trigger Jest tests and integration tests via GitHub Actions.
- Secure secrets stored in GitHub repository settings (`.env.test`)
- Future staging or deployment workflows will be added for mobile builds or backend updates.

**Example:**
```
name: Run Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hopeful4ky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hopeful4ky)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
