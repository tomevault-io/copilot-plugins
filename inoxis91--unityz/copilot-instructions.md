## unityz

> - **Name**: Guild Manager Website

# Guild Manager Website - Context & Instructions

## Project Overview
- **Name**: Guild Manager Website
- **Goal**: A multi-guild World of Warcraft guild management site similar to Warcraft Roster.
- **Key Feature**: Battle.net SSO integration for roster and role synchronization.

## Tech Stack
- **Frontend**: Angular (Latest) with Signals and Standalone Components.
- **Backend**: Node.js (Express) with TypeScript.
- **Database**: PostgreSQL (Raw SQL via `pg` pool, layered in Services).
- **Deployment**: Railway (via GitHub).

## Engineering Standards & Best Practices

### General Principles
- **Strict Typing**: Use of `: any` is strictly forbidden. All data structures must have interfaces or types.
- **Naming Conventions**: 
    - TypeScript/Frontend: `camelCase`.
    - Database/Backend Entities: `snake_case`.
    - Components/Classes: `PascalCase`.
- **DRY & Clean Code**: Refactor redundant logic into services or helper functions. Use descriptive variable names.
- **Internationalisation (i18n)** : Tout nouveau développement sur le frontend doit obligatoirement être internationalisé en utilisant le service `I18nService` (les clés de traduction doivent être enregistrées pour `fr` et `en` dans `frontend/src/app/services/i18n.ts`). Les chaînes de caractères brutes (hardcoded) sont strictement interdites dans les templates HTML et fichiers TS utilisateur.

### Backend Architecture (Layered)
- **Routes Layer**: Handles HTTP requests, basic response sending, and delegates to Services.
- **Service Layer**: Contains all business logic and database interactions (`pool.query`). **ABSOLUTELY NO SQL in routes.**
- **Validation**: All incoming data (req.body, req.params, req.query) MUST be validated using **Zod** schemas before processing.
- **Error Handling**: Use the centralized `errorHandler` middleware. Throw errors with specific status codes.
- **Security**: 
    - Protect routes using `isAuthenticated` and `isAdmin` middlewares.
    - **Character Sync Enforcement**: Users MUST have at least one synchronized character to access non-auth features.
    - Never log or return sensitive user data (like access tokens).
    - Always use `trust proxy` for secure session cookies on Railway.
    - **Session Security**: Cookies must use `httpOnly: true`, `secure: true`, and `sameSite: 'none'` in production.

### Performance & Resource Optimization
- **Frontend Serving**: Use **Nginx Alpine** in production Dockerfiles to minimize RAM usage (< 5MB). Avoid running Node.js process for static assets in production.
- **Assets**: Compress all PNG/JPG assets. Prefer WebP or SVG for icons to reduce network bandwidth.
- **Database**: Use connection pooling (`pg` pool) for efficient query management.

### Frontend Standards (Angular)
- **Route Guards**: Protect access to pages using `authGuard` and `adminGuard`. No manual redirects in `ngOnInit`.
- **State Management**: Use **Signals** for reactive state and data sharing.
- **Architecture**:
    - Use Standalone Components.
    - Encapsulate API calls in Services (`AuthService`, `CalendarService`, etc.).
    - Use `computed` signals for derived data (e.g., filtered lists, permission checks).
- **UI/UX**: 
    - Maintain a modern, professional look (rounded corners, soft shadows, clear icons).
    - Use CSS variables for consistent class and role colors.
    - Provide immediate visual feedback for user actions.

### Database Management
- **Schema Updates**: Use `CREATE TABLE IF NOT EXISTS` and `ALTER TABLE` in `src/lib/db.ts` to ensure automatic migrations upon deployment.
- **Integrity**: Use foreign key constraints and cascading deletes to maintain data consistency.

## Development Workflow
1. **Local DB**: Use a local PostgreSQL instance or Railway's development database.
2. **Battle.net API**: Register an application at [Blizzard Developer Portal](https://develop.battle.net/).
3. **Environment Variables**:
   - `BNET_CLIENT_ID`
   - `BNET_CLIENT_SECRET`
   - `BNET_CALLBACK_URL`
   - `DATABASE_URL`
   - `SESSION_SECRET`
   - `FRONTEND_URL` (Required for correct production redirects)

## Final Mandate
Any new feature implementation MUST follow this layered architecture. If a developer or AI identifies technical debt, it should be addressed *before* or *during* the implementation of the new feature.

## Roadmap & Futures Fonctionnalités
Voici les fonctionnalités prévues pour les prochaines phases de développement :

1.  **Intégration Raider.IO API** : Récupérer automatiquement le score MM+ des personnages lors de l'import et l'afficher sur le Dashboard et dans les compositions MM+.
2.  **Attendance Tracker (Suivi de présence)** : Calcul automatique du taux de participation aux raids sur les 30 derniers jours pour aider les officiers.
3.  **Notifications Discord Automatisées** :
    *   Annonce automatique des nouveaux événements dans un canal dédié.
    *   Rappels automatiques (pings) 24h avant le début d'un événement pour les membres non inscrits.
4.  **Gestion des Loots (BiS List)** : Système permettant aux joueurs de lister les objets souhaités sur chaque boss. Interface admin pour faciliter l'attribution des loots (Master Loot / Loot Council).
5.  **Audit Logs** : Historique des actions administratives (création/suppression d'événements, changement de rôles, etc.).

---
> Source: [inoxis91/unityz](https://github.com/inoxis91/unityz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
