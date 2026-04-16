## renters

> Tu travailles sur POURACCORD, une plateforme B2B2C française permettant :

# ============================================================
# POURACCORD — Cursor AI Rules
# Plateforme B2B2C de gestion et validation de dossiers locataires
# Version : 1.0 | Février 2026
# ============================================================
# Ce fichier est lu par Cursor à chaque conversation.
# Il donne à l'IA le contexte permanent du projet.
# Ne pas supprimer ou modifier les sections marquées [CRITIQUE].
# ============================================================


# ────────────────────────────────────────────────────────────
# 1. CONTEXTE PRODUIT
# ────────────────────────────────────────────────────────────

Tu travailles sur POURACCORD, une plateforme B2B2C française permettant :
- Aux LOCATAIRES (gratuit) : constituer un dossier unique sécurisé et le partager avec des agences
- Aux AGENCES (800€ HT/mois, essai 30j) : accéder à des dossiers pré-vérifiés par IA anti-fraude
- Aux ADMINS : modérer, gérer les utilisateurs et améliorer l'IA

Valeur clé : validation anti-fraude multicouche par IA (8 niveaux) + conformité RGPD automatisée.

Environnements :
- Dev local     : localhost:3001 (backend) | localhost:3000 (frontend)
- Staging       : staging.pouraccord.com
- Production    : api.pouraccord.com / pouraccord.com


# ────────────────────────────────────────────────────────────
# 2. STACK TECHNIQUE [CRITIQUE]
# ────────────────────────────────────────────────────────────

## Backend
- Runtime       : Node.js 20 LTS
- Framework     : Express 4.x
- Langage       : TypeScript 5.x strict
- ORM           : Sequelize 6.x (MySQL)
- Base de données: MySQL 8.0+ (charset utf8mb4, moteur InnoDB)
- Validation    : Joi (TOUS les inputs sans exception)
- Auth          : jsonwebtoken + speakeasy (2FA TOTP)
- Upload        : Multer + AWS SDK v3 (S3)
- Email         : Nodemailer + SendGrid
- Paiement      : Stripe Node.js SDK
- Logs          : Winston (src/utils/logger.ts)
- Tâches cron   : node-cron
- Sécurité      : helmet, express-rate-limit, cors

## Frontend
- Framework     : React 18.x
- Build         : Vite 4.x
- Langage       : TypeScript 5.x strict
- État global   : Redux Toolkit + react-redux
- Routage       : React Router v6
- Formulaires   : React Hook Form + Yup
- HTTP          : Axios (avec interceptors JWT)
- UI            : Tailwind CSS 3.x + Headless UI
- i18n          : react-i18next (FR prioritaire)

## Microservice IA (Python — dépôt séparé)
- Framework     : FastAPI
- OCR           : Tesseract 5.x
- ML            : scikit-learn / XGBoost
- Communication : HTTP REST (appelé par backend Node.js)
- Endpoint      : POST /analyze

## Tests
- Backend       : Jest + Supertest (cible couverture : 80% minimum)
- Frontend      : React Testing Library + Cypress (E2E)
- Helpers tests : src/__tests__/helpers.ts (createActiveUser, loginTestUser, etc.)

## CI/CD
- GitHub Actions (.github/workflows/ci.yml)
- Lint → Tests → Coverage → Build → Deploy staging


# ────────────────────────────────────────────────────────────
# 3. STRUCTURE DU PROJET [CRITIQUE]
# ────────────────────────────────────────────────────────────

