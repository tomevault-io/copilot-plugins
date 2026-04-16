## social-mds

> **MyDigitalScoop** est un réseau social nouvelle génération. Le projet implémente **MyDigitalScoop**, un réseau social interne exclusif pour MyDigitalSchool, permettant aux étudiants, intervenants et gestionnaires de partager et interagir dans un environnement sécurisé et modéré.

# MyDigitalScoop - Project Context

## 🎯 Objectif du Projet

**MyDigitalScoop** est un réseau social nouvelle génération. Le projet implémente **MyDigitalScoop**, un réseau social interne exclusif pour MyDigitalSchool, permettant aux étudiants, intervenants et gestionnaires de partager et interagir dans un environnement sécurisé et modéré.

## 🛠️ Stack Technique

- **Frontend :** React 19, Next.js 16 (App Router), TypeScript
- **Backend :** Java Spring Boot avec base de données PostgreSQL
- **Style :** Tailwind CSS 4 + Framer Motion pour les animations
- **Icônes :** Lucide React
- **Tooling :** Biome (Linter & Formatter)
- **Package Manager :** pnpm

## 🔐 Contraintes d'Accès & Sécurité

### Accès Restreint par Mailing List

- **Pré-requis :** L'accès à la plateforme est strictement limité aux adresses email présentes dans la **mailing list officielle de MyDigitalSchool**
- **Authentification :** Système **MagicLink** (token unique envoyé par email, pas de mot de passe traditionnel)
- **Validation :** Le backend vérifie l'appartenance à la mailing list avant d'envoyer le MagicLink

### Contraintes de Connexion & Publication

#### 🏫 Membres Simples (Étudiants)

- **Connexion :** Peuvent se connecter partout via MagicLink
- **Publication :** **Uniquement depuis le WiFi de MDS** (vérification IP côté backend)
- **Restriction :** Tentatives de post hors WiFi école = refus automatique

#### ⭐ Membres VIP (Intervenants & Gestionnaires BDE)

- **Connexion :** Partout via MagicLink
- **Publication :** **Aucune restriction géographique** (WiFi école + extérieur)
- **Privilèges :** Accès complet à toutes les fonctionnalités

#### 👨‍💼 Admin (Alain - Modérateur Principal)

- **Accès :** Complet partout + **Dashboard d'administration**
- **Modération :** Outils de modération avancés (validation/suppression posts, gestion utilisateurs)
- **Gestion :** Administration de la mailing list et des rôles utilisateurs

### 🤖 Système de Modération Hybride

#### Modération Automatisée par IA

- **Filtrage intelligent :** Détection automatique de contenu inapproprié
- **Bannissement de mots-clés :** Dictionnaire de termes interdits configurable
- **Flags de contenu :** Signalement automatique pour review humaine
- **Actions :** Blocage instantané ou mise en quarantaine selon la gravité

#### Modération Humaine (Alain)

- **Interface dédiée :** Dashboard pour review des contenus flaggés par l'IA
- **Actions disponibles :** Approuver, Supprimer, Bannir utilisateur
- **Gestion proactive :** Configuration des filtres IA et mise à jour du dictionnaire de mots bannis

## 📜 Règles de Code

- Utilise toujours des **Composants Fonctionnels** (Arrow Functions)
- Favorise la clarté sur la concision : nomme les variables de façon explicite
- Toutes les réponses concernant le code doivent être en **français**, mais les commentaires dans le code doivent être en **anglais**
- Priorise l'accessibilité (ARIA labels)
- Utilise `clsx` et `tailwind-merge` pour les styles conditionnels

## 📂 Structure du Projet

- `/src/components` : Composants réutilisables (Header, Footer, etc.)
- `/src/app` : Routes et pages (App Router Next.js 16)
- `/src/hooks` : Logique personnalisée React
- `/back` : Backend Java Spring Boot
- `/ai_docs` : Documentation d'architecture et spécifications
- `/PRPs` : Product Requirements & Personas
- `/concept_library` : Bibliothèque de concepts et templates

## 🧪 Tests

- Nous utilisons **Jest** et **React Testing Library**
- Chaque nouveau composant doit avoir un fichier de test `.test.tsx` associé

## 🔌 Intégrations MCP

MyDigitalScoop agit comme un **MCP Host** et peut se connecter à plusieurs serveurs MCP :

- Utilise le serveur `filesystem` pour lire les logs dans `/logs`
- Utilise le serveur `github` pour vérifier l'état des Pull Requests avant de proposer une modification
- Utilise le serveur `Context7` pour obtenir la documentation à jour des bibliothèques et frameworks utilisés
- Utilise le serveur `neon` pour gérer les bases de données PostgreSQL serverless

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/remimoul) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
