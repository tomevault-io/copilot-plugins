## botify

> **Botify** is a **full-stack multi-messaging bot marketplace platform** where users can:

# Botify Bot Marketplace Platform - Complete Project Preview

## 1. PROJECT OVERVIEW

**Botify** is a **full-stack multi-messaging bot marketplace platform** where users can:
- Buy and sell bots across multiple messaging platforms
- Create and manage email bots with campaign functionality
- Integrate with WhatsApp, Telegram, Discord, Slack, Instagram, and Email
- Access a marketplace to discover and purchase bots from other sellers
- Manage bot credentials and deployment

**Current Phase**: Phase 1
- ✅ User authentication (signup/login/password reset)
- ✅ Role-based access control (Admin, Seller, Buyer)
- ✅ Dashboard interfaces for all user roles
- ✅ Email bot creation and campaigns
- ✅ WhatsApp bot integration
- ✅ Marketplace for buying/selling bots
- ✅ Multi-language support (English, Spanish, Malayalam, Hindi, Tamil)

---

## 2. TECH STACK

### Frontend
- **Framework**: React 19 with Vite (bundler)
- **Styling**: Tailwind CSS
- **Routing**: React Router DOM v6
- **HTTP Client**: Axios
- **State Management**: React Context API
- **Auth Storage**: localStorage with JWT tokens
- **Route Protection**: PrivateRoute component with role-based checks

### Backend
- **Runtime**: Node.js (using ES modules: `"type": "module"`)
- **Framework**: Express.js
- **Database**: Supabase (PostgreSQL)
- **Authentication**: JWT (jsonwebtoken) + bcryptjs
- **Third-Party Integrations**:
  - Telegraf (Telegram bot SDK)
  - whatsapp-web.js (WhatsApp client with LocalAuth)
  - Nodemailer (Email delivery)

### Database (Supabase PostgreSQL)
- **users** table - User accounts with role_id
- **roles** table - Admin (1), Seller (2), Buyer (3)
- **bots** table - Bot metadata, credentials, configuration
- **marketplace_bots** table - Seller bot listings
- **purchases** table - Buyer transaction history
- **email_campaigns** table - Campaign tracking
- **whatsapp_campaigns** table - WhatsApp campaign data
- **password_reset_tokens** table - Password reset flow

---

## 3. PROJECT STRUCTURE

```
botify/
│
├── back-end/                          # Express server
│   ├── config/
│   │   ├── database.js               # Supabase client initialization
│   │   ├── db-init.sql               # Legacy PostgreSQL schema
│   │   └── *-migration.sql           # Feature-specific SQL migrations
│   ├── middleware/
│   │   └── auth.js                   # JWT verification + role checking
│   ├── routes/
│   │   ├── auth.js                   # /api/auth/* endpoints
│   │   ├── bot.js                    # /api/bot/* endpoints
│   │   └── marketplace.js            # /api/marketplace/* endpoints
│   ├── controllers/
│   │   └── WhatsAppController.js     # WhatsApp QR session singleton
│   ├── services/
│   │   └── telegramBotFactory.js     # Telegram bot creation
│   ├── utils/
│   │   └── botTokenCrypto.js         # Bot credential encryption
│   ├── uploads/                      # File upload directory
│   ├── server.js                     # Express app entry point
│   ├── package.json
│   └── .env.example
│
├── front-end/                        # React + Vite
│   ├── src/
│   │   ├── components/
│   │   │   ├── BotTable.jsx          # Bot listing table
│   │   │   ├── Modal.jsx             # Reusable modal
│   │   │   ├── PrivateRoute.jsx      # Route protection
│   │   │   ├── Navbar.jsx            # Navigation bar
│   │   │   ├── ErrorBoundary.jsx     # Error handling
│   │   │   └── ...
│   │   ├── pages/
│   │   │   ├── AdminDashboard.jsx    # Admin panel
│   │   │   ├── SellerDashboard.jsx   # Seller panel
│   │   │   ├── BuyerDashboard.jsx    # Buyer panel
│   │   │   ├── EmailBot.jsx          # Email bot creator
│   │   │   ├── WhatsAppBot.jsx       # WhatsApp bot setup
│   │   │   ├── Marketplace.jsx       # Bot marketplace
│   │   │   ├── Login.jsx             # Authentication
│   │   │   └── ...
│   │   ├── context/
│   │   │   ├── AuthContext.jsx       # Authentication state
│   │   │   └── LanguageContext.jsx   # Localization state
│   │   ├── locales/
│   │   │   ├── en.json               # English translations
│   │   │   ├── es.json               # Spanish translations
│   │   │   ├── ml.json               # Malayalam translations
│   │   │   ├── hi.json               # Hindi translations
│   │   │   └── ta.json               # Tamil translations
│   │   ├── utils/
│   │   │   └── api.js                # Axios instance with auth
│   │   ├── App.jsx                   # Route definitions
│   │   └── main.jsx                  # Entry point
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── package.json
│
├── README.md                         # Project overview
├── SETUP_GUIDE.md                    # Environment setup
├── QUICK_REFERENCE.md                # Commands & API
└── copilot-instructions.md           # Development guidelines
```