pouraccord/
├── backend/
│   ├── src/
│   │   ├── config/          # env.ts, database.ts, s3.ts, stripe.ts
│   │   ├── controllers/     # AuthController, UserController, FolderController...
│   │   ├── middlewares/     # auth.middleware.ts, rateLimit.ts, audit.ts, upload.ts
│   │   ├── models/          # User.ts, Agency.ts, Folder.ts, Document.ts...
│   │   ├── routes/          # auth.routes.ts, users.routes.ts, folders.routes.ts...
│   │   ├── services/        # EmailService.ts, S3Service.ts, StripeService.ts, AIService.ts
│   │   ├── utils/           # logger.ts, response.ts, validators.ts, crypto.ts
│   │   ├── jobs/            # cleanupExpiredDocuments.ts, cleanupRefreshTokens.ts
│   │   ├── types/           # express.d.ts, index.ts
│   │   └── app.ts           # Point d'entrée Express
│   ├── migrations/          # Séquentiels : 001_create_users.ts, 002_create_agencies.ts...
│   ├── seeders/             # document_types.ts, admin_user.ts
│   └── __tests__/
│       ├── helpers.ts       # createActiveUser(), loginTestUser(), createTestAgency()...
│       ├── setup.ts         # Connexion DB test, migrations
│       └── teardown.ts      # Nettoyage DB test
│
├── frontend/
│   ├── src/
│   │   ├── components/      # Composants réutilisables (Button, Input, Modal...)
│   │   ├── pages/           # Register, Login, Dashboard, Documents, Settings...
│   │   ├── store/           # authSlice.ts, folderSlice.ts, store.ts
│   │   ├── services/        # api.ts (Axios instance), auth.service.ts...
│   │   ├── hooks/           # useAuth.ts, useFolder.ts, useDebounce.ts
│   │   ├── types/           # User.ts, Folder.ts, Document.ts, Agency.ts
│   │   └── utils/           # formatters.ts, validators.ts, constants.ts
│   └── cypress/             # Tests E2E
│
├── .cursorrules             # CE FICHIER
├── .cursor/mcp.json         # Configuration MCP (Trello, etc.)
└── .github/workflows/       # CI/CD


# ────────────────────────────────────────────────────────────
# 4. FORMAT DE RÉPONSE API [CRITIQUE — JAMAIS DÉROGER]
# ────────────────────────────────────────────────────────────

TOUTES les réponses API utilisent OBLIGATOIREMENT ce format :

```typescript
// Succès
{
  success: true,
  data: { ... },          // null si pas de données à retourner
  message: "...",         // message humain en français
  errors: []              // tableau vide en cas de succès
}

// Erreur
{
  success: false,
  data: null,
  message: "...",         // message humain en français
  errors: ["CODE_ERREUR"] // codes machine pour le frontend
}
```

Helpers à utiliser (src/utils/response.ts) :
```typescript
successResponse(res, data, message, statusCode = 200)
errorResponse(res, message, errors = [], statusCode = 400)
```

Codes HTTP à respecter :
- 200 : succès GET/PATCH
- 201 : ressource créée (POST)
- 204 : succès sans body (DELETE, logout)
- 400 : erreur validation input
- 401 : non authentifié (token absent/invalide/expiré)
- 403 : authentifié mais rôle insuffisant
- 404 : ressource inexistante
- 409 : conflit (email déjà utilisé, SIRET existant)
- 422 : entité non traitable
- 429 : trop de requêtes (rate limiting)
- 500 : erreur serveur inattendue


# ────────────────────────────────────────────────────────────
# 5. AUTHENTIFICATION & SÉCURITÉ [CRITIQUE]
# ────────────────────────────────────────────────────────────

## JWT
- Access token  : expire 24h (86400s), signé HS256
- Refresh token : expire 7j (604800s), UUID v4, stocké en BDD (table refresh_tokens)
- Payload JWT   : { sub: user.id, email, role, iat, exp }

## Middleware d'authentification (src/middlewares/auth.middleware.ts)
```typescript
authenticateToken     // Vérifie JWT Bearer — à appliquer sur tout endpoint protégé
requireRole('admin')  // Vérifie le rôle — ex: requireRole('admin', 'agency_owner')
```

## Rôles utilisateurs
- 'tenant'        : locataire (accès à son dossier uniquement)
- 'agency_owner'  : gérant d'agence (accès dossiers agence + billing)
- 'agency_agent'  : agent d'agence (accès dossiers agence)
- 'admin'         : administrateur plateforme (accès total)

## Rate limiting (à appliquer SYSTÉMATIQUEMENT sur endpoints sensibles)
- /auth/login           : 3 tentatives échouées / 15 min / IP
- /auth/register        : 5 tentatives / 15 min / IP
- /auth/forgot-password : 3 tentatives / 30 min / IP
- API générale          : 100 req / 15 min / IP

## DONNÉES SENSIBLES — JAMAIS EXPOSÉES dans les réponses API
- password_hash
- totp_secret
- token de vérification email
- token de reset password

## Sécurité fichiers uploadés
- Formats autorisés UNIQUEMENT : PDF, JPG, JPEG, PNG
- Taille max : 5 Mo par fichier
- Stockage : AWS S3 — chemin : /users/{user_id}/documents/{uuid}.{ext}
- Jamais d'exécution de fichiers uploadés


