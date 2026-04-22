## hospital-management

> This is a hospital management system built with **100% Python**:

# Hospital Management - AI Coding Instructions
## 100% Python Healthcare Management System

## Project Overview

This is a hospital management system built with **100% Python**:
- **Backend**: Django + Django REST Framework
- **Frontend**: Streamlit (no HTML/CSS/JavaScript required)
- **Database**: SQLite (production-ready with PostgreSQL)

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Streamlit Frontend                        │
│              (100% Python UI - pages/components)            │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/JSON
┌──────────────────────────▼──────────────────────────────────┐
│                    Django REST API                           │
│                   (hospital/api.py)                          │
└──────────────────────────┬──────────────────────────────────┘
                           │ ORM
┌──────────────────────────▼──────────────────────────────────┐
│                    SQLite Database                          │
│                    (hospital/models.py)                     │
└─────────────────────────────────────────────────────────────┘
```

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Backend | Django | 4.2+ |
| API | Django REST Framework | 3.14+ |
| Frontend | Streamlit | 1.35+ |
| Database | SQLite | 3.x |
| Container | Docker | Latest |

## Key Files

### Backend (Django)
- `hospital/models.py` - Database models (Doctor, Patient, Appointment, Discharge)
- `hospital/api.py` - REST API views
- `hospital/serializers.py` - DRF serializers
- `hospital/views.py` - Django views (legacy HTML templates)
- `hospital/api_urls.py` - API URL routing

### Frontend (Streamlit)
- `streamlit_app/main.py` - Entry point
- `streamlit_app/pages/` - Multi-page navigation
- `streamlit_app/components/ui.py` - Reusable UI components
- `streamlit_app/services/api_client.py` - API service layer
- `streamlit_app/styles/custom.css` - Clinical Clarity design system

## Code Style

### Python
- Follow PEP 8
- Use type hints where beneficial
- Prefer `async def` for API calls
- Use f-strings for string formatting

### Streamlit
- Use `st.cache_data` for expensive operations
- Group related widgets in columns
- Use forms for submit buttons
- Use session state for user data

### Django
- Use class-based views (APIView)
- Validate input with serializers
- Return consistent response format

## Common Tasks

### Add a new page
1. Create `streamlit_app/pages/new_page.py`
2. Import from `streamlit_app.utils.session`
3. Call `show_sidebar()` at the start
4. Call `page_header()` for page title

### Add API endpoint
1. Add serializer in `hospital/serializers.py`
2. Add view in `hospital/api.py`
3. Add URL in `hospital/api_urls.py`

### Add component
1. Add function in `streamlit_app/components/ui.py`
2. Follow existing patterns

## Design System

See `DESIGN_SYSTEM.md` for:
- Color palette (Clinical Clarity theme)
- Typography (Inter font family)
- Component library
- Spacing system

## Testing

```bash
# Backend tests
pytest hospital/tests.py -v

# Frontend tests
pytest tests/ -v

# Code quality
ruff check .
```

## Deployment

### Docker
```bash
docker-compose up -d
```

### Manual
```bash
# Backend
python manage.py migrate
gunicorn hospitalmanagement.wsgi:application

# Frontend
streamlit run streamlit_app/main.py
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| SECRET_KEY | Django secret key | Required in production |
| DEBUG | Debug mode | True |
| DATABASE_URL | Database connection | SQLite |

## API Endpoints

### Auth
- `POST /api/auth/login/` - Login
- `POST /api/auth/logout/` - Logout

### Admin
- `GET /api/admin/dashboard/` - Dashboard stats
- `GET/POST /api/admin/doctors/` - List/create doctors
- `GET/PUT/DELETE /api/admin/doctors/{id}/` - Doctor detail
- `GET/POST /api/admin/patients/` - List/create patients
- `GET/PUT/DELETE /api/admin/patients/{id}/` - Patient detail
- `GET/POST /api/admin/appointments/` - List/create appointments
- `POST /api/admin/appointments/{id}/approve/` - Approve appointment
- `POST /api/admin/discharge/{id}/` - Discharge patient

### Doctor
- `GET /api/doctor/dashboard/` - Dashboard
- `GET /api/doctor/patients/` - My patients

### Patient
- `GET /api/patient/dashboard/` - Dashboard
- `GET /api/patient/doctors/` - Available doctors
- `POST /api/patient/book-appointment/` - Book appointment

## Security Notes

- All API endpoints require authentication (except login)
- User roles: ADMIN, DOCTOR, PATIENT
- Use environment variables for secrets
- Enable HTTPS in production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/in4SECxMinDandy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
