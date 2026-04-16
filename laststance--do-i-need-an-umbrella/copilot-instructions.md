## do-i-need-an-umbrella

> - **Next.js (App Router)**: Framework for React applications

# Development Guide

## Technology Stack

- **Next.js (App Router)**: Framework for React applications
- **TypeScript**: For type-safe code
- **TailwindCSS**: For styling
- **React Server Components**: For optimized server-rendered React
- **API Routes**: For backend functionality

## Common Tasks

### Running the Application

```bash
# Development mode
pnpm dev

# Build for production
pnpm build

# Start production build
pnpm start
```

### Working with Weather API

The application uses external weather APIs which are wrapped by internal API routes in:
- **/app/api/weather/route.ts**: Fetches weather data
- **/app/api/geocode/route.ts**: Handles location conversion

### Adding New Pages

1. Create a new directory in the app folder
2. Add a page.tsx file for the main content
3. Use React Server Components for data fetching
4. Implement client-side interactivity with "use client" directive where needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laststance) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