# ────────────────────────────────────────────────────────────
# 6. MODÈLE DE DONNÉES PRINCIPAL
# ────────────────────────────────────────────────────────────

## Table users
- id, email (UNIQUE), password_hash, role (ENUM), status (ENUM)
- status ENUM : 'pending_verification' | 'active' | 'suspended' | 'deleted'
- tenant_profile ENUM : 'employee_cdi' | 'employee_cdd' | 'student' | 'freelance' | 'retired' | 'other'
- totp_secret (chiffré AES-256), is_2fa_enabled, agency_id
- email_verified_at, last_login_at, created_at, updated_at, deleted_at (soft delete)

## Table agencies
- id, name, siret (UNIQUE, 14 chars), legal_name, address, phone
- status ENUM : 'trial' | 'active' | 'suspended' | 'cancelled'
- trial_ends_at, subscription_id (Stripe), customer_id (Stripe), next_billing_date

## Table folders (dossiers locataires)
- id, user_id (FK), status ENUM ('incomplete' | 'complete' | 'expired' | 'deleted')
- completion_percentage, ai_score_global, ai_status ('pending' | 'analyzing' | 'verified' | 'manual_review' | 'rejected')
- expires_at (6 mois après création — RGPD)

## Table documents
- id, folder_id (FK), document_type (FK document_types), file_path (S3)
- status ENUM : 'pending_analysis' | 'valid' | 'rejected' | 'expired'
- ai_score, ai_warnings (JSON), extracted_data (JSON chiffré)
- original_filename, file_size, mime_type, version (remplacement)

## Table sharing_links
- id, folder_id (FK), token (UNIQUE UUID), created_by (agency_id ou null)
- expires_at, max_views (null = illimité), view_count, is_active

## Table refresh_tokens
- id, user_id (FK), token (VARCHAR 512), expires_at, revoked_at

## Table audit_logs (JAMAIS supprimer — conservation 3 ans)
- id, user_id, agency_id, ip_address, action, entity_type, entity_id, details (JSON)


# ────────────────────────────────────────────────────────────
# 7. CONVENTIONS DE CODE [CRITIQUE]
# ────────────────────────────────────────────────────────────

## TypeScript
- Mode strict activé (tsconfig strict: true)
- Pas de `any` non justifié — utiliser les types définis dans src/types/
- Interfaces pour les objets de données, types pour les unions/primitives
- Exports nommés préférés aux exports default (sauf pages React)

## Nommage
- Fichiers controllers : PascalCase + suffixe Controller (ex: AuthController.ts)
- Fichiers routes     : kebab-case + suffixe .routes (ex: auth.routes.ts)
- Fichiers models     : PascalCase (ex: User.ts, RefreshToken.ts)
- Fonctions/variables : camelCase
- Constantes          : UPPER_SNAKE_CASE
- Types/Interfaces    : PascalCase avec préfixe I pour interfaces (IUser, IFolder)

## Gestion d'erreurs
- TOUJOURS utiliser try/catch dans les controllers
- Logger les erreurs avec logger.error() AVANT de retourner errorResponse()
- Ne JAMAIS exposer les stack traces en production
- Pattern standard :
```typescript
try {
  // logique métier
  return successResponse(res, data, 'Message succès');
} catch (error) {
  logger.error('Contexte erreur', { error, userId: req.user?.id });
  return errorResponse(res, 'Une erreur est survenue', ['INTERNAL_ERROR'], 500);
}
```

## Validation Joi — Pattern standard
```typescript
const schema = Joi.object({ ... });
const { error, value } = schema.validate(req.body, { abortEarly: false });
if (error) {
  const errors = error.details.map(d => d.message);
  return errorResponse(res, 'Données invalides', errors, 400);
}
```

## Sequelize
- Utiliser les modèles Sequelize — JAMAIS de requêtes SQL brutes sauf si absolument nécessaire
- Toujours utiliser les transactions pour les opérations multi-tables
- Exclure les champs sensibles des SELECT avec `attributes: { exclude: ['password_hash', 'totp_secret'] }`

## Logs Winston — Niveaux à respecter
- logger.error()  : erreurs inattendues, exceptions
- logger.warn()   : comportements suspects, rate limiting déclenché
- logger.info()   : actions métier importantes (login, upload, partage)
- logger.debug()  : infos de développement (désactivé en production)


