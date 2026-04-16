## drug-prevention

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Drug Prevention Community Support Platform ("Phần mềm Hỗ trợ Phòng ngừa Sử dụng Ma túy trong Cộng đồng") built with a React frontend and Node.js/Express backend. The platform provides risk assessment tools, online courses, counselor appointments, and community program management for drug prevention education.

## Architecture

### Frontend (React + Vite)

- **Location**: `/frontend/`
- **Framework**: React 19 with Vite build tool
- **Styling**: Tailwind CSS v4 with PostCSS
- **Routing**: React Router DOM v7
- **State Management**: React Context (AuthContext)
- **Forms**: React Hook Form
- **HTTP Client**: Axios
- **UI Components**: Heroicons, Framer Motion
- **Notifications**: React Hot Toast

### Backend (Node.js + Express)

- **Location**: `/backend/`
- **Runtime**: Node.js with Express.js framework
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JWT tokens with bcryptjs password hashing
- **Security**: Helmet, CORS middleware
- **File Upload**: Multer
- **Email**: Nodemailer
- **Logging**: Morgan

## Common Development Commands

### Frontend Development

```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Start development server (Vite) on port 5173
npm run build        # Build for production
npm run preview      # Preview production build
npm run lint         # Run ESLint
```

### Backend Development

```bash
cd backend
npm install          # Install dependencies
npm run dev          # Start development server with nodemon on port 3000
npm start            # Start production server
npm test             # Run tests (currently not implemented)
```

### Database Seeding

```bash
cd backend
node scripts/seedData.js     # Seed initial data (users, courses, blogs, counselors)
node seedAssessments.js      # Seed assessment questions (ASSIST, CRAFFT, AUDIT, DAST)
```

## Key API Endpoints

The backend provides REST API endpoints under `/api/` prefix:

- **Authentication**: `/api/auth/` (register, login, profile, change-password)
- **Users**: `/api/users/` (user management)
- **Courses**: `/api/courses/` (online training courses)
- **Assessments**: `/api/assessments/` (ASSIST, CRAFFT, AUDIT, DAST risk assessments)
- **Appointments**: `/api/appointments/` (counselor scheduling)
- **Programs**: `/api/programs/` (community programs)
- **Counselors**: `/api/counselors/` (counselor management)
- **Blogs**: `/api/blogs/` (blog/news content)

## Database Schema Highlights

### User Model (`backend/models/User.js`)

- Comprehensive user profiles with role-based permissions (guest, member, staff, consultant, manager, admin)
- Age group categorization (student, university_student, parent, teacher, other)
- Assessment history tracking (ASSIST, CRAFFT, AUDIT, DAST scores with risk levels)
- Course enrollment and progress tracking with certificates
- Appointment and program participation history
- Notification preferences and multi-language support
- Role hierarchy and permission system built into the model

### Key Features

- Risk assessment tools with standardized scoring and recommendations
- Course progress tracking with certificate generation
- Counselor appointment scheduling with history tracking
- Multi-language support (Vietnamese primary, English secondary)
- Email verification and notification system
- Role-based access control throughout the application

## Environment Configuration

### Backend (.env)

```env
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/drug_prevention
JWT_SECRET=your_super_secret_jwt_key
FRONTEND_URL=http://localhost:5173
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASS=your_app_password
```

### Frontend (.env)

```env
VITE_API_URL=http://localhost:3000/api
VITE_APP_NAME=Phòng chống Ma túy Cộng đồng
```

## Code Conventions

- **Frontend**: Uses ES modules, JSX, and modern React patterns with hooks
- **Backend**: CommonJS modules, async/await patterns
- **Styling**: Tailwind CSS v4 utility classes with PostCSS
- **Database**: Mongoose schemas with validation, middleware, and virtual properties
- **Authentication**: JWT token-based with bcryptjs hashing (salt rounds: 12)
- **Error Handling**: Centralized error middleware in backend with environment-specific responses
- **Security**: Helmet for security headers, CORS configuration, 10MB request limit
- **API Architecture**: RESTful endpoints with consistent `/api/` prefix
- **State Management**: React Context with useReducer pattern for authentication

## Development Notes

- The application is bilingual (Vietnamese primary, English support)
- Uses MongoDB for data persistence with Mongoose ODM
- Frontend runs on port 5173 (Vite default), backend on port 3000
- No testing framework currently configured
- ESLint configured for frontend code quality
- Health check endpoint available at `/api/health`
- Axios interceptors handle automatic token refresh on 401 responses
- Role-based permissions implemented in User model with hierarchy system
- Assessment types supported: ASSIST, CRAFFT, AUDIT, DAST with scoring algorithms

## Key Architecture Patterns

### Authentication Flow

- JWT tokens with refresh token support
- Automatic token refresh via axios interceptors
- localStorage persistence for tokens
- Role-based access control with permission hierarchy

### Data Models Structure

- User model includes assessment history, course progress, appointments, and program participation
- Assessment results stored with risk levels (low, moderate, high) and recommendations
- Course progress tracked with completion percentages and certificate URLs
- Counselor appointments managed with scheduling and history tracking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khoatq201) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
