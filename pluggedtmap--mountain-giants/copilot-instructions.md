## mountain-giants

> Ce fichier sert de référence pour l'IA (Gemini) afin de comprendre rapidement l'architecture, les contraintes et les flux de travail du projet.

# Contexte du Projet : Mountain Giants

Ce fichier sert de référence pour l'IA (Gemini) afin de comprendre rapidement l'architecture, les contraintes et les flux de travail du projet.

## 1. Architecture Générale
- **Backend** : Node.js avec Express dans `/backend/server.js`.
- **Frontend** : HTML/JS Vanilla dans le dossier `/frontend/`.
- **Base de Données** : Fichiers JSON dans `/backend/data/` (`data.json`, `orders.json`).
- **Déploiement** : Dockerfile et Compose dans `/deployment/`.
- **Scripts** : Utilitaires dans `/scripts/`.
- **Port** : `3018` (par défaut).

## 2. Dépendances Critiques
- **FFmpeg** : Requis sur le serveur hôte (ou conteneur) pour la conversion vidéo. Le serveur détecte les fichiers `.mov`/`.qt` et les convertit en `.mp4` via `exec('ffmpeg ...')`.
- **GitHub API** : Utilisé comme stockage de fichiers (hébergement d'images/vidéos). Le serveur agit comme proxy/client pour uploader vers le repo `pluggedtmap/Mountain Giants-` (ou autre configuré).

## 3. Déploiement & DevOps
- **Méthode actuelle** : Script `deploy.sh`.
    1. Commit & Push sur GitHub.
    2. Le VPS fait un `git pull`, et redémarre via Docker.
- **Docker** : Encapsule `node` + `ffmpeg` et sécurise l'exécution.

## 4. Sécurité
- **Authentification Admin** : Mot de passe hashé (bcrypt) stocké dans `data.json`.
- **Uploads** : Multer utilisé localement temporairement avant envoi vers GitHub. 
- **Sécurité Serveur** : Headers manuels (X-Content-Type-Options, X-XSS-Protection), filtrage d'extensions.

## 5. Points d'Attention
- **Extensions de fichiers** : Le back-end refuse les fichiers non-médias.
- **Limites** : Upload limité à 95MB (Github limit 100MB).
- **Logs** : Le serveur loggue chaque requête dans la console.

## 6. Commandes Utiles
- Local : `node server.js` ou `./scripts/start_local.sh`
- Docker : `./scripts/start_docker.sh`
- Deploy : `./scripts/deploy.sh "Message commit"`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pluggedtmap)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pluggedtmap)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
