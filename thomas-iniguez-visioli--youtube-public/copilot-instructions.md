## youtube-public

> - **Système de favoris** : Possibilité de marquer des vidéos comme favorites avec une vue dédiée `/favorites`, un bouton dédié sur le lecteur et des icônes d'action sur la bibliothèque.

# Gemini CLI - Journal des modifications

## [1.2.5] - 2026-03-13
### Ajouté
- **Système de favoris** : Possibilité de marquer des vidéos comme favorites avec une vue dédiée `/favorites`, un bouton dédié sur le lecteur et des icônes d'action sur la bibliothèque.
- **Compteur de favoris** : Affichage dynamique du nombre de favoris dans la barre de navigation.
- **Gestion AJAX** : Ajout/retrait des favoris sans rechargement de page.

## [1.1.61] - 2026-03-01
### Ajouté
- **File d'attente de lecture (Queue)** : Système permettant d'ajouter des vidéos à une file d'attente prioritaire avec gestion AJAX, retrait automatique après lecture et vue dédiée pour la gestion.
- **Indicateur d'historique** : Affichage dynamique de l'utilisation de l'historique (nombre actuel / limite maximale) dans la barre de navigation.
- **Compteur de file** : Badge dynamique affichant le nombre de vidéos en attente sur toutes les pages.
- **Affichage de version** : La version de l'application est désormais visible dans la barre de navigation.
- **Correction des binaires** : Ajout des fichiers de résolution de binaires (`binaryResolver.js`) manquants dans les versions précédentes.

## [1.1.58] - 2026-02-20
### Ajouté
- **Playlists par chaîne** : Création automatique de playlists basées sur l'uploader pour une meilleure organisation.
- **Lecture séquentielle intelligente** : Le système détecte désormais si une vidéo est lue via une playlist (personnalisée ou par chaîne) et enchaîne automatiquement sur la vidéo suivante de cette liste.

## [1.1.57] - 2026-02-19
### Corrigé
- **Compatibilité Extension** : Correction du port API dans `background.js` (passage de 3000 à 8001) pour correspondre à l'application.
- **Logs Serveur** : Correction du message de log indiquant le mauvais port au démarrage (8000 -> 8001).

## [1.1.56] - 2026-02-19
### Ajouté
- **Moteur de recherche Fuzzy** : Intégration de Fuse.js dans la bibliothèque pour une recherche ultra-rapide et tolérante aux fautes de frappe dans les titres et noms de chaînes.

## [1.1.55] - 2026-02-19
### Corrigé
- **Lancement de yt-dlp** : Suppression de l'option `shell: true` qui provoquait des erreurs "Fichier introuvable" sur certains systèmes Windows.
- **Journalisation des erreurs** : Amélioration de la capture des erreurs lors du lancement des processus externes (spawn) pour faciliter le diagnostic.

## [1.1.54] - 2026-02-19
### Ajouté
- **Système de Playlists** : Création, gestion et lecture de listes de lecture personnalisées.
- **Organisation** : Possibilité d'ajouter/retirer des vidéos des playlists directement depuis l'interface de lecture.

## [1.1.53] - 2026-02-19
### Ajouté
- **Raccourcis clavier YouTube** : Support des touches `k`, `j`, `l`, `f`, `m`, `Maj+N` et `0-9` pour une navigation identique à YouTube.
- **Persistance de lecture** : Sauvegarde automatique de la position de lecture pour reprendre chaque vidéo là où elle s'était arrêtée.
- **Restauration de session** : L'application mémorise la dernière page consultée et y revient automatiquement au démarrage.
- **Mise à jour forcée des assets** : Système de détection de version forçant le re-téléchargement des binaires (yt-dlp, bun) lors d'une mise à jour de l'application pour garantir la stabilité.

## [1.1.52] - 2026-02-19
### Changé
- **Optimisation de la mémoire** : Implémentation d'un système de nettoyage automatique du cache de session Electron toutes les 10 minutes et après chaque téléchargement pour réduire l'empreinte mémoire.

## [1.1.51] - 2026-02-19
### Corrigé
- **Affichage complet de la bibliothèque** : Suppression du filtre excluant l'historique sur la page d'accueil pour que tous les fichiers téléchargés soient visibles.
- **Gestion robuste des métadonnées** : Les vidéos dont le fichier `.info.json` est manquant ne sont plus supprimées de la base de données et du disque, assurant la conservation des fichiers média.
- **Correction de bug DB** : Implémentation de la méthode `removeFile` manquante dans la classe `FileDatabase`.

