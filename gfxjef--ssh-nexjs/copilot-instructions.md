## ssh-nexjs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a full-stack application with a **Flask backend** and **Next.js 15 frontend** that manages a document/catalog management system with AWS S3 integration.

### Backend (Flask + MySQL)
- **Main entry point**: `backend/app.py`
- **Database config**: `backend/db/config.py` (uses environment variables)
- **Architecture**: Modular blueprint structure with separate modules for different domains
- **Key modules**:
  - `db/bienestar/` - Employee wellness management with document attachments
  - `db/encuestas/` - Survey system with dynamic form generation  
  - `db/formularios/` - Dynamic form builder with configurable field types
  - `db/marketing/` - Stock management, purchase requests, and email signatures
    - `marketing/firmas/` - HTML email signature generation with FTP upload and SMTP delivery
  - `db/pdf_manager/` - Document processing with S3 storage (replaces local storage)
  - `db/login/` - JWT-based authentication system

### Frontend (Next.js 15)
- **App directory structure**: Uses Next.js App Router
- **Port**: Runs on port 5000 (configured in package.json)
- **Authentication**: JWT-based with middleware protection
- **Key features**:
  - Dashboard with protected routes
  - Rich text editing with TinyMCE and Summernote
  - Charts with ApexCharts and Chart.js
  - QR code generation
  - File upload handling

## Development Commands

### Backend (Flask)
```bash
# Setup virtual environment (first time only)
cd backend
python -m venv venv

# Activate virtual environment
source venv/Scripts/activate  # On Windows
# source venv/bin/activate     # On Linux/Mac

# Install Python dependencies
pip install -r requirements.txt

# Start backend development server (with venv activated)
python app.py

# Start with Gunicorn (production-like)
gunicorn --config gunicorn.conf.py app:app

# Database utilities
cd backend
python list_users.py          # List all users
python migrate_data_urls.py   # Migrate S3 URLs

# Module-specific database initialization
python -c "from db.bienestar.setup import init_bienestar_db; init_bienestar_db()"
python -c "from db.formularios.setup import init_formularios_db; init_formularios_db()"
```

### Frontend (Next.js)
```bash
cd frontend

# Install dependencies
npm install

# Development server (port 5000)
npm run dev

# Production build and start
npm run build
npm run start

# Linting
npm run lint

# File upload tests (for S3 integration)
npm run test:upload
npm run test:delete
```

## Database Configuration

The application uses **MySQL** with configuration managed through environment variables:
- **Connection Management**: `backend/db/mysql_connection.py` provides MySQLConnection class
- **Per-request Connections**: Each query establishes and closes its own connection (no persistent pooling)
- **Configuration**: Database details loaded from `backend/db/config.py` via environment variables
- **Module Setup**: Each module has `setup.py` for database schema initialization
- **Query Pattern**: Modules use MySQLConnection().execute_query() for database operations

**Important**: Database credentials are stored in `.env` file - never commit these to git.

### Database Architecture Patterns
```python
# Standard query pattern used throughout modules
db_ops = MySQLConnection()
results = db_ops.execute_query("SELECT * FROM table WHERE id = %s", (user_id,))

# Transaction pattern for multiple operations
db_ops = MySQLConnection()
db_ops.start_transaction()
try:
    db_ops.execute_query("INSERT INTO table1 ...", params1)
    db_ops.execute_query("UPDATE table2 ...", params2) 
    db_ops.commit_transaction()
except Exception as e:
    db_ops.rollback_transaction()
```

## Key Technical Details

### Authentication Flow
- JWT tokens stored in HTTP-only cookies
- Middleware protection on `/dashboard/*` routes (frontend/middleware.ts:8)
- Token validation happens server-side in Flask

### File Upload & S3 Integration
- Documents processed and stored in AWS S3
- PDF processing with page conversion to WebP/PNG
- Metadata tracking in `catalogos` and `catalogos_docs` tables
- S3 configuration documented in `backend/configurar_s3_*.md`

### Frontend Architecture Patterns
- **Component Organization**: Feature-based in `frontend/components/` (e.g., `marketing/`, `bienestar/`)
- **API Proxy Routes**: `frontend/app/api/` proxies requests to Flask backend
- **App Router**: Next.js 15 app directory with nested layouts
- **Permission System**: Role-based access control via `frontend/lib/permissions/`
  - MenuAccessManager.ts defines route permissions by user role
  - PermissionsContext.tsx provides React context for permission checks
- **TypeScript**: Strict configuration throughout with interface definitions

### Environment Configuration
- Frontend: Uses `.env.local` for Next.js environment variables
- Backend: Uses `.env` for Flask environment variables  
- Production: Configured for Render deployment (render.yaml)

## Common Debugging

### Database Connection Issues
- Check `backend/db/config.py` for proper environment variable loading
- Verify MySQL credentials in `.env` file
- Test connection with `python backend/test_imports.py`

