## pokemon-trainer-manager

> A React-based web application for managing Pokemon trainers, with Supabase as the backend database and Cloudflare R2 for image storage. The app integrates with the Pokemon API for additional functionality.

# Pokemon Trainer Manager (pmon)

## Project Overview
A React-based web application for managing Pokemon trainers, with Supabase as the backend database and Cloudflare R2 for image storage. The app integrates with the Pokemon API for additional functionality.

## Tech Stack
- **Frontend**: React 18.2.0 with TypeScript
- **Database**: Supabase Postgres
- **Object Storage**: Cloudflare R2
- **Styling**: Tailwind CSS (Mobile-First Design)
- **Package Manager**: pnpm
- **Language**: TypeScript 5.8.3

## Project Structure
```
src/
├── components/
│   ├── TrainerOverview.tsx   # Main trainer list view
│   ├── TrainerCard.tsx       # Individual trainer display
│   └── AddTrainerForm.tsx    # Form for adding new trainers
├── services/
│   └── trainerService.ts    # Trainer CRUD via Vercel API
├── api/
│   ├── trainers/            # Supabase CRUD endpoints
│   └── storage/             # Cloudflare R2 image endpoints
├── types/
│   └── trainer.ts           # TypeScript type definitions
├── App.tsx                  # Main app component
├── index.tsx                # React entry point
├── App.css                  # App-specific styles
└── index.css               # Global styles
```

## Current Features
- View all trainers in a grid layout
- Add new trainers with form validation
- Delete trainers with confirmation
- Store trainer data: name, age, location, favorite Pokemon type, description
- Full TypeScript support with type safety
- Mobile-first responsive design with Tailwind CSS
- Touch-friendly UI components

## Environment Setup Required
Configure these variables in `.env.local` for server-side API functions:
- R2_ACCOUNT_ID
- R2_ACCESS_KEY_ID
- R2_SECRET_ACCESS_KEY
- R2_BUCKET_NAME
- SUPABASE_URL
- SUPABASE_SERVICE_ROLE_KEY

## Trainer Data Model
```javascript
{
  name: string (required),
  age: number (optional),
  location: string (optional),
  favoriteType: string (Pokemon type),
  description: string (optional),
  createdAt: ISO string
}
```

## Available Scripts
- `pnpm start` - Runs the app in development mode
- `pnpm build` - Builds the app for production
- `pnpm test` - Launches the test runner

## Future Features (Planned)
- Integration with Pokemon API
- Trainer Pokemon team management
- Pokemon battle statistics
- Trainer profiles with avatars
- Search and filter functionality

## TypeScript Types
The project includes comprehensive TypeScript types in `src/types/trainer.ts`:
- `Trainer` interface for trainer data structure
- `TrainerFormData` interface for form handling
- Full type safety throughout the application

## Getting Started
1. Install dependencies: `pnpm install`
2. Configure `.env.local` with Supabase and R2 credentials
3. Start the development server: `pnpm start`
4. Open http://localhost:3000 to view the app

## Development Notes
- All components are written in TypeScript with proper typing
- Trainer data is persisted via Vercel API functions (Supabase + R2)
- Strict TypeScript configuration for better code quality
- Mobile-first responsive design approach
- Uses Tailwind CSS utility classes for styling
- Touch-friendly UI with 44px minimum button heights
- Responsive breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px)

## Styling Architecture
- **Mobile-First**: If something new is implemented it should be optimized for mobile. Deskopt is second place.
- **Tailwind CSS**: Utility-first CSS framework
- **Responsive Grid**: Adaptive layout for trainer cards
- **Custom Colors**: Extended Tailwind palette with primary, success, and danger colors
- **Typography Scale**: Responsive text sizes that adapt to screen size
- **Naming**: Components and Component file names should be CamelCase. function pascalCase.
- **No Smileys** Vermeide die Verwendung von Smileys für Buttons. Außer ausdrücklich erwähnt

## Claude Memories
- dont use smileys. only if specifically wished for

## Package Manager
**Important**: This project uses `pnpm` as the package manager. Always use `pnpm` commands:
- `pnpm install` - Install dependencies
- `pnpm start` - Start development server
- `pnpm build` - Build for production
- `pnpm test` - Run tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LamprechtRobin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
