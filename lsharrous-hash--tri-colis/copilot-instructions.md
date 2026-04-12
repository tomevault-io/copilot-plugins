## tri-colis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Vue d'ensemble du projet

**Tri-Colis** est un système complet de tri de colis pour les sous-traitants dans le secteur de la livraison. Il permet aux livreurs et trieurs de scanner efficacement les colis via QR codes/codes-barres, de visualiser les numéros de commande et les chauffeurs assignés.

## Stack technique

### Backend
- **Runtime**: Node.js avec Express 5.2.1
- **Base de données**: Fichiers JSON (data.json, users.json, sessions.json)
- **Librairies clés**: pdf-parse, xlsx, multer, bcryptjs, cors
- **Port**: 3000
- **Déploiement**: Render (https://trizee.onrender.com)

### Web Frontend
- **Framework**: React 19 avec TypeScript
- **Build**: Vite 7.2.4
- **HTTP Client**: Axios + React Query
- **Routing**: React Router DOM

### Mobile Frontend
- **Framework**: React Native 0.81.5 avec Expo 54
- **Fonctionnalités**: expo-camera (scan QR), expo-document-picker, expo-secure-store
- **Package Android**: com.jnr.tricolis

## Structure du projet

```
tri-colis/
├── backend/
│   ├── index.js          # Serveur Express principal (API complète)
│   ├── importSpoke.js    # Parser pour fichiers Spoke
│   ├── data.json         # Base de données (tournées + colis)
│   ├── users.json        # Utilisateurs et rôles
│   └── sessions.json     # Sessions actives
│
├── web/src/
│   ├── pages/            # Pages React (Dashboard, Tours, Import, Users...)
│   ├── contexts/         # AuthContext (gestion auth globale)
│   └── lib/              # API clients (api.ts, colisApi.ts)
│
├── mobile/
│   ├── App.tsx           # Logique principale (fichier monolithique)
│   └── components/       # Button, Card, Input, ModalView
```

## Modèles de données

### Utilisateur
```json
{
  "id": number,
  "login": string,
  "role": "ADMIN" | "DISPATCHER" | "TRIEUR",
  "sousTraitantName": string | null,
  "passwordHash": string
}
```

### Tournée (Tour)
```json
{
  "id": number,
  "date": "YYYY-MM-DD",
  "chauffeurName": string,
  "sousTraitantName": string | null,
  "tourneeName": string,
  "colisCount": number,
  "sourceFile": string,
  "isCaniao": boolean
}
```

### Colis
```json
{
  "trackingNumber": string,
  "orderNumber": number,
  "chauffeurName": string,
  "sousTraitantName": string,
  "address": string,
  "tourId": number,
  "date": string
}
```

## Endpoints API principaux

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| POST | `/api/login` | Authentification |
| GET | `/api/colis` | Liste des colis (filtre par date) |
| POST | `/api/scan` | Enregistrer un scan |
| GET | `/api/tours` | Liste des tournées |
| POST | `/api/tours/import` | Importer une tournée |
| DELETE | `/api/tours/:id` | Supprimer une tournée |
| GET/POST | `/api/users` | Gestion utilisateurs (ADMIN) |

## Rôles utilisateurs

- **ADMIN**: Accès complet (gestion utilisateurs, toutes tournées)
- **DISPATCHER**: Import de tournées, gestion de ses propres tournées
- **TRIEUR**: Scan de colis uniquement

## Commandes de développement

### Backend
```bash
cd backend && npm install   # Installation des dépendances
cd backend && npm start     # Démarre le serveur Express sur port 3000
```

### Web
```bash
cd web && npm install       # Installation des dépendances
cd web && npm run dev       # Démarre Vite dev server avec HMR
cd web && npm run build     # Build de production
cd web && npm run lint      # ESLint
```

### Mobile
```bash
cd mobile && npm install    # Installation des dépendances
cd mobile && npx expo start --port 8084  # Démarre Expo dev server
cd mobile && npx expo start --android    # Lancement Android
```

## Configuration

### Production
- `DEV_MODE=false` dans `mobile/App.tsx`
- API: https://trizee.onrender.com
- Web `.env`: `VITE_API_BASE_URL=https://trizee.onrender.com`

### Développement
- `DEV_MODE=true` dans `mobile/App.tsx` (modifier `API_BASE_URL`)
- Backend local: http://localhost:3000
- Web `.env`: `VITE_API_BASE_URL=http://localhost:3000`

## Architecture

### Flux d'authentification
1. Login via `/api/login` retourne un token JWT
2. Token stocké dans `localStorage` (web) ou `SecureStore` (mobile)
3. Token envoyé dans header `Authorization: Bearer <token>`
4. Intercepteur 401 déclenche logout automatique (web: événement `tricoli:auth-logout`)

### Import de tournées
- **Fichiers Spoke** (.txt/.pdf): Parsés via `importSpoke.js`, regex pour extraction tracking numbers
- **Fichiers Excel** (.xlsx): Parsés via bibliothèque `xlsx`
- **Caniao**: Format spécifique avec endpoint dédié `/api/tours/caniao`

### Données
Les données sont persistées dans des fichiers JSON (`backend/data.json`, `users.json`, `sessions.json`). Pas de base de données traditionnelle.

## Points d'attention

1. **Parsing PDF Spoke**: L'extraction d'adresses depuis les PDF Spoke ne fonctionne pas parfaitement (affiche "Adresse à vérifier")
2. **Format Excel**: Le format Louis.xlsx utilise des noms de colonnes différents
3. **Déduplication**: Logique de déduplication pour le comptage des colis scannés
4. **Mobile App.tsx**: Fichier monolithique (~4000+ lignes), contient toute la logique de l'app mobile

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lsharrous-hash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lsharrous-hash)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
