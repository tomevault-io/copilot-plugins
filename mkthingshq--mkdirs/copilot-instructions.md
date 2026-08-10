## 02-tech-stack

> This document outlines the key technologies used in the Mkdirs template and how they work together.

# Tech Stack and Architecture

This document outlines the key technologies used in the Mkdirs template and how they work together.

## Core Technologies

- **Next.js**: The React framework used for building the application with server-side rendering, API routes, and server components
- **TypeScript**: Used for type-safe code across the entire codebase
- **Sanity CMS**: Headless CMS for managing content
- **Tailwind CSS**: Utility-first CSS framework for styling
- **Radix UI**: Unstyled, accessible UI components
- **NextAuth**: Authentication solution for Next.js applications
- **Stripe**: Payment processing
- **React Email**: For creating and sending emails
- **React Hook Form**: Form handling
- **Zod**: Data validation

## File Structure

- **package.json**: Contains all dependencies and scripts
- **next.config.mjs**: Next.js configuration
- **tailwind.config.ts**: Tailwind CSS configuration
- **sanity.config.ts**: Sanity CMS configuration
- **src/app**: Next.js app router pages and layouts
- **src/components**: Reusable UI components
- **src/lib**: Utility functions and shared code
- **src/sanity**: Sanity schema definitions and utilities

## Key Architectural Patterns

- **App Router**: Uses Next.js App Router for routing
- **Server Components**: Leverages React Server Components for better performance
- **Server Actions**: Uses Next.js Server Actions for server-side data mutations
- **Content Management**: Sanity CMS for structured content
- **Authentication**: Secure authentication with NextAuth
- **API Routes**: Backend functionality exposed through API routes
- **Responsive Design**: Mobile-first approach with Tailwind CSS

## Build and Deployment

- **Development**: `pnpm dev` for local development
- **Build**: `pnpm build` for production builds
- **Docker Support**: Includes Dockerfile for containerization

---
> Source: [MkThingsHQ/mkdirs](https://github.com/MkThingsHQ/mkdirs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