---

## 4. KEY FEATURES

### 4.1 Authentication & Authorization
- **Signup**: Email-based user registration with role selection
- **Login**: Email/password or Google OAuth
- **Password Reset**: Token-based reset flow via email
- **Role-Based Access Control**: Admin, Seller, Buyer roles with specific permissions
- **JWT Tokens**: Issued on login, stored in localStorage
- **Protected Routes**: PrivateRoute component enforces authentication and role requirements

### 4.2 User Dashboards
- **Admin Dashboard**: Manage users, moderate marketplace, view analytics
- **Seller Dashboard**: Create bots, list on marketplace, track earnings
- **Buyer Dashboard**: Browse marketplace, purchase bots, manage library

### 4.3 Bot Creation & Management
- **Email Bots**: Create automated email campaigns with scheduling
- **WhatsApp Bots**: Integrate via QR code scanning (whatsapp-web.js)
- **Telegram Bots**: Bot token-based integration
- **Bot Credentials**: Securely store and manage bot tokens/passwords
- **Campaign Tracking**: Monitor email and WhatsApp campaign performance

### 4.4 Marketplace
- **Seller Listings**: Showcase bots with descriptions and pricing
- **Bot Filtering**: Filter by platform (Email, WhatsApp, Telegram, Discord, Slack, Instagram)
- **Bot Sorting**: Sort by newest, most popular, price (ascending/descending)
- **Purchase Flow**: One-click bot purchase with transaction history
- **Resource Sharing**: Provide documentation links and configuration guides

### 4.5 Multi-Language Support
- **Supported Languages**: English, Spanish, Malayalam, Hindi, Tamil
- **Localization Pattern**: Nested JSON keys with dot notation
- **Dynamic Interpolation**: Support for `{{variableName}}` in translations
- **Fallback Strategy**: Target language → English → raw key
- **Context-Based**: React LanguageContext provides `t(key, replacements)` function

---

## 5. API ENDPOINTS

### Authentication
```
POST   /api/auth/signup              # Register new user
POST   /api/auth/login               # Login with credentials
POST   /api/auth/google-login        # Google OAuth
POST   /api/auth/verify-token        # Validate JWT
POST   /api/auth/password-reset      # Request reset token
POST   /api/auth/reset-password      # Set new password
```

### Bot Management
```
POST   /api/bot/create-email         # Create email bot
GET    /api/bot/list                 # List user's bots
DELETE /api/bot/:botId               # Delete bot
POST   /api/bot/whatsapp/connect     # Generate WhatsApp QR
GET    /api/bot/whatsapp/qr          # Fetch QR code
POST   /api/bot/telegram/create      # Create Telegram bot
```

