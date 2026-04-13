## plex-movie-picker

> "Plex Movie Picker" est une application compagnon pour Plex conçue pour résoudre le "Paradoxe du choix".

# PRODUCT CONTEXT & VISION : Plex Movie Picker

## 1. La Mission (Value Proposition)
"Plex Movie Picker" est une application compagnon pour Plex conçue pour résoudre le "Paradoxe du choix".
Les utilisateurs passent plus de temps à chercher un film qu'à le regarder. Notre but est de réduire ce temps de friction grâce à un système de curation et de vote collaboratif.

**Rôle de l'app :** Le "Sommelier" intelligent de la vidéothèque personnelle.

## 2. Architecture Technique
Le projet suit une **Clean Architecture (Hexagonale)** stricte avec NestJS.
- **Libs/Auth :** Gestion des utilisateurs (Firebase) et des groupes.
- **Libs/Mediacenter :** Abstraction du fournisseur de média (actuellement Plex, futur Jellyfin/Emby).
- **Libs/Movies :** Cœur de métier (Domaine Movie, Library).
- **Libs/Session (À FAIRE) :** Le moteur de jeu multijoueur.
- **DB :** Postgres via Prisma ORM.

**Règle d'or :** Le domaine ne doit jamais dépendre de l'infrastructure (Framework ou DB).

## 3. Fonctionnalité Clé (En cours de dév) : "Le Mode Session"
C'est la prochaine grosse feature à implémenter.
**Objectif :** Permettre à un groupe de choisir un film ensemble de manière ludique.

**Workflow Utilisateur (User Journey) :**
1. **L'Hôte** filtre sa bibliothèque (ex: "Films d'horreur, < 2h, après 2010") et génère un "Deck" de 60 films.
2. **La Session** est créée avec un Code Court (ex: ABCD).
3. **Les Invités** rejoignent via une WebApp mobile (scan QR Code) sans créer de compte (Auth anonyme/Guest).
4. **Le Vote** se fait via une interface de Swipe (Tinder-like) ou Tournoi (Bracket).
   - Informations visibles : Affiche, Titre, Note, Trailer.
5. **Le Match** : Dès qu'un consensus est trouvé (ou à la fin de la pile), le résultat s'affiche.
6. **L'Action** : L'hôte peut lancer le film sur la TV (Deep link Plex) depuis l'app.

## 4. Principes UX
- **Zero Friction :** Pas de login pour les invités.
- **Mobile First :** L'interface de vote est pensée pour le smartphone.
- **Performance :** Les images (posters) doivent être légères.
- **Satisfaction :** Feedback visuel immédiat lors d'un match.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samhot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Samhot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
