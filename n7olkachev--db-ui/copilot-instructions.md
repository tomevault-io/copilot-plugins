## project-overview

> This project is a modern, user-friendly database management web interface built with Next.js, TypeScript, and shadcn/ui. It aims to provide a simpler and more elegant alternative to traditional database management tools like pgAdmin or phpmyadmin.


# Database Management UI Project Overview

## Description

This project is a modern, user-friendly database management web interface built with Next.js, TypeScript, and shadcn/ui. It aims to provide a simpler and more elegant alternative to traditional database management tools like pgAdmin or phpmyadmin.

## Tech Stack

- **Framework**: Next.js with App Router
- **Language**: TypeScript
- **UI Components**: shadcn/ui
- **Database**: PostgreSQL, MySQL and MSSQL must be supported
- **Styling**: Tailwind CSS

## Project Goals

1. **Simplicity**: Provide an intuitive interface for database operations
2. **Modern Design**: Clean, responsive UI with a focus on user experience
3. **Performance**: Efficient data handling and real-time updates
4. **Security**: Secure database connections and operations
5. **Developer Experience**: Type-safe development with TypeScript

## Key Features

- Database connection management
- Table browsing and editing
- Query execution interface
- Schema visualization
- Data export/import capabilities

## Development Guidelines

1. Follow TypeScript best practices and maintain type safety
2. Use server components for data fetching operations
3. Implement responsive design for all components
4. Write clean, maintainable code with proper documentation
5. Follow shadcn/ui design patterns for consistency

## Server Components Usage

Always prefer server components for data fetching operations. Use 'use server' directive and server components by default unless there's a specific reason to use client components (like interactivity or browser APIs).

### Examples

#### Good

```tsx
"use server";
// In a server component
async function getData() {
  const data = await db.query("SELECT * FROM users");
  return data;
}
```

#### Bad

```tsx
// In a client component
'use client';
useEffect(() => {
  fetch('/api/users').then(...);
}, []);
```

---
> Source: [n7olkachev/db-ui](https://github.com/n7olkachev/db-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
