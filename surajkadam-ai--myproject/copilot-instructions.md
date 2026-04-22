## myproject

> This is a full-stack Applicant Tracking System (ATS) built with React frontend and Node.js/Express backend, using MySQL database for data persistence. The system is designed for managing job requisitions, candidates, applications, interviews, and complete hiring workflows.

# CLAUDE.md - ATS Applicant Tracking System

## Project Overview
This is a full-stack Applicant Tracking System (ATS) built with React frontend and Node.js/Express backend, using MySQL database for data persistence. The system is designed for managing job requisitions, candidates, applications, interviews, and complete hiring workflows.

## Project Structure
```
Hkthon_ATS_latest_V5.24/
├── backend/                 # Node.js/Express API server
│   ├── src/
│   │   ├── config/         # MySQL database and Swagger configuration
│   │   ├── controllers/    # Request handlers for routes
│   │   ├── middleware/     # Auth, validation, error handling
│   │   ├── models/         # Data models (Application, Candidate, Interview, etc.)
│   │   ├── routes/         # API route definitions
│   │   ├── utils/          # Utility functions
│   │   └── server.js       # Main Express server
│   ├── scripts/            # Database seeding scripts
│   └── package.json        # Backend dependencies
├── frontend/               # React application
│   ├── src/
│   │   ├── components/     # Reusable UI components (shadcn/ui)
│   │   ├── pages/          # Page components
│   │   ├── services/       # API service layer (axios)
│   │   ├── store/          # Redux store and slices
│   │   ├── contexts/       # React contexts
│   │   ├── hooks/          # Custom React hooks
│   │   ├── lib/            # Utility functions
│   │   ├── utils/          # Helper utilities
│   │   ├── App.jsx         # Main app component
│   │   └── main.jsx        # App entry point
│   ├── public/             # Static assets
│   └── package.json        # Frontend dependencies
└── package.json            # Root package scripts
```

## Technology Stack

### Backend
- **Framework**: Express.js 4.19.2
- **Database**: MySQL 9.4 with mysql2 driver (3.15.1)
- **Authentication**: JWT (jsonwebtoken 9.0.2) with bcryptjs 2.4.3
- **Validation**: express-validator 7.1.0, joi 17.13.3
- **Security**: helmet, cors
- **File Upload**: multer 2.0.2
- **Documentation**: Swagger UI Express + swagger-jsdoc
- **Development**: nodemon 3.1.0

### Frontend
- **Framework**: React 18.3.1
- **Routing**: React Router DOM 6.28.0
- **State Management**: Redux Toolkit 2.2.6 (@reduxjs/toolkit)
- **HTTP Client**: Axios 1.7.9
- **UI Components**: shadcn/ui built on Radix UI primitives
  - @radix-ui/react-dialog, select, tabs, toast, etc.
- **Styling**: Tailwind CSS 3.4.14 with tailwindcss-animate
- **Forms**: React Hook Form 7.52.2 with @hookform/resolvers
- **Validation**: Zod 3.23.8
- **Date Handling**: date-fns 3.6.0
- **Icons**: Lucide React 0.427.0
- **Charts**: Recharts 3.2.1
- **Drag & Drop**: react-dnd 16.0.1 with html5-backend
- **Tables**: @tanstack/react-table 8.21.3
- **Build Tool**: Vite 5.3.4
- **Utilities**: clsx, class-variance-authority, tailwind-merge

## Database Schema

### MySQL Database: `ats_db`

**Main Tables:**
1. **users** - System users (admin, hr_manager, recruiter, interviewer, hiring_manager)
2. **jobRequisitions** - Job postings and requisitions
3. **candidates** - Candidate profiles with skills and experience
4. **applications** - Job applications linking candidates to jobs
5. **interviews** - Interview scheduling and feedback
6. **auditLogs** - System audit trail

**Key Fields:**
- `candidate_code` - Human-readable candidate identifier (CAND-XXX)
- `application_code` - Human-readable application identifier (APPL-XXX)
- `scheduledDate` - DATETIME field for interview scheduling
- `interviewerId` - Single interviewer assignment (not array)

## Development Commands

