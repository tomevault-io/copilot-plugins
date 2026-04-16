## together-app

> Ceci est une application mobile et web construite avec **React Native** et **Expo**. L'application, nommée "Together", sert de plateforme pour connecter les bénévoles avec les associations pour diverses missions. Elle prend en charge différents rôles d'utilisateur, y compris `volunteer` (bénévole), `association` et `admin`, chacun avec une disposition et des routes d'application dédiées. Le projet utilise **TypeScript** et le routage basé sur les fichiers fourni par `expo-router`.

# Projet : Application Together

## Aperçu du projet

Ceci est une application mobile et web construite avec **React Native** et **Expo**. L'application, nommée "Together", sert de plateforme pour connecter les bénévoles avec les associations pour diverses missions. Elle prend en charge différents rôles d'utilisateur, y compris `volunteer` (bénévole), `association` et `admin`, chacun avec une disposition et des routes d'application dédiées. Le projet utilise **TypeScript** et le routage basé sur les fichiers fourni par `expo-router`.

L'application est conteneurisée à l'aide de **Docker** et **Docker Compose**, avec un `Dockerfile` multi-étapes qui construit la version web de l'application et la sert à l'aide d'un serveur web `nginx`.

## Construction et exécution

### Prérequis

*   Node.js et npm
*   Docker (facultatif, pour le déploiement conteneurisé)

### Exécution de l'application

1.  **Installer les dépendances :**
    ```bash
    npm install
    ```

2.  **Démarrer le serveur de développement :**
    ```bash
    npx expo start
    ```
    Cela ouvrira les outils de développement Expo dans votre navigateur. Vous pourrez alors choisir d'exécuter l'application sur un émulateur Android, un simulateur iOS ou en tant qu'application web.

### Scripts de construction et d'exécution

Le fichier `package.json` contient plusieurs scripts pour gérer l'application :

*   `npm run lint` : Lint le code source à l'aide d'ESLint.
*   `npm run build:web` : Crée une version web de production dans le répertoire `dist/`.
*   `npm run serve` : Sert la version web localement.
*   `npm run docker:compose:build` : Construit et démarre l'application dans un conteneur Docker.

## Conventions de développement

### Messages de commit

Le projet suit la spécification **Conventional Commits**. Chaque message de commit doit respecter le format suivant :

`<type>: <ID-JIRA> <description>`

*   **type** : `feat` (fonctionnalité), `fix` (correctif), `build` (construction), `chore` (tâche de routine), `ci` (intégration continue), `docs` (documentation), `style` (style), `refactor` (refactorisation), `perf` (performance), `test` (test), `revert` (retour arrière).
*   **ID-JIRA** : L'ID du ticket JIRA correspondant (le cas échéant).
*   **description** : Une description concise et en minuscules des modifications.

### Style de code

Le projet utilise ESLint pour le linting du code. Exécutez `npm run lint` pour vérifier les problèmes de style.

## Structure du projet

*   `app/` : Contient les routes de l'application, organisées par rôles d'utilisateur (`admin`, `association`, `guest`, `volunteer`).
*   `assets/` : Actifs statiques tels que les images et les polices.
*   `components/` : Composants React réutilisables.
*   `constants/` : Constantes globales, telles que les couleurs et les configurations de thème.
*   `context/` : Fournisseurs de contexte React, y compris `AuthContext` pour gérer l'authentification des utilisateurs.
*   `hooks/` : Hooks React personnalisés.
*   `models/` : Interfaces TypeScript définissant les modèles de données de l'application.
*   `services/` : Modules responsables de l'interaction avec l'API backend et d'autres services externes.
*   `styles/` : Feuilles de style pour les composants et les pages.

## API Backend

L'application communique avec une API backend. L'URL de base de l'API est configurée dans `services/api.ts` :

*   **Développement** : `http://<VOTRE-IP-LOCALE>:3000`
*   **Production** : `https://together-api.out-online.net`

Le service API utilise `axios` et comprend un intercepteur pour gérer automatiquement le rafraîchissement des jetons JWT.

## Authentification

L'authentification est gérée via l'`AuthContext` (`context/AuthContext.tsx`). Ce contexte fournit des informations sur l'utilisateur, l'état de l'authentification et les méthodes de connexion, de déconnexion et de rafraîchissement du profil de l'utilisateur. Un mode débogage est disponible dans l'`AuthContext` qui utilise un utilisateur simulé à des fins de test.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lyric-s) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
