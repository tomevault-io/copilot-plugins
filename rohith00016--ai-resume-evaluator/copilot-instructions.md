## ai-resume-evaluator

> ├── client/                 # React frontend application

## 📁 Project Structure

### Root Directory Layout

```
vidly-v2/
├── client/                 # React frontend application
├── server/                 # Node.js backend API
├── API_ROUTES.md          # API documentation (MANDATORY)
└── .env                   # Environment variables (DO NOT COMMIT)
```

## 🎨 Frontend Development Rules

### Technology Stack

- **Language**: Plain JavaScript only - NO TypeScript files
- **Styling**: Tailwind CSS utility classes throughout
- **UI Components**: shadcn/ui for complex UI elements
- **Icons**: Lucide icons only (no other icon libraries)
- **State Management**: Redux Toolkit with RTK Query for API calls
- **Routing**: React Router v6

### Component Design Principles

- Keep components small, focused, and reusable
- Use shadcn/ui for input fields, buttons, dialogs, popovers, modals, etc.
- Compose shadcn and Tailwind—apply utility classes for layout and responsive design
- Maintain responsive design for all devices
- Use Lucide icons exclusively for all icon needs

### Styling Guidelines

- Use Tailwind CSS utility classes throughout—avoid custom CSS unless necessary
- Overwrite with @apply for shadcn overrides only in `/styles`
- Ensure all UI components are responsive for all device sizes

### State Management

- Use Redux Toolkit (createSlice, createAsyncThunk, etc.)
- Put each feature's state in `features/featureName/`
- Use Redux for cross-component/shared state (user auth, project/video list, notifications)
- Use internal useState for local form/UI state
- **Use RTK Query for all API calls** - centralize in `/services`

### API Integration

- Centralize API logic in `/services` using RTK Query
- Use custom hooks for data fetching and logic sharing
- Connect to server running on `http://localhost:3000` in development

### Routing & Authentication

- Use React Router v6; pages in `/pages/`
- Protect routes based on auth state in Redux using JWT

### Code Quality

- Use Prettier and ESLint—enforce via Cursor workspace rules
- Meaningful file/folder, component, and slice naming
- Delete unused code before merge

## 🔧 Backend Development Rules

### Technology Stack

- **Runtime**: Node.js with Express
- **Database**: MongoDB with Mongoose
- **Authentication**: JWT with bcrypt for password hashing
- **Validation**: express-validator/Joi
- **Testing**: Jest (unit) and Supertest (integration)

### Project Structure Principles

- Separate logic: controllers (business), routes (define endpoints), models (Mongo schemas), middleware (auth, error), config (db)
- No business logic in routes—routes call controller handlers only

### Authentication & Security

- Use JWT for login/session, secure endpoints with auth middleware
- Passwords hashed with bcrypt
- Never log sensitive information
- Validate all incoming data (express-validator/Joi)
- CORS setup for client URL in dev/prod

### Database & Models

- Define clear, validated models for Users, Projects, Videos, Comments, Annotations
- Use mongoose lean queries for performance

### Error Handling

- Central error handler middleware
- Consistent API error shape

### Environment & Deployment

- All secrets (DB, JWT, etc.) in `.env` - DO NOT commit `.env`
- Use PM2 or Docker for production
- Log events securely using winston or similar logger

## 📋 General Development Rules

### Version Control

- Descriptive commit messages
- Pull requests reviewed before merges

### Documentation

- README with setup, usage, and deployment steps
- JSDoc-style comments for functions/components as needed

### Cursor-Specific Workflow

- Use Cursor's AI code suggestions to scaffold, but review and refactor before merging
- Use workspaces to keep backend and frontend separate but linked
- Leverage Cursor's in-editor docs and refactoring tools
- Share workspace rules and Prettier/ESLint configs among team members

## 📚 MANDATORY Documentation Requirements

### 1. API Route Documentation

**File**: `API_ROUTES.md` (at project root)

**Must Include**:

- Route path, HTTP method, brief description
- Required request parameters/body/query
- Responses (status codes, response shape)
- Auth requirements (public/protected/admin)
- Keep this file up-to-date as routes are added/modified

### 2. Environment Variable Documentation

**Files**:

- `client/README_ENV.md` (or README.md with "Environment Variables" section)
- `server/README_ENV.md` (or README.md with "Environment Variables" section)

**Must Include**:

- Every required (and optional) environment variable
- Example values and descriptions for each variable
- **WARNINGS**: NOT to commit `.env` or secrets

## ⚠️ Critical Reminders

- **NEVER commit `.env` files or secrets**
- **Always use Lucide icons** - no other icon libraries
- **use Tailwind version 4** - for styling
- **Maintain responsive design** for all UI components
- **Use RTK Query** for all API calls in the frontend
- **Server runs on `http://localhost:5000`** in development

- **Plain JavaScript only** - no TypeScript files
- **Dont wait for application to install** - notify and proceed for next steps
- **dont hallucinate and code just follow instruction** - always follow user instructions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohith00016) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