### Installation
```bash
# Install all dependencies (backend + frontend)
npm run install:all

# Install backend only
npm run install:backend

# Install frontend only
npm run install:frontend
```

### Development
```bash
# Run both frontend and backend concurrently
npm run dev

# Run backend only (port 5000)
npm run dev:backend

# Run frontend only (port 3000)
npm run dev:frontend
```

### Production
```bash
# Build frontend for production
npm run build

# Start backend in production mode
npm start
```

### Database Management
```bash
# Seed job requisitions
npm run seed:job-requisitions

# Clear job requisitions
npm run clear:job-requisitions
```

### Code Quality
```bash
# Frontend linting
cd frontend && npm run lint
cd frontend && npm run lint:fix
```

### Port Management
```bash
# Kill ports if needed
npx kill-port 3000 5000

# Quick start script
start-dev.bat
```

## Key Features

### Core Functionality
- **User Management**: Role-based authentication and authorization
- **Job Requisitions**: Create, manage, and track job postings
- **Candidate Management**:
  - Profile management with candidate codes (CAND-XXX)
  - Skills tracking (primary and other skills)
  - Resume/document upload
  - Bulk candidate import via CSV
- **Application Workflow**:
  - Application tracking with codes (APPL-XXX)
  - Status management (applied, screening, interviewed, assessment, offer, hired, rejected, withdrawn)
  - Stage tracking throughout hiring pipeline
  - Candidate-to-job mapping
- **Interview Scheduling**:
  - Multiple interview types (phone, video, in-person, technical, panel)
  - Interviewer assignment
  - Date/time scheduling with MySQL DATETIME format
  - Duration tracking
  - Location/meeting link management
  - Feedback and rating system
  - Interview list with clickable rows → navigate to application details
- **Workflow Tabs**:
  - Overview
  - Screening
  - Interviews
  - Offer
  - Candidate Engagement
  - Pre-Onboarding & Joining
  - History / Audit Trail
- **Reports & Analytics**: Dashboard with metrics and recent activity
- **Audit Logging**: Comprehensive tracking of all system actions

### UI/UX Features
- Modern, responsive design with Tailwind CSS
- Reusable shadcn/ui components
- Real-time form validation
- Loading states and error handling
- Search, filter, and pagination
- Drag-and-drop functionality
- Interactive charts and visualizations
- Quick actions menu
- Toast notifications

## Environment Configuration

### Backend (.env)
```env
PORT=5000
NODE_ENV=development
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRES_IN=7d

# MySQL Database Configuration
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=root
DB_NAME=ats_db
DB_PORT=3306

# Frontend
FRONTEND_URL=http://localhost:3000

# API
API_VERSION=v1
```

### Frontend (.env)
```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=ATS - Applicant Tracking System
VITE_APP_VERSION=1.0.0
```

## API Documentation
- **Swagger UI**: http://localhost:5000/api-docs
- **API Base URL**: http://localhost:5000/api/v1
- **Health Check**: http://localhost:5000/health
- **Authentication**: JWT Bearer token required for most endpoints

## Port Configuration
- **Frontend**: http://localhost:3000 (Vite dev server) - FIXED PORT
- **Backend**: http://localhost:5000 (Express server) - FIXED PORT
- **MySQL**: localhost:3306
- **API Documentation**: http://localhost:5000/api-docs

### Port Management Notes
- Frontend uses `strictPort: true` in Vite config to prevent auto-increment
- Backend explicitly sets PORT=5000 in package.json scripts
- Use `npx kill-port 3000 5000` to clean up ports if needed
- Quick start available via `start-dev.bat`

## Recent Updates & Fixes

### Interview Scheduling Feature (Fixed)
1. **Database Alignment**:
   - Fixed frontend to send `interviewerId` (string) instead of `interviewers` (array)
   - Fixed `scheduledDate` format to MySQL DATETIME (`YYYY-MM-DD HH:MM:SS`)
   - Updated interview types to match database enum: `phone`, `video`, `in-person`, `technical`, `panel`

