## t2-sportlink-web-backend

> This document outlines the development standards focused solely on backend development, incorporating Supabase integration and TypeScript usage.

# 🚀 Sportlink Development Standards (SCDS) – Backend (TypeScript & Supabase)

This document outlines the development standards focused solely on backend development, incorporating Supabase integration and TypeScript usage.

---

## 📌 Folder & File Naming Conventions

1. **All folder names must be in lowercase and kebab-case**  
   Example: `controllers`, `middleware`, `services`, `routes`
2. **All file names for components such as models and controllers must use PascalCase**  
   Example: `User.ts`, `ProductController.ts`
3. **API route files should be named in camelCase**  
   Example: `authRoutes.ts`, `productRoutes.ts`
4. **Utility/helper function names should be in camelCase**  
   Example: `generateToken.ts`, `hashPassword.ts`
5. **Environment variables in the `.env` file must be defined in UPPER_CASE**  
   Example: `JWT_SECRET`, `SUPABASE_URL`, `SUPABASE_KEY`

---

## 📌 Backend Coding Standards

1. **TypeScript Usage:**  
   Use TypeScript throughout the codebase to ensure static type checking and a more maintainable, readable structure.

2. **ES Modules:**  
   The `import` syntax should be used instead of `require`.

3. **Error Handling:**  
   All asynchronous functions must implement `try/catch` blocks to handle errors properly.

4. **Layered Architecture:**  
   Database operations should not be performed directly within route handlers. Instead, use controller and service layers.



5. **Database Integration:**  
   - Supabase should be used instead of MongoDB.
   - Utilize Supabase’s TypeScript-compatible SDK to manage database connections and queries.

---

These standards are designed to create a consistent, secure, and maintainable backend development process. By integrating Supabase and utilizing TypeScript, this approach aligns with modern application development trends while excluding frontend-related rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Divizyon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