## [1.1.50] - 2026-02-19
### Ajouté
- **Surveillance du backlog** : Le fichier `backlog.txt` sur le bureau est désormais surveillé. Toute modification externe (ajout ou suppression d'URLs manuelle) est automatiquement détectée et rechargée par l'application sans créer de boucle infinie.

## [1.1.49] - 2026-02-19
### Corrigé
- **ReferenceError: backlogFile is not defined** : Correction de l'erreur de rendu dans les templates EJS en rendant la variable optionnelle et en assurant sa définition dans toutes les configurations possibles de l'application (root et src).

## [1.1.48] - 2026-02-19
### Ajouté
- **Mise à jour automatique et redémarrage** : L'application redémarre désormais automatiquement 3 secondes après avoir fini de télécharger une mise à jour pour l'installer.

## [1.1.47] - 2026-02-19
### Ajouté
- **Affichage du chemin du backlog** : Le lien complet vers le fichier `backlog.txt` est désormais affiché sous la barre de téléchargement dans l'interface utilisateur.

## [1.1.46] - 2026-02-19
### Ajouté
- **Persistence du backlog** : Le backlog des téléchargements en attente est désormais stocké dans un fichier `backlog.txt` sur le bureau de l'utilisateur. Il est chargé au démarrage de l'application et mis à jour lors de chaque ajout ou complétion de téléchargement.

## [1.1.45] - 2026-02-19
### Ajouté
- **Barre de recherche sur la vue lecture** : Intégration de la barre de téléchargement de vidéos directement dans la vue de lecture pour faciliter le téléchargement sans repasser par l'accueil.

## [1.1.44] - 2026-02-18
### Changé
- **Limite d'historique** : La longueur de l'historique est désormais dynamique et limitée à 80% du nombre total de vidéos en base pour une meilleure gestion de l'espace.

## [1.1.41] - 2026-02-17
### Ajouté
- **Téléchargement de chaîne** : Ajout d'un bouton "Télécharger la chaîne" dans la vue par chaîne pour faciliter l'archivage complet.

## [1.1.40] - 2026-02-17
### Changé
- **Suggestions de vidéos** : Affichage de seulement 5 vidéos aléatoires dans la barre latérale pour améliorer la navigation.

## [1.1.37-6] - 2026-02-15
### Ajouté
- **Système d'historique** : Suivi des vidéos visionnées avec une nouvelle vue dédiée `/history`.
- **Détection de navigateur pour les cookies** : Utilisation automatique des cookies de Firefox (prioritaire) ou Chrome pour les téléchargements.
- **Publication automatique** : Job GitHub Action pour publier la release une fois les builds terminés.
- **Migration vers Bun** : Remplacement de npm par Bun pour la gestion des paquets, les scripts et les tests. Utilisation de Bun dans le workflow CI/CD.

## [1.1.37-5] - 2026-02-15
### Ajouté
- **Tests automatisés** : Suite de tests unitaires pour la file d'attente, la base de données et le module de téléchargement.
- **Modularité** : Extraction de la logique de téléchargement (`downloader.js`) et de mise à jour (`updater.js`) pour une meilleure testabilité.

### Corrigé
- **Séquence de démarrage** : Correction d'une erreur de référence au boot et ajout d'une attente pour les fichiers essentiels.
- **Fiabilité yt-dlp** : Amélioration de la gestion des arguments et des chemins Windows pour éviter les échecs d'exécution.

## [1.1.37-4] - 2026-02-14
### Ajouté
- **Système de visualisation par chaîne** : Nouvelle route `/channel` permettant de filtrer les vidéos par uploader.
- **Navigation améliorée** : Les noms des créateurs sont désormais des liens cliquables sur la page d'accueil et la page de lecture.
- **Interface contextuelle** : Affichage du nom de la chaîne filtrée dans l'en-tête de la bibliothèque.

### Corrigé
- **Stabilisation du téléchargement** : Implémentation d'une file d'attente asynchrone pour éviter la saturation des ressources.
- **Chemins système** : Correction des chemins vers `ytdlp.exe` et `ffmpeg` pour pointer vers le dossier `AppData` de l'utilisateur.
- **Sécurisation de la route `/watch`** : Ajout de vérifications pour éviter les plantages si les fichiers de métadonnées sont absents.

### Changé
- **Refonte UI/UX** : Intégration de Plyr.io pour le lecteur vidéo et passage à un thème sombre moderne inspiré de YouTube.
- **Consolidation des vues** : Déplacement des fichiers EJS de la racine vers `src/views/` pour une structure plus propre.

## Instructions de développement
- **prepush** : Avant chaque push, s'assurer que la version dans `package.json` est à jour et que les tests passent.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thomas-iniguez-visioli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