2. **Backend Model Updates** (`backend/src/models/Interview.js`):
   - Fixed `create()` method to use correct field names
   - Updated `getInterviewsWithDetails()` to properly await database calls and handle null values
   - Fixed `findByInterviewer()` to use `interviewerId` equality check (not array)
   - Fixed `getInterviewStatistics()` to work with flat rating field
   - Updated all date references from `scheduledAt` to `scheduledDate`

3. **Validation Rules** (`backend/src/middleware/validation.js`):
   - Updated `interviewValidation.create` to match database structure
   - Fixed field names and enum values

4. **Frontend Improvements** (`frontend/src/pages/interviews/`):
   - **CreateInterviewPage.jsx**: Fixed data submission format to match backend
   - **InterviewsPage.jsx**:
     - Made table rows clickable
     - Navigate to application detail page on row click
     - Pass `?tab=interviews` query parameter

5. **Application Details Navigation**:
   - **ApplicationDetailsPage.jsx**: Added support for `?tab=interviews` query parameter
   - Automatically switches to Interviews tab when coming from interview list

### Application Management
- Added `application_code` generation (APPL-XXX format)
- Enhanced `getApplicationsWithDetails()` to include `candidateName` and `jobTitle`

## Node.js Requirements
- Node.js >= 20.0.0 (specified in engines)
- Frontend uses ES modules (type: "module")
- Backend uses CommonJS

## Development Best Practices
1. **Concurrency**: Uses `concurrently` to run both servers in development
2. **Hot Reload**: Enabled for both frontend (Vite) and backend (nodemon)
3. **Modern React**: Functional components with hooks (no class components)
4. **State Management**: Redux Toolkit for predictable state updates
5. **Form Handling**: React Hook Form + Zod for validation
6. **Type Safety**: PropTypes and TypeScript definitions where applicable
7. **Code Style**: ESLint configuration for consistent code quality
8. **API Layer**: Centralized API service layer with axios interceptors
9. **Error Handling**: Comprehensive error boundaries and try-catch blocks
10. **Security**: JWT authentication, CORS, Helmet, input validation

## Important Implementation Notes

### Database vs Code Mismatches (Resolved)
- ✅ Interview model now matches MySQL schema exactly
- ✅ Validation rules align with database constraints
- ✅ Frontend sends data in correct format
- ✅ All date fields use proper MySQL DATETIME format

### MySQL Date/Time Handling
```javascript
// Correct format for scheduledDate field
const scheduledDate = new Date(formData.scheduledDate)
const mysqlDate = scheduledDate.toISOString().slice(0, 19).replace('T', ' ')
// Result: "2024-09-30 14:30:00"
```

### Interview Types Enum
Valid values: `'phone'`, `'video'`, `'in-person'`, `'technical'`, `'panel'`

### Navigation Pattern
```javascript
// From InterviewsPage to ApplicationDetailsPage with tab selection
navigate(`/applications/${applicationId}?tab=interviews`)
```

## Sample Data Files
- `sample_candidates_import.csv` - Candidate bulk import template
- `sample_applications_import.csv` - Application bulk import template
- `test-candidate.json` - Test candidate data structure

## Additional Documentation
- `README.md` - General project information
- `DEPLOYMENT.md` - Deployment instructions
- `USER_GUIDE.txt` - End-user guide for the application

## Troubleshooting

### Common Issues
1. **Port Already in Use**: Run `npx kill-port 3000 5000`
2. **MySQL Connection**: Check MySQL server is running and credentials are correct
3. **Backend Crashes**: Check for undefined values in database queries
4. **Validation Errors**: Ensure frontend data matches backend validation rules
5. **Date Issues**: Use MySQL DATETIME format, not ISO strings

### Debug Commands
```bash
# Check MySQL connection
mysql -u root -proot ats_db -e "SELECT 1;"

# View backend logs
cd backend && npm run dev

# Check running processes on ports
netstat -ano | findstr :3000
netstat -ano | findstr :5000
```

## Future Enhancements
- Unit and integration tests
- Email notifications for interview scheduling
- Calendar integration
- Advanced reporting and analytics
- Document management system
- Offer letter generation
- Background check integration
- Employee onboarding workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SurajKadam-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