### Marketplace
```
GET    /api/marketplace/bots         # List all marketplace bots
POST   /api/marketplace/list-bot     # Seller creates listing
DELETE /api/marketplace/:botId       # Remove from marketplace
POST   /api/marketplace/purchase     # Buyer purchases bot
GET    /api/marketplace/purchases    # Buyer's purchase history
```

### Health
```
GET    /api/health                   # Server health check (no auth)
```

---

## 6. DEVELOPMENT COMMANDS

### Backend Setup (from `back-end/` directory)
```bash
npm install                          # Install dependencies
npm run dev                          # Start with nodemon (auto-reload)
npm start                            # Production start
```

### Frontend Setup (from `front-end/` directory)
```bash
npm install                          # Install dependencies
npm run dev                          # Vite dev server (localhost:3000)
npm run build                        # Build for production
npm run preview                      # Preview production build
```

### Full-Stack Development
```bash
# Terminal 1: Backend
cd back-end
npm run dev                          # Runs on localhost:5000

# Terminal 2: Frontend
cd front-end
npm run dev                          # Runs on localhost:3000
```

---

## 7. ENVIRONMENT CONFIGURATION

### Backend (.env file)
```env
PORT=5000
NODE_ENV=development

# Supabase Configuration
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=eyJobXV0Lm...

# JWT Configuration
JWT_SECRET=your-strong-random-secret-key

# Email Configuration (Optional)
NODEMAILER_EMAIL=your-email@gmail.com
NODEMAILER_PASSWORD=your-app-password

# Telegram Bot (Optional)
TELEGRAM_BOT_TOKEN=your-telegram-bot-token

# Database (Legacy PostgreSQL - Optional)
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=botify
DB_USER=postgres
DB_PASSWORD=your-password
```

---

## 8. KEY CONVENTIONS

### Authentication Flow
1. User signs up → Password hashed with bcryptjs
2. JWT issued on login → Stored in localStorage
3. Axios intercepts requests → Attaches `Authorization: Bearer <token>` header
4. Backend middleware validates JWT → Enforces role requirements
5. Token expiry → Frontend redirects to login

### Database Naming
- Tables use snake_case: `users`, `marketplace_bots`, `email_campaigns`
- Foreign keys: `role_id`, `user_id`, `bot_id`
- Timestamps: `created_at`, `updated_at`
- Soft deletes (where applicable): `deleted_at`

### API Response Format
**Success:**
```json
{
  "success": true,
  "data": { /* response payload */ },
  "message": "Action successful"
}
```

**Error:**
```json
{
  "success": false,
  "error": "Description of error",
  "statusCode": 400
}
```

### Component Patterns
- **Context for State**: Use React Context API for auth and language
- **Custom Hooks**: `useAuth()`, `useLanguage()` for state access
- **Protected Components**: `PrivateRoute` wrapper with role checking
- **Error Handling**: `ErrorBoundary.jsx` catches React errors
- **Localization**: Use `t(key, replacements)` for all user-facing strings

### File Organization
- Components in `src/components/` (reusable UI elements)
- Pages in `src/pages/` (route-level components)
- Utilities in `src/utils/` (helpers, API client)
- Context in `src/context/` (global state)
- Locales in `src/locales/` (translation JSON files)

---

## 9. IMPORTANT TECHNICAL DETAILS

### WhatsApp Integration
- **Library**: whatsapp-web.js with LocalAuth
- **Session Storage**: `.wwebjs_auth/` directory (auto-created)
- **Flow**: 
  1. User requests WhatsApp connection
  2. Backend generates QR code
  3. User scans with phone
  4. Session persisted locally
  5. Messages sent via authenticated client
- **Key Class**: `WhatsAppController.js` (singleton pattern)

### Email Bot Campaigns
- **Library**: Nodemailer
- **Features**: Scheduling, recipient management, analytics
- **Storage**: `email_campaigns` table tracks sent emails
- **Constraints**: SMTP credentials configured in backend

