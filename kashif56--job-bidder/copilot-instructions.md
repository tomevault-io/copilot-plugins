## job-bidder

> This document outlines coding rules and conventions to follow when contributing to the **Proposly** project (Django REST + React + Vite + Tailwind + Redux Toolkit + OpenAI).


# 🧠 Project Coding Rules & Best Practices

This document outlines coding rules and conventions to follow when contributing to the **Proposly** project (Django REST + React + Vite + Tailwind + Redux Toolkit + OpenAI).

---

## Build it as if someone else will scale it, and someone else will debug it

## 🚀 General Principles

- ✅ Prioritize **clarity** over cleverness.
- ✅ Write **modular**, reusable, and DRY (Don’t Repeat Yourself) code.
- ✅ Document assumptions in code comments.
- ✅ Use meaningful naming conventions.
- ✅ Follow RESTful API principles and frontend state separation.

---

## 🌐 Frontend (React + Vite + Tailwind)

### 🔧 Configuration

- ✅ **Set base URL in `vite.config.js`** so API base doesn't need repeating in every request:

```js
// vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': 'http://localhost:8000',
    },
  },
})


✅ Use .env for environment-specific values (API URL, Stripe key, etc.)

📦 Structure
/src
  /api             → Axios services & API logic
  /components      → Reusable UI components
  /pages           → Page-level components
  /redux           → Toolkit slices for auth, proposals, etc.
  /utils           → Helper functions (formatters, validators)
  /hooks           → Custom hooks (e.g. useAuth, useFetch)
  App.jsx
  main.jsx

⚙️ Best Practices
✅ Use Axios instance with interceptors for auth tokens.

✅ State management with Redux Toolkit: slices > reducers > store.

✅ Style with Tailwind CSS using utility-first classes.

✅ Use framer-motion for animation transitions (form, modal, etc.).

✅ Use React Icons only where relevant for better performance.

✅ Lazy-load pages and




##Django

/backend
  /users           → Auth, profile, registration
  /proposals       → Proposal generation, history
  /billing         → Stripe integration
  /core            → Settings, middleware, utilities
  /utils           → Reusable logic (prompt templates, credit system)



🔐 3. Authentication Best Practices
🔸 Frontend
Store JWT in localStorage

Use interceptors for refreshing tokens (if using access/refresh token flow).

Redirect unauthenticated users to login automatically.

🔸 Backend
Use SimpleJWT with access and refresh tokens.

Protect endpoints with @permission_classes([IsAuthenticated]).


✨ 4. OpenAI Integration
Use a central utility (e.g., openai_service.py) to call OpenAI API.

Keep prompts reusable with templating (Jinja2 or Python .format()).

Handle API errors gracefully and retry once on timeout/overload.


🎨 5. Tailwind CSS & React Best Practices
Keep styling in Tailwind classes directly in JSX (avoid custom CSS unless needed).

Use semantic HTML: <section>, <article>, <main>, etc.

Break UI into reusable components: <Button>, <Card>, <Input>, etc.

Use icons via react-icons (e.g., FiCopy, FiTrash).




## 7. Backend Best Practices

Separate serializers, views, and models clearly.

Write reusable permissions and pagination classes.

Split business logic into service layers or utils (avoid bloated views).

Use signals for post-save credit assignment, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kashif56) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
