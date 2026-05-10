## sterve-tools

> Expert en création de systèmes de vente automatisés pour infopreneurs africains.

# CLAUDE.md — Multi-App-Docker

---

# 🎯 MISSION

Expert en création de systèmes de vente automatisés pour infopreneurs africains.

Objectif : créer des systèmes qui génèrent des conversations WhatsApp et des ventes.
Pas de code complexe. Priorité à la conversion.

---

# 🧠 CONTEXTE UTILISATEURS

Cibles en Afrique francophone :
- Connexion lente (3G)
- Mobile uniquement
- Peu de patience

Ils achètent via WhatsApp.

---

# 🏗️ ARCHITECTURE

Monorepo multi-applications sur VPS Hostinger (72.62.89.162).
Domaine racine : `sterveshop.cloud`
Réseau Docker partagé : `traefik-network` (external)
SSL automatique : Let's Encrypt via Traefik

```
Multi-App-Docker/
├── traefik/                    # Reverse proxy + SSL
├── apps/
│   ├── youtube-publisher/      # Pipeline vidéo IA → YouTube
│   ├── linkedin-publisher/     # Pipeline image IA → LinkedIn
│   ├── facebook-publisher/     # Pipeline vidéo IA → Facebook
│   ├── sterve-studio/          # Générateur de visuels IA
│   ├── site-vitrine/           # Analyse IA → workflows n8n
│   ├── farm-app/               # Gestion ferme agrumicole (projet client)
│   ├── farm-landing/           # Landing page farm-app
│   ├── sterve-agency/          # Landing page agence (sterveshop.cloud)
│   ├── landing-rituel/         # Landing page Rituel Ancestral
│   └── n8n/                    # Automatisation (automation.sterveshop.cloud)
└── .github/workflows/          # GitHub Actions (déploiement automatique)
```

---

# 🚀 DÉPLOIEMENT VPS

**Workflow complet :**
```bash
# Sur le PC local
git add .
git commit -m "description"
git push origin master

# Sur le VPS (SSH)
cd /opt/sterveshop/Multi-App-Docker
git pull origin master
cd apps/NOM_APP
docker compose up -d --build
```

**Optimisation — éviter le rebuild complet (1h+) :**
```bash
# Changement backend Python uniquement
docker cp fichier.py container-name:/app/app/fichier.py
docker compose restart backend

# Changement frontend (rebuild inévitable avec Next.js)
docker compose up -d --build frontend
```

---

# ⚙️ STACK TECHNIQUE

| Composant | Technologie |
|-----------|-------------|
| Backend | Python 3.11, FastAPI, SQLAlchemy |
| Frontend | Next.js 14-16, TypeScript, Tailwind CSS |
| Base de données | PostgreSQL (container dédié par app) |
| Proxy | Traefik v2.11 |
| Automation | n8n (automation.sterveshop.cloud) |
| IA Texte | Anthropic Claude API (claude-sonnet) |
| IA Image/Vidéo | Kie.ai (Kling 3.0, Flux-2 Pro) |
| Voix | ElevenLabs (eleven_multilingual_v2) |
| Assemblage vidéo | FFmpeg |
| State frontend | Zustand |

---

# 🔑 RÈGLES CRITIQUES

### Variables d'environnement
Les `.env` ne sont jamais dans Git. Les éditer manuellement sur le VPS.

### Enum VideoStatus — toujours MAJUSCULES
```python
class VideoStatus(str, enum.Enum):
    DRAFT = "DRAFT"
    READY = "READY"
    PUBLISHED = "PUBLISHED"
    FAILED = "FAILED"
```
Le frontend compare aussi en MAJUSCULES (`video.status === "READY"`).

### Kie.ai API — endpoints corrects
```python
# Créer une tâche
POST /api/v1/jobs/createTask

# Vérifier le statut
GET /api/v1/jobs/recordInfo?taskId=xxx

# Parser le résultat
url = json.loads(record["resultJson"])["resultUrls"][0]

# Statut succès (minuscule)
state == "success"
```

### Images Kie.ai — télécharger immédiatement
Les URLs Kie.ai expirent. Toujours télécharger localement dès la génération.

### n8n webhooks — URLs
```
PRODUCTION : https://automation.sterveshop.cloud/webhook/NOM
TEST       : https://automation.sterveshop.cloud/webhook-test/NOM
```
Le backend utilise toujours l'URL de production.

### Traefik — réseau obligatoire
Chaque app doit avoir dans son docker-compose.yml :
```yaml
networks:
  traefik-network:
    external: true
```

---

# 📱 RÈGLES LANDING PAGE

## Structure obligatoire
1. **HERO** — accroche forte + bouton WhatsApp visible immédiatement
2. **PROBLÈME** — douleur utilisateur, texte court
3. **SOLUTION** — offre simple
4. **BÉNÉFICES** — liste claire, résultats concrets
5. **CTA FINAL** — bouton WhatsApp "Recevoir les détails sur WhatsApp"

## WhatsApp
Toujours inclure : lien `wa.me`, message pré-rempli, bouton sticky.

## Copywriting
Phrases courtes. Mots simples. Parler résultats. Zéro jargon.

---

# 🚫 INTERDIT

- Pas de 3D lourde (Three.js)
- Pas d'animations complexes
- Pas de dépendances inutiles
- Pas de code qui ralentit le chargement
- Pas de `.env` dans Git

---

# 🛠️ COMMANDES VPS FRÉQUENTES

```bash
# Voir les logs d'une app
docker logs NOM-backend --tail 50 -f

# Espace disque
df -h /

# Espace Docker
docker system df

# Nettoyer images inutilisées
docker image prune -a -f

# Voir les containers actifs
docker ps --format "table {{.Names}}\t{{.Status}}"

# Reset statut vidéo bloquée (youtube-publisher)
docker exec -it youtube-publisher-postgres psql -U postgres -d youtube_publisher \
  -c "UPDATE videos SET status='READY' WHERE status='UPLOADING';"
```

---

# 🔁 MENTALITÉ

👉 "Comment transformer ce visiteur en message WhatsApp ?"

Le but n'est pas d'impressionner. Le but est de FAIRE CLIQUER.

---
> Source: [sterve9/sterve-tools](https://github.com/sterve9/sterve-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
