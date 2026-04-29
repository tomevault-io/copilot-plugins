## drinkweise

> Development workflow rules for the project.


# Development Workflow

## Available Scripts

From [package.json](mdc:package.json):

### Development

- `yarn start`: Start Expo development server
- `yarn ios`: Run on iOS simulator
- `yarn android`: Run on Android emulator
- `yarn web`: Run on web browser

### Building

- `yarn build:dev`: Build development version
- `yarn build:preview`: Build preview version
- `yarn build:prod`: Build production version
- `yarn prebuild`: Run Expo prebuild

### Code Quality

- `yarn lint`: Check linting and formatting
- `yarn format`: Auto-fix linting and formatting
- `yarn type:check`: TypeScript type checking
- `yarn test`: Run tests in watch mode

### Supabase

- `yarn supabase:gen:types`: Generate TypeScript types
- `yarn supabase:gen:migrate`: Create new migration

### Security

- `yarn audit:ci`: Security audit

## Development Environment Setup

1. Install dependencies: `yarn install:immutable`
2. Set up environment variables in [env.js](mdc:env.js)
3. Configure Supabase local development
4. Start development server: `yarn start`

## Common Development Tasks

### Adding New Features

1. Create components in appropriate [src/components/](mdc:src/components/) directory
2. Add API services in [src/api/](mdc:src/api/)
3. Update Redux store if needed in [src/store/](mdc:src/store/)
4. Add navigation routes in [src/app/](mdc:src/app/)
5. Write tests in `__tests__/` directories

### Database Changes

1. Create migration: `yarn supabase:gen:migrate "description"`
2. Update types: `yarn supabase:gen:types`
3. Test with seed data in [supabase/seeds/](mdc:supabase/seeds/)

### Styling

- Use NativeWind classes (see [tailwind.config.js](mdc:tailwind.config.js))
- Follow design system in [src/theme/](mdc:src/theme/)
- Use responsive design patterns

### State Management

- Use Redux for global state (see [src/store/](mdc:src/store/))
- Use React Query for server state
- Use MMKV for local storage

## Debugging Tools

- React Query DevTools (configured in [src/app/\_layout.tsx](mdc:src/app/_layout.tsx))
- React Navigation DevTools
- MMKV DevTools
- Redux DevTools

## Deployment

- Use EAS Build for app store deployment
- Configure builds in [eas.json](mdc:eas.json)
- Use different profiles for dev/preview/production

## Common Patterns

- Feature-based development workflow
- Service layer for API interactions
- Type-safe database operations
- Consistent error handling
- Modular state management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/the-devbear) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
