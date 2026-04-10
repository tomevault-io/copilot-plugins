## tics-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TICS Store is a modern e-commerce web application with a complete user management system, built with Node.js/Express backend and vanilla JavaScript frontend.

## Development Commands

```bash
# Install dependencies
npm install

# Development server with auto-reload
npm run dev

# Production server
npm start

# Linting
npm run lint

# Testing
npm test
```

## Architecture Overview

### Backend (Node.js/Express)
- **Database**: SQLite with normalized schema
- **Authentication**: JWT tokens with bcrypt password hashing
- **API**: RESTful endpoints for users, products, orders
- **Security**: Rate limiting, CORS, helmet, input validation
- **File Structure**:
  - `/database/` - Database initialization and schema
  - `/controllers/` - Business logic controllers
  - `/routes/` - API route definitions
  - `/middleware/` - Authentication and security middleware

### Frontend (Vanilla JavaScript)
- **Architecture**: Modular JavaScript with class-based components
- **Styling**: CSS with custom properties, responsive design
- **Features**: Product catalog, cart, user authentication, admin panel
- **File Structure**:
  - `/css/` - Stylesheets (main + admin)
  - `/js/` - JavaScript modules
  - `/data/` - Static product data (temporary)

## Key Technologies

- **Backend**: Node.js 18+, Express.js, SQLite3, JWT, bcryptjs
- **Frontend**: HTML5, CSS3 (Grid/Flexbox), ES6+ JavaScript
- **Security**: helmet, express-rate-limit, express-validator, CORS
- **Development**: nodemon, Docker, nginx
- **Database**: SQLite with tables for users, products, orders

## Database Schema

### Users Table
- Authentication and user management
- Roles: customer, admin
- Soft delete with is_active flag

### Products Table
- Normalized product catalog
- Categories, pricing, inventory

### Orders Table
- Order management with related order_items

## API Endpoints

### Authentication
- `POST /api/users/register` - User registration
- `POST /api/users/login` - User login

### User Management (Protected)
- `GET /api/users/profile` - Current user profile
- `GET /api/users` - List users (admin only)
- `PUT /api/users/:id` - Update user
- `DELETE /api/users/:id` - Delete user (admin only)

## Environment Setup

1. Copy `.env` file and configure:
   - `JWT_SECRET` - Secure random string for JWT signing
   - `PORT` - Server port (default: 3000)
   - `DB_PATH` - SQLite database file path
   - `CORS_ORIGIN` - Allowed CORS origins

2. Database auto-initializes on first run
3. Default admin user: `admin@ticsstore.com` / `admin123`

## Development Workflow

### Adding New Features
1. **Backend**: Create routes → controllers → update database schema if needed
2. **Frontend**: Add JavaScript modules → update CSS → create/update HTML
3. **Testing**: Test endpoints → test UI functionality

### Code Conventions
- Use ES6+ features (classes, modules, async/await)
- Follow REST conventions for API design
- Use semantic HTML and BEM-like CSS naming
- Validate all user inputs on both client and server
- Handle errors gracefully with user-friendly messages

## Security Considerations

- JWT tokens stored in localStorage (consider httpOnly cookies for production)
- All passwords hashed with bcrypt
- Input validation on all endpoints
- Rate limiting on API routes
- CORS properly configured
- SQL injection prevention through parameterized queries

## Deployment

### Development
```bash
npm run dev
# Access: http://localhost:3000
# Admin: http://localhost:3000/admin
```

### Production with Docker
```bash
docker-compose up -d
# With nginx: docker-compose --profile production up -d
```

## Admin Panel

- Located at `/admin`
- Full CRUD operations for users
- Dashboard with statistics
- Role-based access control
- Responsive design for mobile/desktop

## Important Notes

- Database file is gitignored but directory structure is maintained
- Static product data in JSON format (to be migrated to database)
- Frontend uses vanilla JavaScript (no frameworks)
- Images sourced from Unsplash API for consistency
- Mobile-first responsive design approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Avillegasa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Avillegasa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
