## genius-fleets

> Genius Fleets is a comprehensive car rental management system designed for Al Jisr Car Rentals. It provides a complete solution for managing fleet operations, customer bookings, financial tracking, HR management, and document processing.

# Genius Fleets - Car Rental Management System

## About
Genius Fleets is a comprehensive car rental management system designed for Al Jisr Car Rentals. It provides a complete solution for managing fleet operations, customer bookings, financial tracking, HR management, and document processing.

## Architecture
**Hybrid Full-Stack Application**
- **Frontend**: React SPA (Single Page Application) built with Vite
- **Backend**: Express.js server for API routes and static file serving
- **Database**: PostgreSQL (migrated from Supabase)
- **Authentication**: Auth0 for user management and role-based access
- **File Storage**: Railway volumes for document and image storage
- **Deployment**: Railway platform

## Tech Stack

### Frontend
- **React 18** - UI library
- **Vite** - Build tool and dev server
- **Tailwind CSS** - Styling framework
- **Radix UI** - Component primitives
- **Framer Motion** - Animations
- **React Router DOM** - Client-side routing
- **React Hook Form** - Form management
- **Recharts** - Data visualization
- **Leaflet** - Maps integration

### Backend
- **Express.js 4** - Web server framework
- **Node.js** - Runtime environment
- **PostgreSQL** - Primary database
- **Railway Volumes** - File storage system

### Authentication & Security
- **Auth0** - Authentication provider
- **@auth0/auth0-react** - Auth0 React SDK
- **Role-based access control** - Multi-tier user permissions

### Database & ORM
- **Supabase Client** - Database abstraction layer (transitioning to native PostgreSQL)
- **Custom Entity Classes** - Data access layer
- **PostgreSQL** - Relational database

### Deployment & Infrastructure
- **Railway** - Cloud platform
- **Docker/Nixpacks** - Containerization
- **Railway Volumes** - Persistent file storage
- **Health Check API** - System monitoring

## Key Features

### Fleet Management
- Vehicle inventory tracking
- Maintenance scheduling
- Damage logging and incident reports
- Vehicle image library
- GPS tracking integration

### Booking System
- Customer booking management
- Rental agreements
- Calendar-based availability
- Mobile booking interface

### Financial Management
- Invoice generation and tracking
- Payment processing
- Expense management
- Asset depreciation
- Tax/VAT calculations
- Financial reporting and KPIs

### Human Resources
- Employee management
- Attendance tracking
- Leave request system
- Payroll management
- Shift scheduling

### Document Management
- Customer document upload
- Vehicle documentation
- Staff document storage
- Legal document management
- AI-powered document processing

### User Management
- Role-based access control
- Multi-module permissions
- User access rules
- Corporate client management

## Environment Configuration

### Required Environment Variables
```bash
# Database
DATABASE_URL=postgresql://username:password@host:port/database

# Auth0 (Client-side)
VITE_AUTH0_DOMAIN=your-domain.auth0.com
VITE_AUTH0_CLIENT_ID=your-auth0-client-id
VITE_AUTH0_AUDIENCE=your-api-audience

# Auth0 (Server-side)
AUTH0_SECRET=your-generated-secret-key
AUTH0_BASE_URL=https://your-app.railway.app

# Storage
STORAGE_PATH=/app/storage

# Application
NODE_ENV=production
PORT=3000
```

## Development Commands
```bash
# Development
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Build and start (Railway deployment)
npm run start:railway

# Linting
npm run lint
```

## File Structure
```
src/
├── api/                 # Database entities and integrations
├── components/          # Reusable UI components
├── contexts/           # React contexts (Auth, etc.)
├── pages/              # Main application pages
├── lib/                # Utility libraries
└── utils/              # Helper functions

server.js               # Express server entry point
railway.toml           # Railway deployment configuration
```

## Migration Status
- ✅ **Auth**: Migrated from Supabase Auth to Auth0
- ✅ **Storage**: Migrated from Supabase Storage to Railway volumes
- 🔄 **Database**: Currently using Supabase client with Railway PostgreSQL
- ✅ **Deployment**: Migrated from Vercel to Railway

## Deployment
The application is deployed on Railway with:
- Automatic builds from Git
- PostgreSQL database service
- Volume-mounted file storage
- Health check monitoring
- Environment variable management

Access the live application at: `https://genius-fleets-production.up.railway.app/`

---
> Source: [hith3sh/genius-fleets](https://github.com/hith3sh/genius-fleets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
