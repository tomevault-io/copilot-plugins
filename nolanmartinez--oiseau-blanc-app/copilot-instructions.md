## oiseau-blanc-app

> Application web pour **L'Oiseau Blanc Traiteur** permettant de gérer des frigos connectés installés en entreprise.

# CLAUDE.md — L'Oiseau Blanc Traiteur - Application Frigos Connectés

## Description du projet

Application web pour **L'Oiseau Blanc Traiteur** permettant de gérer des frigos connectés installés en entreprise.
Deux interfaces distinctes :
- **Panel Admin** : gestion utilisateurs, dashboard analytics, notifications, connexion Bicom
- **Panel Public** : consultation des plats, dépôt d'avis, préférences alimentaires, inscription notifications

Le client est Frédéric Bartoli, gérant de L'Oiseau Blanc Traiteur.

## Stack technique

- **Frontend** : React (Vite), React Router, Tailwind CSS
- **Backend** : Node.js, Express
- **Base de données** : PostgreSQL
- **ORM** : Prisma
- **Auth admin** : JWT + bcrypt
- **Emails** : Mailchimp API (déjà utilisé par le client)
- **Push notifications** : Web Push (service workers) — à confirmer
- **API externe** : Bicom (lecture seule, clé API en attente)

## Structure du projet

```
oiseau-blanc-app/
├── CLAUDE.md
├── client/                  # Frontend React
│   ├── src/
│   │   ├── components/      # Composants réutilisables
│   │   ├── pages/
│   │   │   ├── admin/       # Pages panel admin
│   │   │   └── public/      # Pages panel public
│   │   ├── hooks/           # Custom hooks
│   │   ├── services/        # Appels API (axios)
│   │   ├── context/         # Auth context, etc.
│   │   └── utils/
│   └── package.json
├── server/                  # Backend Node.js/Express
│   ├── src/
│   │   ├── routes/          # Routes API
│   │   ├── controllers/     # Logique métier
│   │   ├── middleware/      # Auth, validation, error handling
│   │   ├── services/        # Bicom, Mailchimp, push
│   │   └── utils/
│   ├── prisma/
│   │   └── schema.prisma    # Schéma BDD
│   └── package.json
├── database/                # Scripts SQL complémentaires, seeds
└── README.md
```

## Modèle de données

### admins
- id (UUID, PK)
- email (unique, not null)
- password_hash (not null)
- role (enum: SUPER_ADMIN, ADMIN)
- created_at, updated_at

### subscribers
Utilisateurs du panel public. Pas de mot de passe, identification par email/téléphone.
- id (UUID, PK)
- email (unique, nullable)
- phone (nullable)
- consent_email (boolean, default false)
- consent_push (boolean, default false)
- push_token (nullable)
- created_at, updated_at

Contrainte : au moins email OU phone requis.

### reviews
- id (UUID, PK)
- subscriber_id (FK → subscribers, not null)
- dish_id (string, référence Bicom)
- rating (integer, 1-5)
- comment (text, nullable)
- created_at

### preference_surveys
Sondages créés par l'admin (questions récurrentes type qualité/prix, quantité).
- id (UUID, PK)
- title (not null)
- questions (JSONB) — structure : [{id, label, type, options}]
- active (boolean)
- created_at, updated_at

### preference_responses
- id (UUID, PK)
- survey_id (FK → preference_surveys)
- subscriber_id (FK → subscribers)
- answers (JSONB)
- created_at

### menu_votes
Votes sur les menus à venir (ex: "dans 10 jours, couscous ou penne ?").
- id (UUID, PK)
- title (not null)
- options (JSONB) — liste des plats proposés
- vote_deadline (timestamp)
- created_at

### menu_vote_responses
- id (UUID, PK)
- menu_vote_id (FK → menu_votes)
- subscriber_id (FK → subscribers)
- selected_options (JSONB)
- created_at

## Fonctionnalités par lot

