## audio-accessibility-system

> Système de diffusion audio pour salles de spectacle (~450 places), destiné à des personnes en situation de handicap sensoriel (malentendants, déficients visuels). Toute modification doit respecter les trois principes suivants de manière non négociable.


# Règles du workspace — Audio Accessibility System

## Contexte

Système de diffusion audio pour salles de spectacle (~450 places), destiné à des personnes en situation de handicap sensoriel (malentendants, déficients visuels). Toute modification doit respecter les trois principes suivants de manière non négociable.

---

## 1. Sécurité (Security by Design)

### Content Security Policy (CSP)
- Toute page HTML servie doit inclure un header `Content-Security-Policy` strict.
- Les scripts inline et styles inline doivent utiliser un **nonce** généré côté serveur (jamais `'unsafe-inline'`).
- Le nonce doit être injecté sur **tous** les `<script>` et `<style>`, y compris ceux avec attribut `src=`.
- Les attributs `on*` inline (`onclick`, `onsubmit`, etc.) sont **interdits** — utiliser `addEventListener` et délégation d'événements.
- Les `style=` inline sont **interdits** — utiliser `element.style.setProperty()` ou des classes CSS.

### Directives CSP minimales requises
- `script-src 'self' 'nonce-{nonce}' blob:` — blob: requis pour HLS.js
- `worker-src 'self' blob:` — 'self' pour le service worker, blob: pour HLS.js
- `media-src 'self' blob:` — blob: requis pour la lecture audio HLS
- `connect-src 'self' wss://{host} https://{host}` — WebSocket + API
- `object-src 'none'`, `frame-src 'none'`, `base-uri 'self'`

### Authentification & sessions
- Le mot de passe admin est hashé avec bcrypt (rounds ≥ 12).
- Les tokens admin sont des JWT signés avec un secret fort (≥ 32 chars), expirés en 8h max.
- Aucun credential ne doit transiter en query param (URL). Les formulaires n'ont pas d'attribut `name` sur les champs sensibles pour éviter la soumission GET par le navigateur.
- L'interface admin est protégée par `requireAdmin` sur toutes les routes `/api/admin/*`.

### Validation des entrées
- Tous les inputs API passent par le middleware `validate.js` (whitelist stricte, sanitisation, protection path traversal).
- Les chemins de fichiers sont validés contre des préfixes autorisés (`/app/uploads/`, `/app/sdp/`).
- Les uploads sont limités en taille et en type MIME.

### TLS
- HTTPS obligatoire (port 8443). TLS 1.2 minimum, TLS 1.3 recommandé.
- Ciphers ANSSI-compatibles : ECDHE + AES-GCM/CHACHA20. Pas de RC4, 3DES, export.
- Le certificat doit inclure un SAN (`subjectAltName`) pour l'IP ou le hostname cible.
- Les clients doivent accepter le certificat (ou le faire approuver en CA de confiance) avant que HLS.js puisse charger les segments.

### HLS.js
- `enableWorker: false` — le contexte Web Worker avec certificat auto-signé produit des erreurs SSL non récupérables.
- Ordre d'initialisation : `hls.loadSource(url)` puis `hls.attachMedia(audio)`.

---

## 2. Accessibilité (A11y by Design)

### Principes généraux
- Ce système est un **outil d'accessibilité** pour des personnes malentendantes, déficientes visuelles ou en situation de handicap cognitif. L'interface utilisateur doit être utilisable sans friction, y compris au premier usage avec un smartphone prêté à l'entrée.

### Interface auditeur (`index.html` — PWA)
- Tous les éléments interactifs ont un `aria-label` explicite et un `role` approprié.
- Les boutons de lecture indiquent leur état via `aria-pressed` et `aria-label` mis à jour dynamiquement.
- Le contraste couleur respecte WCAG 2.1 AA (ratio ≥ 4.5:1 pour le texte normal).
- La PWA est installable (manifest + service worker) pour un accès sans navigateur visible.
- Le QR code à l'entrée est le seul point d'entrée — aucune app à installer, aucun compte à créer.
- Les toasts d'information sont annoncés via `aria-live` pour les lecteurs d'écran.

### Interface admin (`admin.html`)
- L'interface admin n'est pas destinée au grand public — niveau A11y standard suffisant.
- Conserver les `label` associés aux `input` via `for`/`id`.

### Sources audio
- Le système supporte plusieurs canaux simultanés : audiodescription, renforcement pour malentendants, interprétation simultanée multilingue.
- Ne jamais supprimer ou fusionner des canaux sans validation explicite de l'opérateur.

### Conformité légale
- **Loi du 11 février 2005** — accessibilité des ERP (établissements recevant du public).
- **Décret 2014-1332** — accessibilité des salles de spectacle.
- Recommandation complémentaire : boucle magnétique T (norme IEC 60118-4) pour porteurs d'appareils auditifs.

---

## 3. RGPD by Design

### Principe général
- Aucune donnée personnelle n'est collectée, stockée ou transmise. Ce principe est non négociable et doit être maintenu dans toute évolution du système.

### Service Worker (`sw.js`)
- Seuls les assets statiques sans données personnelles sont mis en cache (`hls.min.js`, `manifest.json`).
- Les flux HLS (`/hls/*`), les réponses API (`/api/*`), les pages HTML et les WebSockets ne sont **jamais** mis en cache.
- Aucun cookie, aucun identifiant utilisateur ne transite par le service worker.

### API publique
- L'endpoint `/api/channels` ne retourne que : id, nom, description, langue, icône, couleur, nombre d'auditeurs (agrégé, non individuel).
- Le comptage des auditeurs est anonyme et agrégé — aucun identifiant de session n'est exposé.

### Logs serveur
- Les logs d'accès ne doivent pas persister les adresses IP au-delà de la session en cours.
- Aucun log ne doit contenir de données permettant d'identifier un auditeur individuel.

### Cookies & sessions
- Aucun cookie de tracking, analytics ou publicité.
- Le token admin est stocké en mémoire JavaScript (variable), pas dans `localStorage` ni `sessionStorage` — il disparaît à la fermeture de l'onglet.

### Minimisation des données
- Ne jamais ajouter de champs de profil, d'historique d'écoute, de géolocalisation ou de fingerprinting.
- Toute nouvelle fonctionnalité doit être évaluée à l'aune de ce principe avant implémentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dewiweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
