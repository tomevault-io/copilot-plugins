## saasberinia

> Bienvenue dans la documentation du système BerinIA, un écosystème d'agents IA autonomes et collaboratifs pour l'automatisation de la prospection commerciale.

# Règles de travail pour les agents et le backend – Projet BerinIA

## 📚 CONTEXTE DOCUMENTAIRE

Bienvenue dans la documentation du système BerinIA, un écosystème d'agents IA autonomes et collaboratifs pour l'automatisation de la prospection commerciale.

### 🗂️ Structure de la documentation

La documentation est organisée en sections thématiques pour faciliter la navigation :

#### 📊 Vue d'ensemble du système
- Synthèse complète du système : `./résumé/synthèse_système_BerinIA.md`

#### 🏗️ Architecture
- Architecture détaillée : `./architecture/ARCHITECTURE.md`
- Vue simplifiée : `./architecture/overview.md`
- Système d'agents : `./architecture/agents-system.md`
- Communication entre composants : `./architecture/communication.md`

#### 🔌 Intégrations
- Base de données : `./integrations/database.md`
- Twilio (SMS) : `./integrations/sms-twilio.md`
- Instantly.ai : `./integrations/instantly.md`

#### ⚙️ Services et déploiement
- Services Systemd : `./services/systemd_services.md`
- API : `./api.md`
- Variables d’environnement : `./services/env_variables_api.md`

#### 📘 Guides
- Utilisation API par l’IA : `./api_usage_guide_for_ai.md`

### 🚀 Installation et démarrage

1. Vue d’ensemble : `./architecture/overview.md`
2. Déploiement des services : `./services/systemd_services.md`

### 🔄 Utilisation quotidienne

1. Démarrage des services (voir `systemd_services.md`)
2. Interface langage naturel : `python interact.py`
3. Logs : via API ou `journalctl`

### 📋 Administration système

1. Gestion des services via `/api/services/`
2. Configuration des intégrations
3. Suivi des logs

---

⚠️ Règles à respecter impérativement :

1. Ne jamais appeler fs.readFileSync ou équivalent sans vérifier que le chemin est un fichier.
   ➤ Utiliser fs.statSync(path).isFile() avant toute lecture.

2. Ne jamais passer un dossier comme argument à readFile, readFileSync ou open.
   ➤ Toujours valider le chemin : log ou vérifie avec fs.existsSync et fs.lstatSync.

3. Si un chemin est dynamique ou inconnu, faire un log de debug avant d'y accéder.

4. En cas d’erreur, elle doit être catchée proprement avec try/catch, sans crasher le process.

⛔ Une erreur de type “EISDIR” est interdite et signifie que les vérifications n’ont pas été faites correctement.

---
## ENVIRONNEMENT DE TRAVAIL

- Active toujours l’environnement Python (`.venv`) avant toute modification.
- Base de données : `/root/berinia/backend`.

---

## CRÉATION DE FICHIERS ET DE FONCTIONNALITÉS

- Vérifie toujours l’existence d’un fichier avant de le créer.
- N’écrase jamais un fichier stable sans raison.
- Les tests doivent être faits dans `/root/berinia/infra-ia/tests`.

---

## VALIDATION DES FONCTIONNALITÉS

- Toute fonctionnalité modifiée ou ajoutée doit être testée immédiatement.
- Ne jamais stopper une tâche tant que des erreurs sont présentes.

---

## DOCUMENTATION

- Chaque ajout ou modification fonctionnelle doit être **documenté** :
  - Si le sujet existe : **modifie le fichier existant**.
  - Si le sujet est nouveau : **crée un nouveau fichier** dans `infra-ia/documentation`.
  - Si un fichier connexe existe : **complète la documentation existante** plutôt que créer du doublon.
- Toujours analyser ce qui existe **avant** d’écrire ou créer un fichier.
- La documentation est déjà dense, l’objectif est de la maintenir claire et synthétique.

---

## OUTILS & GESTION DE DÉPENDANCES

- Utilise **exclusivement `pnpm`**
- `npm` est **interdit**, même ponctuellement.

🚫 RÈGLE ABSOLUE POUR TOUS LES AGENTS

❗ INTERDICTION TOTALE D’UTILISER DES MOCKDATA, DONNÉES FICTIVES, OU TESTS SIMULÉS NON CONNECTÉS AU SYSTÈME RÉEL.

Tous les tests, réponses, traitements, statistiques ou affichages doivent obligatoirement s’appuyer sur des données réelles extraites de la base de données, de l’API, ou d’une source de vérité authentifiée.

✅ OBJECTIF : Ne jamais "faire croire" que quelque chose fonctionne. Tout comportement simulé ou factice est considéré comme un échec.

🔒 Aucune exception n’est permise. Toute tentative de substitution avec des valeurs en dur est considérée comme une violation critique de la logique de véracité du système.

## Vue d'ensemble des services

| Service | Description | Dépendances | Port |
|---------|-------------|-------------|------|
| berinia-api.service | API backend principale | PostgreSQL | 8000 |
| berinia-next.service | Frontend Next.js | berinia-api | 3000 |
| berinia-webhook.service | Serveur webhook pour réception d'événements externes | berinia-api | 8001 |
| berinia-qdrant.service | Base de données vectorielle Qdrant | Docker | 6333 |
| berinia-agents.service | Environnement d'exécution des agents IA | PostgreSQL, berinia-qdrant | - |
| berinia-scheduler.service | Planificateur de tâches et exécutions planifiées | PostgreSQL | - |
| berinia-telegram-bot.service | Bot Telegram v1 (seule version active) | berinia-api | - |

1. Services de base de données:
   - PostgreSQL (service système)
   - `berinia-qdrant.service`

2. Services d'API:
   - `berinia-api.service`
   - `berinia-webhook.service`

3. Services d'agents:
   - `berinia-agents.service`
   - `berinia-scheduler.service`

4. Services frontend et intégrations:
   - `berinia-next.service`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MartialAndCo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