### Panel Admin
1. **Auth admin** — login JWT, gestion comptes admin, reset password
2. **Gestion utilisateurs** — CRUD admins, visualisation subscribers
3. **Connexion Bicom** — ⚠️ EN ATTENTE (clé API pas encore reçue). Préparer un service Bicom avec des données mockées en attendant. Lecture seule : frigos, plats, stocks, prix.
4. **Notifications** — intégration Mailchimp API (ajout contacts, déclenchement campagnes), fréquence configurable par l'admin
5. **Dashboard** — moyennes par plat, tendances avis, résultats sondages/votes. Export CSV/PDF (nice to have).
6. **Gestion sondages** — création/modification des formulaires de préférences
7. **Gestion votes menus** — proposer des menus futurs, consulter les résultats

### Panel Public (sans compte)
1. **Interface principale** — sélection frigo, liste plats disponibles, fiche détaillée
2. **Dépôt d'avis** — formulaire notation + commentaire, email ou téléphone obligatoire
3. **Préférences** — réponse aux sondages actifs
4. **Vote menus** — choix parmi les menus proposés
5. **Inscription notifications** — collecte email, consentement push/email

## Conventions de code

- **Langue du code** : anglais (variables, fonctions, composants)
- **Langue des contenus/UI** : français
- **Nommage** : camelCase (JS), PascalCase (composants React), snake_case (colonnes BDD)
- **API REST** : préfixe `/api/v1/`, JSON
- **Routes admin** : `/api/v1/admin/*` (protégées par middleware JWT)
- **Routes public** : `/api/v1/public/*` (ouvertes ou auth légère par email)
- **Gestion erreurs** : middleware centralisé, codes HTTP standards
- **Validation** : Zod côté serveur
- **Pas de console.log en production** — utiliser un logger (winston ou pino)

## Variables d'environnement attendues

```env
# Server
PORT=3001
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/oiseau_blanc
JWT_SECRET=

# Bicom (en attente)
BICOM_API_URL=
BICOM_API_KEY=

# Mailchimp
MAILCHIMP_API_KEY=
MAILCHIMP_SERVER_PREFIX=
MAILCHIMP_LIST_ID=

# Push (à confirmer)
VAPID_PUBLIC_KEY=
VAPID_PRIVATE_KEY=
```

## Points bloqués / à confirmer

- [ ] Clé API Bicom — en attente de l'éditeur
- [ ] Documentation API Bicom — obtenir un accès pour explorer les données
- [ ] Push notifications — web push ou mobile ? À confirmer avec Frédéric
- [ ] Hébergement — probablement OVH, à clarifier
- [ ] Indicateurs dashboard — à définir précisément

## Ordre de développement recommandé

1. ✅ Setup projet (Vite + Express + Prisma + PostgreSQL)
2. ✅ Auth admin (JWT, login, CRUD admins)
3. Modèle subscribers + collecte email/téléphone
4. Système d'avis (formulaire public + stockage)
5. Sondages préférences (admin: création / public: réponse)
6. Votes menus (admin: création / public: vote)
7. Intégration Bicom (quand API dispo, mock en attendant)
8. Intégration Mailchimp (sync contacts + campagnes)
9. Dashboard admin (analytics, graphiques, exports)
10. Notifications push (si web app confirmée)

---

## Ce qui a été développé — descriptions détaillées

### Lot 1 — Setup projet

**`server/src/index.ts`**
Point d'entrée du serveur Express. Configure CORS (autorise uniquement le client React sur :5173), parse le JSON, monte les routes, et démarre le serveur sur le port défini dans `.env`. Contient aussi une route `/api/v1/health` pour vérifier que le serveur tourne.

**`server/src/utils/logger.ts`**
Logger basé sur `pino`. En développement, affiche des logs colorés et lisibles dans le terminal. En production, sort du JSON brut (compatible avec les agrégateurs de logs type Datadog, Logtail). Remplace tous les `console.log`.

