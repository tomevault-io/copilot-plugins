## messadestudio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Messad Estudio is an architectural firm's website with two main systems:
1. **Authentication System**: Complete user authentication with Node.js backend
2. **Lead Processing System**: Automated normative analysis using n8n workflow automation with Claude AI

The project combines a static frontend (HTML/CSS/JS) with backend services for authentication and intelligent lead processing.

## Common Development Commands

### Backend Development
```bash
# Install dependencies and setup
./setup-backend.sh

# Start backend server (development)
cd backend && npm run dev

# Start backend server (production)
cd backend && npm start

# Test backend health
curl http://localhost:3001/api/health

# Test setup script
./test-setup.sh
```

### Frontend Development
Use VS Code Live Server extension or any local server to serve the frontend files.

## Architecture Overview

### Frontend Structure
- **Static Website**: Bootstrap-based architecture website with SCSS compilation
- **Authentication UI**: Modal-based login/registration system
- **Client Libraries**: Google OAuth integration, JWT token management
- **Responsive Design**: Mobile-first approach with Bootstrap grid system

### Backend Structure (`/backend`)
- **Express.js API**: REST API with authentication endpoints
- **MongoDB Database**: User storage with Mongoose ODM
- **Authentication**: JWT tokens with 7-day expiration, bcrypt password hashing
- **Security**: CORS, rate limiting, input validation, helmet security headers

### Key Files
- `js/simple-auth.js`: Frontend authentication manager with Google OAuth and email/password login
- `backend/server.js`: Express server with security middleware and API routes
- `backend/models/User.js`: User schema with authentication methods
- `backend/routes/auth.js`: Authentication endpoints
- `backend/routes/users.js`: User management endpoints

## Development Workflow

### Setting Up Development Environment
1. Run `./setup-backend.sh` to install dependencies and create configuration
2. Configure `backend/.env` with MongoDB connection and OAuth credentials
3. Start backend with `cd backend && npm run dev`
4. Use VS Code Live Server for frontend development
5. Test authentication flow end-to-end

### Authentication System Architecture
- **Dual Authentication**: Email/password and Google OAuth support
- **Token Management**: JWT tokens stored in localStorage with automatic verification
- **Session Persistence**: Users remain logged in across browser sessions
- **Security Features**: Rate limiting (100 requests/15min), CORS protection, password hashing with bcrypt (12 rounds)

### API Endpoints
- `POST /api/auth/register` - User registration with email/password
- `POST /api/auth/login` - User login with email/password
- `POST /api/auth/google` - Google OAuth authentication
- `POST /api/auth/verify-token` - JWT token verification
- `POST /api/auth/logout` - User logout
- `GET /api/users/profile` - Get user profile
- `PUT /api/users/profile` - Update user profile
- `GET /api/health` - System health check

### Frontend Components
- **Authentication Modal**: Tabbed interface for login/registration
- **Google OAuth Button**: One-click Google sign-in integration
- **Session Management**: Automatic token refresh and user state persistence
- **UI Updates**: Dynamic login button updates based on authentication state

## Configuration Requirements

### Essential Configuration
- **MongoDB**: Local installation or MongoDB Atlas connection string
- **Environment Variables**: JWT secret, database URL, CORS settings
- **Backend Server**: Must run on port 3001 for frontend integration

### Optional Configuration
- **Google OAuth**: Client ID and secret for Google sign-in functionality
- **Production Settings**: Production MongoDB, domain configuration
- **SMTP**: For future email verification features

## Testing

### Development Testing
- Backend health: `curl http://localhost:3001/api/health`
- Registration: Test email/password account creation
- Login: Test email/password and Google OAuth login
- Session persistence: Refresh browser and verify login state
- Logout: Test session clearing

### Key Test Scenarios
- User registration with validation
- Email/password login flow
- Google OAuth authentication (if configured)
- JWT token persistence across browser sessions
- Authentication modal functionality
- Mobile responsive authentication UI

## Security Considerations

### Implemented Security Features
- **Password Security**: bcrypt hashing with 12 rounds
- **JWT Tokens**: Secure token generation with 7-day expiration
- **Rate Limiting**: 100 requests per 15 minutes per IP
- **Input Validation**: Server-side validation with express-validator
- **CORS Protection**: Configured for development and production origins
- **Security Headers**: Helmet middleware for security headers

### Best Practices
- Never commit `.env` files or secrets to repository
- Use HTTPS in production for secure token transmission
- Keep dependencies updated for security patches
- Monitor authentication logs for suspicious activity

## Production Deployment

### Backend Deployment Options
- **Heroku**: Easy deployment with add-on MongoDB
- **DigitalOcean**: VPS with PM2 process management
- **AWS/Azure**: Cloud platforms with managed databases

### Production Environment Variables
```env
NODE_ENV=production
MONGODB_URI=mongodb+srv://...
JWT_SECRET=secure-production-secret
GOOGLE_CLIENT_ID=production-client-id
FRONTEND_URL_PRODUCTION=https://yourdomain.com
```

### Frontend Updates for Production
- Update `apiBaseUrl` in `js/simple-auth.js` to production backend URL
- Configure Google OAuth for production domain
- Ensure CORS settings include production frontend URL

---

## n8n Lead Processing System

### Overview
Automated workflow that receives lead data from the website, performs normative analysis using Claude AI, and sends detailed reports via email.

**Workflow Location**: n8n Cloud at federvgh.app.n8n.cloud
**Workflow Name**: "Messad Estudio - Lead Processing"
**Workflow ID**: EG5DPQd3LeHy7yv0