# ────────────────────────────────────────────────────────────
# 8. CONVENTIONS DE TESTS [CRITIQUE]
# ────────────────────────────────────────────────────────────

## Règle d'or : TDD — écrire les tests AVANT l'implémentation

## Structure des tests backend
```typescript
describe('NOM_ENDPOINT ou NOM_FONCTIONNALITÉ', () => {
  // Setup
  beforeAll(async () => { /* connexion DB, seed data */ });
  afterEach(async () => { /* nettoyer les données créées */ });
  afterAll(async () => { /* fermer connexions */ });

  // Tests dans cet ordre :
  test('should [comportement nominal]', async () => { ... });
  test('should return 400 if [validation échoue]', async () => { ... });
  test('should return 401 if [non authentifié]', async () => { ... });
  test('should return 403 if [rôle insuffisant]', async () => { ... });
  test('should return 404 if [ressource inexistante]', async () => { ... });
  test('should return 409 if [conflit]', async () => { ... });
  test('should return 429 if [rate limit atteint]', async () => { ... });
});
```

## Helpers disponibles (src/__tests__/helpers.ts)
```typescript
createActiveUser(email?, password?)     // Crée un user actif
createPendingUser(email?, password?)    // Crée un user non vérifié
loginTestUser(role?)                    // Retourne { accessToken, refreshToken, user }
createTestAgency()                      // Crée une agence en trial
createTestFolder(userId)               // Crée un dossier pour un locataire
createPasswordResetToken(userId)       // Crée un token de reset valide
```

## Règle de sécurité des tests
- JAMAIS vérifier qu'un champ sensible est présent
- TOUJOURS vérifier qu'il est ABSENT :
```typescript
expect(res.body.data.password_hash).toBeUndefined(); // ✅
expect(res.body.data.totp_secret).toBeUndefined();   // ✅
```

## Couverture minimale requise : 80%
- 100% sur les modules auth et RGPD (critique)
- 80% sur les modules métier
- Rapport : npm run test:coverage


# ────────────────────────────────────────────────────────────
# 9. RGPD — RÈGLES ABSOLUES [CRITIQUE]
# ────────────────────────────────────────────────────────────

Ces règles ne sont JAMAIS négociables :

1. CONSENTEMENT : tout accès données locataire nécessite consentement explicite tracé
   - Table user_consents : consent_type, consented_at, ip_address

2. DURÉE DE VIE : dossiers supprimés automatiquement après 6 mois (CRON quotidien)
   - Alerte email 30j avant expiration
   - Job : src/jobs/cleanupExpiredDocuments.ts

3. SUPPRESSION : endpoint DELETE /users/me
   - Soft delete user (deleted_at)
   - Suppression physique fichiers S3 (async)
   - Hard delete extracted_data (données sensibles)
   - Anonymisation audit_logs (user_id → '<deleted>')
   - Délai de grâce : 30 jours avant hard delete définitif

4. AUDIT : tout accès à un dossier est loggé (table audit_logs)
   - Conservation minimum : 3 ans
   - JAMAIS supprimer des audit_logs

5. EXPORT : endpoint GET /users/me/data-export
   - ZIP contenant JSON de toutes les données + fichiers S3
   - Presigned URL S3 (expire 24h)

6. WATERMARKING : tout PDF téléchargé par une agence porte un watermark
   - Visible : nom agence + date
   - Invisible (stéganographie LSB) : identifiant unique agence/agent


# ────────────────────────────────────────────────────────────
# 10. MODULE IA ANTI-FRAUDE — RAPPEL ARCHITECTURE
# ────────────────────────────────────────────────────────────

Le microservice IA est un service Python séparé.
Communication : backend Node.js → POST http://ai-service/analyze

8 niveaux d'analyse :
1. Exhaustivité     : documents obligatoires présents selon profil ?
2. Conformité       : format valide, lisible, OCR possible ?
3. Validité         : NIR valide, MRZ correcte, SIRET existant ?
4. Authenticité     : employeur vérifié INSEE, adresse vérifiée API gouv ?
5. Cohérence intra  : cohérence interne d'un même document
6. Cohérence inter  : cohérence entre documents (nom, dates, revenus)
7. Intégrité        : métadonnées PDF, détection altérations visuelles (ELA)
8. Adéquation       : revenus ≥ 3x loyer ?

