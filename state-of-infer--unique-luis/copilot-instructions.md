## unique-luis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Project Architecture

This is a React 19 application built with Vite that implements a car rental website with auction functionality. The project uses a feature-based architecture with Redux Toolkit for state management and Firebase for data persistence.

### Key Technologies
- **React 19** with React Router DOM v7 for routing
- **Redux Toolkit** for state management with multiple slices
- **Firebase/Firestore** for database operations
- **Vite** for build tooling and development
- **Axios** for API communication
- **Bootstrap 5** with custom styling

### State Management Architecture

The application uses Redux Toolkit with feature-based slices:

- `auth/userSlice` - User authentication and session management
- `modal/modalSlice` - Modal dialog state management  
- `registro/registroSlice` - User registration workflows
- `slideshow/slideshowSlice` - Image carousel management
- `subastaSlice/subastaSlice` - Auction functionality and bidding
- `busqueda/busquedaSlice` - Search and filtering logic
- `vender/venderSlide` - Car selling/listing functionality
- `loader/loaderSlice` - Loading states across the app

### API Layer

The application uses a custom API wrapper (`src/api/api.js`) with:
- `fetch()` - Authenticated requests with JWT tokens
- `fetchSinToken()` - Unauthenticated requests
- `postFile()` - File upload functionality
- `getFile()` - File download with authentication

### Component Structure

- **Layout Components**: `Header`, `Footer` provide consistent navigation
- **Page Components**: Route-based components in `src/pages/`
- **Section Components**: Reusable page sections (hero, testimonials, etc.)
- **Input Components**: Form controls with validation in `src/components/input/`
- **UI Components**: Utility components like `ScrollTop`, `Breadcrumb`

### Firebase Integration

Firebase is configured in `src/db/firebase.js` with Firestore for data persistence. The configuration is imported from `src/db/config/firebase.js`.

### Directory Structure

```
src/
├── api/              # API layer and HTTP utilities
├── components/       # Reusable React components (Feature-Based)
│   ├── shared/      # Components shared between features
│   │   ├── ui/      # UI utilities (Breadcrumb, ScrollTop)
│   │   ├── forms/   # Form input components (InputText, etc.)
│   │   ├── layout/  # Layout components (Header, Footer, Modals)
│   │   └── feedback/ # Loading, messages, notifications
│   ├── business/    # Business logic specific components
│   │   ├── auction/ # Auction functionality (PujasMartillo)
│   │   ├── search/  # Search functionality (Busqueda, CarSidebar)
│   │   └── vehicle/ # Vehicle-related components
│   └── sections/    # Page sections (moved to views as needed)
├── const/           # Constants and configuration
├── db/              # Firebase configuration
├── hooks/           # Custom React hooks
├── layout/          # Layout wrapper components
│   ├── Layout.jsx   # Main app layout wrapper
│   ├── LayoutPage.jsx # Protected routes layout
│   └── GlobalMessage.jsx # Global messaging component
├── router/          # Routing configuration
│   ├── AppRouter.jsx # Main application router
│   ├── PrivateRoute.jsx # Private route wrapper
│   ├── ProtectedRoutes.jsx # Protected routes component
│   └── PublicRoute.jsx # Public route wrapper
├── store/           # Redux store and slices
├── utils/           # Utility functions
└── views/           # Feature-based view components
    ├── home/        # Home page functionality
    ├── about/       # About page functionality
    ├── subastas/    # Auction/Cars functionality
    ├── contact/     # Contact page functionality
    ├── services/    # Services page functionality
    ├── blog/        # Blog functionality
    ├── detalle/     # Auction detail view
    └── vender/      # Selling functionality
```

## Working with the Codebase

### Adding New Features
1. Create feature directory in `src/views/` for new functionality
2. Create Redux slice if state management is needed
3. Add API functions in `src/api/api.js` if backend integration required
4. Create components following the existing naming conventions
5. Update routing in `src/router/AppRouter.jsx` for new pages

### Router Architecture
The application uses a sophisticated routing system with authentication:

- **AppRouter**: Main router that handles authentication state and renders appropriate routes
- **Layout Components**: Provide consistent structure across the application
- **Route Protection**: PrivateRoute and PublicRoute components control access based on authentication
- **Authentication Flow**: Uses JWT tokens with automatic refresh functionality

### State Management Patterns
- Use Redux Toolkit `createSlice` for new state
- Async operations use `createAsyncThunk` in thunk files
- Access state via `useSelector` and dispatch via `useDispatch`

### API Integration
- Use `fetch()` for authenticated requests
- Use `fetchSinToken()` for public endpoints
- Handle loading states with `loaderSlice`
- Store authentication tokens in localStorage

### Component Conventions
- Functional components with hooks
- Props validation and TypeScript support available
- Consistent naming: PascalCase for components, camelCase for functions
- Modular CSS with Bootstrap classes and custom styles

## Firebase Setup

The project requires Firebase configuration in `src/db/config/firebase.js`. Ensure this file contains valid Firebase project credentials before running the application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/state-of-infer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
