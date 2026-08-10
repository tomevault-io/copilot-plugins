## 06-development-guide

> This document provides guidelines for developing and contributing to the Mkdirs template.

# Development Guide

This document provides guidelines for developing and contributing to the Mkdirs template.

## Development Setup

1. Clone the repository
2. Install dependencies with `pnpm install`
3. Set up environment variables (copy `.env.example` to `.env`)
4. Run the development server with `pnpm dev`

## Environment Variables

Key environment variables needed for development:

- **NEXT_PUBLIC_SITE_URL**: Base URL of the site
- **NEXT_PUBLIC_SANITY_PROJECT_ID**: Sanity project ID
- **NEXT_PUBLIC_SANITY_DATASET**: Sanity dataset name
- **SANITY_API_TOKEN**: Sanity API token
- **NEXTAUTH_SECRET**: Secret for NextAuth
- **STRIPE_SECRET_KEY**: Stripe API secret key

## Code Style and Linting

The project uses Biome for code formatting and linting:

- **biome.json**: Biome configuration
- Run `pnpm lint` to check for code style issues
- Run `pnpm format` to format the codebase

## Scripts

Utility scripts for common tasks:

- **scripts**: Directory containing utility scripts
- **Batch Operations**: Scripts for batch operations on items
- **Email Export**: Tools for exporting user emails
- **Microlink Integration**: Scripts for working with microlink

## Build and Deployment

Steps for building and deploying the application:

1. Run `pnpm build` to create a production build
2. Verify the build with `pnpm start`
3. Deploy to your hosting platform of choice

### Docker Deployment

For Docker-based deployment:

- **Dockerfile**: Docker configuration
- **next.config.docker.mjs**: Next.js configuration for Docker

---
> Source: [MkThingsHQ/mkdirs](https://github.com/MkThingsHQ/mkdirs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