Scores : 0-100 par document + score global dossier
Status : 'verified' (≥85) | 'manual_review' (60-84) | 'rejected' (<60)

APIs externes utilisées par le microservice IA :
- INSEE SIRET  : https://api.insee.fr/entreprises/sirene/V3/siret/{SIRET}
- API Adresse  : https://api-adresse.data.gouv.fr/search/
- MRZ parsing  : librairie mrz (Python)


# ────────────────────────────────────────────────────────────
# 11. PROCESSUS DE DÉVELOPPEMENT AVEC CURSOR
# ────────────────────────────────────────────────────────────

## Avant de coder
1. Lire la carte Trello correspondante (via MCP) pour avoir le contexte complet
2. Vérifier les dépendances — le ticket indique les tickets prérequis
3. Identifier les tables BDD touchées

## Ordre de travail (TDD)
1. Écrire les tests (describe + tous les cas)
2. Lancer les tests → ils doivent être ROUGES
3. Implémenter le code minimal pour les faire passer
4. Refactoriser si besoin
5. npm run lint → zéro erreur
6. npm run test:coverage → ≥ 80%

## Définition de "Done"
- ✅ Tous les critères d'acceptance de la carte Trello sont satisfaits
- ✅ npm test → 100% passing
- ✅ Coverage ≥ 80% sur le nouveau code
- ✅ npm run lint → zéro erreur
- ✅ npm run build (frontend) → zéro erreur TypeScript
- ✅ CI GitHub Actions verte
- ✅ Aucune donnée sensible exposée dans les réponses
- ✅ Carte Trello déplacée en "Done"

## Quand utiliser Agent Mode vs Ask Mode
- Agent Mode  → implémenter une feature, créer des fichiers, écrire des tests
- Ask Mode    → comprendre du code existant, planifier, poser des questions d'archi

## Parallélisation (multi-agents)
- Tickets indépendants peuvent tourner en parallèle (ex: AUTH-backend + AUTH-frontend)
- Vérifier les dépendances avant de paralléliser
- Merger dans l'ordre des dépendances


# ────────────────────────────────────────────────────────────
# 12. VARIABLES D'ENVIRONNEMENT — RÉFÉRENCE
# ────────────────────────────────────────────────────────────

Toutes les variables sont définies dans .env (local) / .env.test (tests).
Elles sont validées au démarrage via Joi dans src/config/env.ts.
JAMAIS de valeur en dur dans le code.

Variables critiques disponibles via process.env :
- DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD
- JWT_SECRET (min 32 chars), JWT_EXPIRES_IN
- JWT_REFRESH_SECRET (min 32 chars), JWT_REFRESH_EXPIRES_IN
- AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, AWS_S3_BUCKET
- SENDGRID_API_KEY, EMAIL_FROM, EMAIL_FROM_NAME
- TOTP_SECRET_ENCRYPTION_KEY (32 chars — AES-256 pour chiffrer les secrets TOTP)
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- AI_SERVICE_URL (URL du microservice Python)
- FRONTEND_URL (pour CORS et liens emails)


# ────────────────────────────────────────────────────────────
# 13. SPRINTS EN COURS
# ────────────────────────────────────────────────────────────

Sprint 0 — Fondations (semaine 1-2)
  SETUP-01 : Monorepo backend + frontend
  SETUP-02 : Variables d'environnement + validation Joi
  SETUP-03 : Migrations BDD (users, agencies, refresh_tokens, email_verification_tokens)
  SETUP-04 : GET /health + Logger Winston
  SETUP-05 : CI/CD GitHub Actions

Sprint 1 — Authentification (semaine 3-4)
  AUTH-01 : POST /auth/register
  AUTH-02 : POST /auth/verify-email
  AUTH-03 : POST /auth/login (JWT + refresh token)
  AUTH-04 : POST /auth/refresh + POST /auth/logout
  AUTH-05 : POST /auth/forgot-password + /auth/reset-password
  AUTH-06 : Middleware JWT + GET /users/me + PATCH /users/me
  AUTH-07 : 2FA locataire (enable + verify + disable)
  AUTH-08 : Frontend React (Register, Login, VerifyEmail pages + Redux auth slice)

Sprints à venir : Module Locataire → Module Agence → Module IA → Admin + RGPD

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FredSBAF) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
