## tutorisk-backend

> Plateforme e-learning pour TutoRisk (ABC Sécurité), organisme de formation en gestion

# TutoRisk — LCMS (backend + frontend React)

Plateforme e-learning pour TutoRisk (ABC Sécurité), organisme de formation en gestion
des risques depuis 2002. Ce dépôt contient l'API **et** le frontend React du LCMS.

- Site vitrine : dépôt séparé `tutorisk/tutorisk-site` → https://tutorisk.net
- LCMS (ce dépôt) → https://lcms.tutorisk.net

## Structure

- `src/` — API Node.js / Express / PostgreSQL
  - `controllers/`, `routes/`, `middleware/`, `db/` (schema.sql, migrate.js, seed.js)
- `client/` — frontend React (Vite)
  - `src/App.jsx` — **toute l'application en un seul fichier (~4700 lignes)**
  - `public/` — favicons, logos
- `deploy.sh` — script d'installation initiale du VPS (Ubuntu 24.04)

## Commandes

```bash
# API
npm install && npm run migrate    # migrate est idempotent (CREATE ... IF NOT EXISTS)
node src/server.js                # port 4000 ; GET /health → {"status":"ok"}

# Frontend
cd client && npm install && npm run build   # sort dans client/dist/
```

Ne **jamais** lancer `npm run seed` sur la production : il efface et recrée les données de démo.

## Rôles applicatifs

`admin`, `pedagogue`, `formateur`, `charge` (chargé de formation), `apprenant`.
Chaque rôle a son tableau de bord dans `App.jsx`.

## Production (VPS Ikoula, Ubuntu 24.04)

- API : service systemd `tutorisk` (`journalctl -u tutorisk -f` pour les logs)
- Frontend servi en statique depuis `/var/www/tutorisk-lcms-frontend`
- nginx fait proxy de `/api` vers `127.0.0.1:4000`
- Secrets dans `/var/www/tutorisk-backend/.env` (jamais dans git)

### Déploiement

```bash
cd /var/www/tutorisk-backend && git fetch origin && git reset --hard origin/main && \
cd client && npm run build && \
rm -rf /var/www/tutorisk-lcms-frontend/* && cp -r dist/* /var/www/tutorisk-lcms-frontend/ && \
sed -i 's|</head>|<script>window.TUTORISK_API_BASE="https://lcms.tutorisk.net";</script></head>|' \
  /var/www/tutorisk-lcms-frontend/index.html && \
chown -R www-data:www-data /var/www/tutorisk-lcms-frontend && systemctl reload nginx
```

## Pièges connus (appris à la dure — ne pas les refaire)

1. **`window.TUTORISK_API_BASE` doit être injecté dans `index.html` après chaque build.**
   `App.jsx` fait : `const API_BASE = window.TUTORISK_API_BASE || "http://localhost:4000"`.
   Sans injection, la prod appelle localhost et tout casse. À injecter dans le `<head>`.

2. **`git` refuse les dossiers `/var/www/*` (« dubious ownership »)** car ils appartiennent
   à `www-data`. Il faut `git config --global --add safe.directory /var/www/<dossier>`.
   Sans ça, les commandes git échouent en silence au milieu d'une chaîne `&&`.

3. **Les fichiers en prod sont modifiés par `sed` après déploiement** → `git pull` est bloqué.
   Toujours utiliser `git fetch && git reset --hard origin/main`, puis réappliquer les `sed`.

4. **Cache favicon** : les navigateurs ignorent `?v=N` pour les favicons. Pour en forcer un
   nouveau, il faut **changer le nom du fichier** (d'où `icon-32.png` et non `favicon-32x32.png`).

5. **Ne jamais utiliser `types { ... }` dans nginx** : cette directive *remplace* toute la table
   MIME au lieu de la compléter → tout le site part en `application/octet-stream` (téléchargement
   au lieu d'affichage). Utiliser `default_type` dans une `location` dédiée.

## Dette technique à traiter (par ordre de gain)

1. **`client/src/App.jsx` = 4700 lignes en un fichier.** À découper par domaine
   (auth, catalogue, admin, analytics, pages corporate…).
2. **Aucun code splitting** : bundle unique de ~424 Ko (114 Ko gzip). Un apprenant télécharge
   tout le code admin. `React.lazy` par rôle diviserait le chargement initial par 3-4.
3. **Styles 100 % inline**, sans tokens ni système de design → homogénéité difficile à tenir.

## Règles de travail

- Toujours lancer `cd client && npm run build` après modification de `App.jsx` : le fichier est
  gros, une erreur JSX ne se voit pas à l'œil nu.
- Tester une config nginx avec `nginx -t` **avant** `systemctl reload`.
- Après tout changement d'infra, vérifier avec `curl -sI` que le `Content-Type` est correct.

---
> Source: [tutorisk/tutorisk-backend](https://github.com/tutorisk/tutorisk-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
