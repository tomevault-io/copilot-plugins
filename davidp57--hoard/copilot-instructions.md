## hoard

> Application créée pour remplacer **nPlayer** (iPad, accès SMB au NAS) sur un **laptop Windows 11 tactile**.

# Hoard — Contexte projet

## Origine du projet

Application créée pour remplacer **nPlayer** (iPad, accès SMB au NAS) sur un **laptop Windows 11 tactile**.

Le besoin non couvert par les outils existants (PotPlayer, Jellyfin, Plex, Kodi) :
- Browser de **filesystem brut** (pas une bibliothèque avec métadonnées)
- **État de lecture visible dans la liste** : non vu / en cours (% + barre) / vu
- **Gestion de fichiers intégrée** : déplacer, supprimer
- **Gestes tactiles** sur le player
- Tout dans **une seule interface web**, accessible depuis le laptop ET l'iPad

## Stack technique

- **Backend** : Python 3.12 + FastAPI + uvicorn
- **Base de données** : SQLite (via module `sqlite3` natif — pas d'ORM)
- **Frontend** : HTML/CSS/JS vanilla (un seul fichier `index.html`)
- **Déploiement** : Docker + docker-compose sur NAS Synology (DSM)
- **Chemin média** : `/volume1/downloads` sur le NAS → monté en `/media` dans le container

## Structure du projet

```
hoard/
├── .github/
│   └── workflows/
│       ├── ci.yml               # Lint + tests sur chaque push/PR
│       └── docker-build.yml     # Build image Docker sur main / tags
├── backend/
│   ├── main.py              # Toute la logique backend (FastAPI)
│   └── requirements.txt
├── frontend/
│   └── index.html           # Toute l'UI (CSS + JS inline)
├── tests/
│   ├── conftest.py          # Fixtures pytest + isolation env (temp dirs)
│   └── test_api.py          # 31 tests, 99% coverage
├── docs/                    # Documentation (FR + EN)
├── docker-compose.yml       # Production (Synology)
├── docker-compose.dev.yml   # Dev override (hot-reload + volume local)
├── Dockerfile               # Image non-root + HEALTHCHECK
├── pyproject.toml           # Config pytest (testpaths, pythonpath) + ruff
├── requirements-dev.txt     # fastapi + uvicorn + pytest + httpx + ruff
├── ROADMAP.md               # Milestones v1.1 → v2.0
├── CLAUDE.md                # Ce fichier
└── README.md
```

## Architecture backend (main.py)

### Configuration (variables d'env)
| Variable | Défaut | Description |
|---|---|---|
| `MEDIA_ROOT` | `/media` | Racine des fichiers média dans le container |
| `PREDEFINED_FOLDERS` | `Vu,A revoir,A supprimer` | Dossiers cibles pour déplacement rapide |
| `DB_PATH` | `/data/progress.db` | Chemin SQLite |

### Endpoints API
| Méthode | Route | Description |
|---|---|---|
| GET | `/api/config` | Retourne `predefined_folders` |
| GET | `/api/files?path=` | Liste les fichiers/dossiers d'un chemin relatif |
| GET | `/api/progress?path=` | Lit la progression d'un fichier |
| POST | `/api/progress?path=` | Sauvegarde `{position, duration}` |
| DELETE | `/api/files?path=` | Supprime un fichier ou dossier |
| POST | `/api/files/move?path=` | Déplace vers `{destination}` (chemin relatif) |
| POST | `/api/files/move-to?path=` | Déplace vers `{folder}` (dossier prédéfini) |
| GET | `/api/stream?path=` | Stream vidéo avec support Range (seek) |

### Schéma SQLite
```sql
CREATE TABLE progress (
    path TEXT PRIMARY KEY,    -- chemin relatif depuis MEDIA_ROOT
    position REAL DEFAULT 0,  -- secondes
    duration REAL DEFAULT 0,  -- secondes
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
```

### Sécurité
- `safe_path()` vérifie que tout chemin résolu reste dans `MEDIA_ROOT` (path traversal)

## Architecture frontend (index.html)

### État de lecture (affichage dans la liste)
- 🎬 `percent == 0` → non vu (fond neutre)
- ▶️ `0 < percent < 90` → en cours (fond jaune sombre, barre de progression + %)
- ✅ `percent >= 90` → vu (fond vert sombre)

### Player
- Tag HTML5 `<video>` natif
- Streaming via `/api/stream` avec support `Range` (seeking natif)
- Sauvegarde de position toutes les **5 secondes** via `setInterval`
- Reprise automatique à la position sauvegardée au chargement

### Gestes tactiles (sur l'élément `<video>`)
- **Swipe horizontal** → seek (60s / 100px)
- **Swipe vertical** → volume
- **Double-tap** → ±10s (gauche/droite)
- **Tap simple** → pause/play

### Raccourcis clavier
- `Espace` → pause/play
- `←` / `→` → seek ±10s
- `↑` / `↓` → volume ±10%

### Responsive
- Desktop (>700px) : layout splitté — liste à gauche (380px), player à droite
- Mobile (≤700px) : liste plein écran, player en overlay fullscreen

## Ce qui est fait (MVP)

- [x] Browser de filesystem avec navigation et breadcrumb
- [x] Indicateurs visuels vu / en cours / non vu dans la liste
- [x] Player intégré avec seekbar, contrôles, skip
- [x] Sauvegarde de position automatique toutes les 5s
- [x] Reprise automatique à la position sauvegardée
- [x] Déplacement vers dossiers prédéfinis (modal)
- [x] Suppression avec confirmation
- [x] Gestes tactiles sur le player
- [x] Raccourcis clavier
- [x] Dockerfile + docker-compose pour Synology

## Ce qui reste à faire (backlog)

### Priorité haute
- [ ] **Tags arbitraires** sur les fichiers (ex: "à finir", "excellent") — stockés en SQLite, affichés dans la liste
- [ ] **Déplacement libre** dans l'arborescence (picker de destination dans le filesystem)
- [ ] **Tri** dans la liste : par nom, date, taille, état de lecture

### Priorité moyenne
- [ ] **Renommage** de fichiers depuis l'UI
- [ ] **Rafraîchissement auto** de la liste (détecter les nouveaux fichiers)
- [ ] **Fullscreen** propre sur le player (bouton + raccourci F)
- [ ] **Vitesse de lecture** (0.5x, 1x, 1.5x, 2x)
- [ ] **Sous-titres** (fichiers .srt/.ass dans le même dossier)

### Priorité basse
- [ ] **Recherche** dans les noms de fichiers
- [ ] **Authentification** basique (si exposé hors LAN)
- [ ] **Thème clair** (toggle)
- [ ] **PWA** (installable sur iPad/laptop)

## Conventions de code

- **Backend** : fonctions courtes, typage Pydantic pour les bodies POST, pas d'ORM
- **Frontend** : JS vanilla, pas de framework, tout dans `index.html`
- **Pas de breaking change** sur l'API sans mettre à jour ce fichier
- Variables CSS dans `:root` pour tous les tokens de couleur
- Les chemins sont toujours **relatifs à MEDIA_ROOT** dans la DB et l'API

## Développement local

```bash
cd nas-vid-bro

# Créer et activer le venv
python -m venv .venv
.venv\Scripts\activate  # Linux/Mac: source .venv/bin/activate

# Install deps (dev)
pip install -r requirements-dev.txt

# Lance avec un dossier vidéo local
export MEDIA_ROOT=/chemin/vers/videos
export PREDEFINED_FOLDERS="Vu,A revoir"
export DB_PATH=/tmp/progress.db

uvicorn backend.main:app --reload --port 8000
# → http://localhost:8000

# Tests
python -m pytest tests/ -v
```

## Déploiement Synology

```bash
# Sur le NAS (SSH ou via Portainer)
cd /volume1/docker/hoard

# Adapter le volume dans docker-compose.yml :
# - /volume1/downloads:/media:rw

docker compose up -d --build
# → http://IP-NAS:8000
```

## Infrastructure NAS (contexte)

- **NAS** : Synology avec DSM
- **Docker** géré via **Portainer**
- **Domaine** : `dpierron.me` (en cours de transfert vers Cloudflare)
- **Reverse proxy** : Synology built-in (subdomain → container)
- **Autres stacks Docker** : SteamPulse, DCSServerBot, et d'autres services self-hosted

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidp57)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidp57)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
