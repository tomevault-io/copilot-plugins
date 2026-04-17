## samudra-paket-erp

> 1. Use React Native with Expo SDK for mobile application development


# Mobile Structure Rules

## Architecture

1. Use React Native with Expo SDK for mobile application development
2. Implement TypeScript for type safety and better developer experience
3. Follow offline-first architecture with local data synchronization
4. Use WatermelonDB for reactive database and offline storage
5. Implement role-based application variants with shared core functionality

## Project Structure

1. Organize mobile app with the following folder structure:
   ```
   /mobile
     /src
       /app                  # App entry points and navigation
       /components           # Reusable UI components
         /atoms              # Basic UI elements
         /molecules          # Simple component groups
         /organisms          # Complex components
       /features             # Feature-based modules
         /auth               # Authentication
         /pickup             # Pickup management
         /delivery           # Delivery management
         /collection         # Payment collection
         /warehouse          # Warehouse operations
         /monitoring         # Manager monitoring
       /hooks                # Custom React hooks
       /navigation           # Navigation configuration
       /services             # API and device services
       /store                # State management
       /utils                # Utility functions
       /theme                # Styling and theming
       /types                # TypeScript type definitions
       /database             # Database models and schemas
       /assets               # Static assets
       /i18n                 # Internationalization
   ```

2. Each feature module should contain:
   - Screens
   - Components specific to the feature
   - Feature-specific hooks
   - Feature-specific state management

## Navigation Structure

1. Use React Navigation with the following structure:
   ```
   /navigation
     /stacks                 # Stack navigators
     /tabs                   # Tab navigators
     /drawers                # Drawer navigators
     /auth-navigator.tsx     # Authentication flows
     /app-navigator.tsx      # Main app navigation
     /linking.ts             # Deep linking configuration
     /types.ts               # Navigation type definitions
     /index.ts               # Navigation exports
   ```

2. Implement role-based navigation flows:
   - Courier: Pickup-focused navigation
   - Driver: Delivery-focused navigation
   - Collector: Collection-focused navigation
   - Warehouse: Warehouse operations navigation
   - Manager: Monitoring and oversight navigation

## State Management

1. Organize Redux store with the following structure:
   ```
   /store
     /slices                 # Redux Toolkit slices
     /middleware             # Custom middleware
     /selectors              # Reusable selectors
     /hooks.ts               # Custom Redux hooks
     /index.ts               # Store configuration
   ```

2. Use React Query for server state with this structure:
   ```
   /services
     /api
       /client.ts            # API client configuration
       /auth.ts              # Authentication endpoints
       /pickup.ts            # Pickup endpoints
       /delivery.ts          # Delivery endpoints
       /collection.ts        # Collection endpoints
       /warehouse.ts         # Warehouse endpoints
     /queries                # React Query hooks
     /mutations              # React Query mutations
   ```

## Offline Data Structure

1. Implement WatermelonDB with the following structure:
   ```
   /database
     /models                 # Database models
     /schemas                # Database schemas
     /migrations             # Database migrations
     /adapters               # Database adapters
     /sync                   # Synchronization logic
     /index.ts               # Database exports
   ```

2. Define models for each core entity:
   - Tasks
   - Shipments
   - Customers
   - Addresses
   - Signatures
   - Photos
   - Payments

## Device Integration

1. Organize device services with the following structure:
   ```
   /services
     /device
       /camera.ts            # Camera access
       /location.ts          # Location services
       /signature.ts         # Signature capture
       /barcode.ts           # Barcode scanning
       /printer.ts           # Bluetooth printing
       /storage.ts           # Local storage
       /network.ts           # Network status
       /permissions.ts       # Permission handling
   ```

## Assets and Theming

1. Organize assets with the following structure:
   ```
   /assets
     /images                 # Image assets
     /icons                  # Icon assets
     /fonts                  # Custom fonts
     /animations             # Lottie animations
   ```

2. Implement theming with the following structure:
   ```
   /theme
     /colors.ts             # Color definitions
     /typography.ts          # Typography styles
     /spacing.ts            # Spacing constants
     /shadows.ts            # Shadow styles
     /index.ts              # Theme exports
   ```

## Testing Structure

1. Organize tests with the following structure:
   ```
   /tests
     /components             # Component tests
     /features               # Feature tests
     /services               # Service tests
     /hooks                  # Hook tests
     /utils                  # Utility tests
     /mocks                  # Test mocks
     /fixtures               # Test fixtures
   ```

## Internationalization

1. Organize translations with the following structure:
   ```
   /i18n
     /locales
       /id                   # Indonesian translations
       /en                   # English translations
     /index.ts               # i18n configuration
   ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhammadhilmi007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