### System Architecture

**Frontend → n8n Cloud → External APIs → Email**

1. User submits form on [solicitar-analisis.html](solicitar-analisis.html)
2. [js/simple-lead-capture.js](js/simple-lead-capture.js) sends data to n8n webhook
3. n8n processes through multiple nodes (geocoding, cadastral data, AI analysis)
4. Analysis sent via email to client and architect
5. Data logged to Google Sheets

### Workflow Nodes

1. **Webhook Trigger** - Receives lead data from website
2. **Geo Coding** - Google Maps Geocoding API (gets coordinates)
3. **Formatear Coordenadas Google** - Extracts lat/lon from response
4. **Consultar GeoServer WMS** - Municipality cadastral data API
5. **Preparar Contexto** - CODE node - Prepares Claude prompt with v5 digesto
6. **Message a model** - Native Anthropic node (Claude 3 Opus)
7. **Formatear Resultados** - CODE node - Formats email HTML/plain text
8. **Email al Arquitecto** - Sends analysis to architect
9. **Preparar Datos para Sheets** - CODE node - Formats for logging
10. **Guardar en Google Sheets** - Logs lead data
11. **Webhook Response** - Returns success message to frontend

### Key Implementation Files

**Reference Code** (not deployed, reference for n8n nodes):
- [CODIGO_LIMPIO_PREPARAR_CONTEXTO.js](CODIGO_LIMPIO_PREPARAR_CONTEXTO.js) - "Preparar Contexto" node code
- [CODIGO_FORMATEAR_RESULTADOS_ACTUALIZADO.js](CODIGO_FORMATEAR_RESULTADOS_ACTUALIZADO.js) - "Formatear Resultados" node code

**Active Frontend**:
- [solicitar-analisis.html](solicitar-analisis.html) - Lead capture page
- [js/simple-lead-capture.js](js/simple-lead-capture.js) - Form submission handler
- [css/calculadora.css](css/calculadora.css) - Page styles

### Claude AI Integration

**Model**: Claude 3 Opus (claude-3-opus-20240229)
**Alternative**: Claude 3 Sonnet (80% cost reduction, same quality)

**Input Structure**:
- System prompt with structured v5 digesto (regulatory data)
- User message with lot details (address, dimensions, coordinates)

**Output**: Professional normative analysis in markdown format

**Cost per Analysis**:
- Current (Opus): $0.25 per query
- Recommended (Sonnet): $0.05 per query (change model in "Message a model" node)

**Token Usage**:
- Input: ~8,000 tokens (digesto + instructions + lot data)
- Output: ~1,700 tokens (complete analysis)

### Digesto v5 Structure

The digesto is a structured JSON database embedded in the "Preparar Contexto" node containing:
- Direct street → zone mapping (11 main streets)
- Detailed zone regulations (FOS, FOT, heights, setbacks)
- Step-by-step analysis algorithm
- Calculation formulas
- Default fallback zone

**Accuracy**: ~95% (validated with test cases)

### External API Dependencies

1. **Google Maps Geocoding API**
   - Converts address to coordinates
   - Required for cadastral lookup

2. **Municipality GeoServer WMS**
   - Returns cadastral data (parcel number, catastral surface, neighborhood)
   - URL: https://geoserver.msebas.gob.ar/geoserver/wms

3. **Anthropic Claude API**
   - Normative analysis generation
   - Requires API key in n8n credentials

4. **Gmail SMTP** (or email service)
   - Sends analysis to clients
   - Configure in n8n email node

5. **Google Sheets API**
   - Logs lead data for tracking
   - Requires Google Service Account

### Testing the Workflow

**Test Case (Urquiza 248)**:
```json
{
  "direccion": "Urquiza 248",
  "frente": 10,
  "profundidad": 40,
  "superficie": 400,
  "email_cliente": "test@example.com",
  "nombre": "Test User",
  "whatsapp": "1234567890"
}
```

**Expected Results**:
- Zone: AG-C1 (Área General - Centro 1)
- FOS: 0.85 (85% of lot)
- FOT: 4.25 (425% of lot surface)
- Max height: 22m (Perfil XVI)

### Common Modifications

**Change Claude Model** (cost optimization):
1. Open n8n workflow
2. Find "Message a model" node
3. Change model to: `claude-3-sonnet-20240229`
4. Save and test

**Update Preparar Contexto Code**:
1. Copy code from CODIGO_LIMPIO_PREPARAR_CONTEXTO.js
2. Open "Preparar Contexto" node in n8n
3. Replace all code
4. Save

**Update Email Formatting**:
1. Copy code from CODIGO_FORMATEAR_RESULTADOS_ACTUALIZADO.js
2. Open "Formatear Resultados" node in n8n
3. Replace all code
4. Save

### Frontend Success Message

The webhook returns:
```json
{
  "status": "success",
  "message": "Análisis generado y enviado exitosamente",
  "direccion": "...",
  "email": "..."
}
```

Handle in [js/simple-lead-capture.js](js/simple-lead-capture.js) to show success message to user.

### Important Notes

- **Never modify the digesto structure** without extensive testing
- **Always test with Urquiza 248** before deploying changes
- **Code in .js files is reference only** - actual code runs in n8n Cloud
- **Webhook URL** is in n8n workflow, update in frontend if changed
- **API credentials** are stored in n8n Cloud (not in this repo)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Federvgh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Federvgh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