**`server/src/utils/prisma.ts`**
Instance unique du client Prisma partagée dans tout le serveur. Évite d'ouvrir plusieurs connexions à la base de données.

**`server/prisma/schema.prisma`**
Définit toutes les tables de la base de données : `admins`, `subscribers`, `reviews`, `preference_surveys`, `preference_responses`, `menu_votes`, `menu_vote_responses`. C'est la source de vérité du modèle de données — toute modification ici se propage à PostgreSQL via `npm run db:migrate`.

**`client/vite.config.ts`**
Configure Vite avec le plugin Tailwind CSS v4 et un proxy : toutes les requêtes `/api/*` émises par le frontend sont automatiquement redirigées vers `http://localhost:3001`. Évite les problèmes CORS en développement.

**`client/src/services/api.ts`**
Instance axios préconfigurée avec `baseURL: /api/v1`. Intercepteur qui injecte automatiquement le token JWT (`Authorization: Bearer ...`) dans chaque requête si l'admin est connecté. Tous les appels API du projet passent par ce fichier.

---

### Lot 2 — Auth admin

**`server/src/middleware/auth.ts`**
Middleware Express qui vérifie le JWT sur les routes protégées. Lit le header `Authorization: Bearer <token>`, le décode et attache le payload (`id`, `email`, `role`) à `req.admin`. Contient aussi `requireSuperAdmin` pour restreindre certaines routes aux super admins uniquement.

**`server/src/controllers/auth.controller.ts`**
- `POST /api/v1/admin/auth/login` : vérifie email + mot de passe (bcrypt), retourne un JWT valable 8h + les infos de l'admin.
- `GET /api/v1/admin/auth/me` : retourne les infos de l'admin connecté à partir du token (route protégée).

**`server/src/controllers/admin.controller.ts`**
CRUD complet sur les comptes admin, accessible uniquement aux SUPER_ADMIN :
- `GET /api/v1/admin/admins` — liste tous les admins
- `POST /api/v1/admin/admins` — crée un nouvel admin (hash le mot de passe avec bcrypt)
- `PATCH /api/v1/admin/admins/:id` — modifie email / mot de passe / rôle
- `DELETE /api/v1/admin/admins/:id` — supprime un admin (interdit de se supprimer soi-même)

**`server/prisma/seed.ts`**
Script à exécuter une seule fois (`npm run db:seed`) pour créer le premier compte SUPER_ADMIN (`admin@oiseaublanc.fr` / `Admin1234!`). Idempotent : ne recrée pas le compte s'il existe déjà.

**`client/src/context/AuthContext.tsx`**
Context React qui gère la session admin côté frontend. Au chargement, tente de restaurer la session depuis le `localStorage` (appel `GET /me`). Expose `login()`, `logout()`, et l'objet `admin` courant à tous les composants de l'app.

**`client/src/components/PrivateRoute.tsx`**
Composant wrapper qui redirige vers `/admin/login` si l'utilisateur n'est pas authentifié. Affiche un loader pendant la vérification du token. À utiliser autour de toutes les pages du panel admin.

**`client/src/pages/admin/Login.tsx`**
Page de connexion admin. Formulaire email + mot de passe, appel à `AuthContext.login()`, redirection vers `/admin/dashboard` en cas de succès, affichage d'une erreur en cas d'échec.

---

## Ports de développement

| Service | URL |
|---|---|
| Frontend React (Vite) | http://localhost:5173 |
| Backend Express | http://localhost:3001 |
| Health check API | http://localhost:3001/api/v1/health |
| Prisma Studio (BDD) | http://localhost:5555 (via `npm run db:studio`) |

> ⚠️ La page admin se trouve sur **:5173/admin/login** (React), pas sur :3001 (Express).

---
> Source: [NolanMartinez/oiseau-blanc-app](https://github.com/NolanMartinez/oiseau-blanc-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