### Telegram Integration
- **Library**: Telegraf
- **Setup**: Bot token from Telegram BotFather
- **Factory Pattern**: `telegramBotFactory.js` creates bot instances
- **Lifecycle**: Created, configured, and managed per user

### Localization System
- **Framework**: React Context + JSON catalogs
- **Key Structure**: Nested with dot notation (e.g., `marketplace.filters.email`)
- **Interpolation**: Regex-based `{{variableName}}` replacement
- **Fallback**: Language → English → raw key
- **Usage**: `t('marketplace.title', { seller: 'John' })`

### Bot Credential Management
- **Storage**: Encrypted in `bots` table
- **Crypto Utility**: `botTokenCrypto.js` handles encryption/decryption
- **Fields**: bot_token, bot_password, api_keys (encrypted)
- **Security Note**: Consider vault integration for production

---

## 10. CURRENT DEVELOPMENT STATUS

### Completed
✅ Core authentication system (signup/login/password reset)
✅ Role-based access control with three user types
✅ Three dashboard interfaces (Admin, Seller, Buyer)
✅ Email bot creation with campaign functionality
✅ WhatsApp integration with QR-based authentication
✅ Marketplace for bot discovery and purchase
✅ Multi-language UI (English, Spanish, Malayalam, Hindi, Tamil)
✅ JWT-based route protection

### Known Limitations
⚠️ Bot credentials stored in database (consider vault/HSM for production)
⚠️ WhatsApp integration tied to local file system (`.wwebjs_auth/`)
⚠️ Limited campaign analytics (basic tracking only)
⚠️ No formal test suite (TODO: add Jest/Vitest)

### Planned Features
🔜 Discord and Slack bot integration
🔜 Advanced marketplace filtering and search
🔜 Bot analytics dashboard
🔜 Multi-language campaigns
🔜 Bot template library

---

## 11. CRITICAL PATHS FOR CHANGES

### Adding a New Route
1. Create handler in `back-end/routes/<domain>.js`
2. Add middleware for auth and role checking
3. Create API call in `front-end/src/utils/api.js`
4. Use in component via `const response = await api.get('/endpoint')`

### Adding a Database Migration
1. Create SQL file in `back-end/config/<feature>-migration.sql`
2. Execute manually: `psql -U postgres -d botify -f config/<feature>-migration.sql`
3. Document in SETUP_GUIDE.md

### Adding a Frontend Page
1. Create `.jsx` file in `src/pages/`
2. Define route in `App.jsx` with role requirement
3. Wrap with `<PrivateRoute>` if auth required
4. Use `useAuth()` and `useLanguage()` hooks
5. Call backend APIs and localize all strings

### Adding Translations
1. Add keys to all five locale files (en/es/ml/hi/ta.json)
2. Use dot notation: `section.subsection.key`
3. Use `{{variableName}}` for dynamic content
4. Access in component: `t('section.subsection.key', { variableName: value })`

---

## 12. DEBUGGING CHECKLIST

- **Backend not starting?** → Check .env file, verify Supabase credentials, check port 5000
- **Frontend can't reach backend?** → Verify backend running on localhost:5000, check Axios baseURL
- **WhatsApp QR not showing?** → Restart backend, check `.wwebjs_auth/` permissions
- **JWT token expired?** → Clear localStorage, re-login
- **Localization not working?** → Verify key exists in all five locale files, check dot notation
- **Role-based access denied?** → Verify JWT payload has correct role_id, check middleware requirements

---

## 13. USEFUL RESOURCES

- **Setup Documentation**: [SETUP_GUIDE.md](SETUP_GUIDE.md)
- **API Reference**: [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
- **Project Vision**: [README.md](README.md)
- **Development Rules**: [copilot-instructions.md](.github/copilot-instructions.md)

---

**Generated**: April 3, 2026  
**Project Phase**: 1 (MVP - In Development)  
**Language**: JavaScript/React on Frontend, Node.js on Backend

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MathewJoseph2005)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MathewJoseph2005)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