### Authentication Problems
- Check JWT token presence in browser cookies (`auth-token`)
- Verify token validation in Flask routes
- Check middleware logs in browser console

### S3 Upload Issues  
- Verify AWS credentials are properly configured
- Check S3 bucket permissions and CORS settings
- Review `backend/configurar_s3_cors.md` for setup details

## Development Workflow

1. **Setup**: Ensure both backend and frontend environments are properly configured
2. **Development**: Run backend on default Flask port, frontend on port 5000
3. **Database**: Use provided utility scripts for user management and migrations
4. **Testing**: Use npm test scripts for file upload functionality
5. **Production**: Deploy backend to Render, frontend separately or as static export

## IMPORTANT: Port Management Protocol

**MANDATORY**: Before running ANY test or starting ANY server (frontend, backend, or any port), you MUST follow this exact sequence:

1. **FIRST - Check if the port is in use**:
   ```bash
   # Check port 5000 (frontend)
   netstat -ano | findstr :5000

   # Check port 5001 (backend Flask default)
   netstat -ano | findstr :5001

   # Check any specific port
   netstat -ano | findstr :<PORT_NUMBER>
   ```

2. **SECOND - If the port is in use, KILL the process**:
   ```bash
   # Kill process by PID (get PID from previous command)
   taskkill /PID <PID_NUMBER> /F

   # Alternative: Kill all node processes (for frontend)
   taskkill /F /IM node.exe

   # Alternative: Kill all python processes (for backend)
   taskkill /F /IM python.exe
   ```

3. **THIRD - Verify the port is now free**:
   ```bash
   netstat -ano | findstr :<PORT_NUMBER>
   # Should return empty (no output)
   ```

4. **FOURTH - Only AFTER confirming the port is free, start the server**:
   ```bash
   # Frontend
   cd frontend && npm run dev

   # Backend
   cd backend && python app.py
   ```

**NEVER skip this verification sequence. Always ensure ports are closed before starting new server instances.**

## Module-Specific Notes

### Marketing Module (`db/marketing/`)
- **Stock Management**: Inventory tracking with category-based organization
- **Solicitudes**: Purchase request workflow with approval system
- **Firmas**: Email signature generation system with differentiated instructions:
  - Kossomet users: Hotmail-specific tutorial and email templates
  - Kossodo users: Gmail-specific tutorial and email templates
  - FTP upload for HTML files and custom images
  - SMTP delivery with service-specific instructions

### Bienestar Module (`db/bienestar/`)
- Employee wellness tracking with document attachments
- Document management sub-module with S3 integration
- Employee data used by other modules (e.g., Firmas uses `/api/bienestar/users`)

### PDF Manager (`db/pdf_manager/`)
- **S3 Architecture**: `routes_s3.py` replaces local file storage
- Document processing pipeline: Upload → S3 Storage → Metadata in MySQL
- Page conversion to WebP/PNG for preview functionality

### Formularios (`db/formularios/`)
- Dynamic form builder with field type configuration
- Database schema auto-creation based on form definitions
- Integration with other modules for data collection

### Permission System Architecture
- **Role-based Access**: `admin`, `gerente`, `marketing`, `rrhh`, `sistemas`, `asesor`, `invitado`, `atencion`
- **Menu Access Control (V2 - DB)**: Permissions are stored in the `permisos` table and managed via `/dashboard/sistema/roles`. The old hardcoded `MenuAccessManager.ts` (V1) has been removed.
- **Context-based Checks**: `PermissionsContext` provides React hooks (`hasMenuAccess`, `hasButtonAccess`) that query `MenuAccessManagerV2` which loads permissions from the DB API.
- **Accent-safe comparison**: `MenuAccessManagerV2.hasAccess()` uses `normalizeStr()` to strip diacritics before comparing codes, so `Catálogos` matches `Catalogos`.

**IMPORTANT — Sidebar ↔ DB Sync (Manual Process):**
When adding a new section/submenu to the sidebar (`navItemsConfig` in `frontend/app/dashboard/components/sidebar.tsx`), you MUST also register it in the `permisos` table in the database so it appears in the Roles & Permissions admin page (`/dashboard/sistema/roles`). This is NOT automatic. Steps:
1. Add the menu item to `navItemsConfig` in `sidebar.tsx`
2. Insert the corresponding permission in the DB table `permisos` with `tipo='menu'`, the correct `codigo` (path format: `Parent/Child`), `modulo`, and `padre_id`
3. Assign the permission to the appropriate roles in `roles_permisos` table
4. Optionally update `backend/db/roles/migrations.py` → `MENU_PERMISSIONS` dict for future re-seeding

When working with any module, check its `__init__.py` for blueprint registration and `setup.py` for database initialization.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gfxjef)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gfxjef)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
