## site-web-rds

> Application web MVC de gestion de stages (offres, candidatures, entreprises, étudiants, pilotes).

# CLAUDE.md — Plateforme de Gestion de Stages

## Vue d'ensemble du projet
Application web MVC de gestion de stages (offres, candidatures, entreprises, étudiants, pilotes).
Stack : Apache / PHP (POO) / MySQL / HTML5 / CSS3 / JS / Moteur de template.

---

## Rôles et permissions

| Fonctionnalité | Admin | Pilote | Étudiant | Anonyme |
|---|:---:|:---:|:---:|:---:|
| **SFx1** – Authentification / déconnexion | ✅ | ✅ | ✅ | ✅ |
| **SFx2** – Rechercher / afficher une entreprise | ✅ | ✅ | ✅ | ✅ |
| **SFx3** – Créer une entreprise | ✅ | ✅ | ❌ | ❌ |
| **SFx4** – Modifier une entreprise | ✅ | ✅ | ❌ | ❌ |
| **SFx5** – Évaluer une entreprise | ✅ | ✅ | ❌ | ❌ |
| **SFx6** – Supprimer une entreprise | ✅ | ✅ | ❌ | ❌ |
| **SFx7** – Rechercher / afficher une offre | ✅ | ✅ | ✅ | ✅ |
| **SFx8** – Créer une offre | ✅ | ✅ | ❌ | ❌ |
| **SFx9** – Modifier une offre | ✅ | ✅ | ❌ | ❌ |
| **SFx10** – Supprimer une offre | ✅ | ✅ | ❌ | ❌ |
| **SFx11** – Statistiques des offres (carrousel) | ✅ | ✅ | ✅ | ✅ |
| **SFx12** – Rechercher / afficher un compte Pilote | ✅ | ❌ | ❌ | ❌ |
| **SFx13** – Créer un compte Pilote | ✅ | ❌ | ❌ | ❌ |
| **SFx14** – Modifier un compte Pilote | ✅ | ❌ | ❌ | ❌ |
| **SFx15** – Supprimer un compte Pilote | ✅ | ❌ | ❌ | ❌ |
| **SFx16** – Rechercher / afficher un compte Étudiant | ✅ | ✅ | ❌ | ❌ |
| **SFx17** – Créer un compte Étudiant | ✅ | ✅ | ❌ | ❌ |
| **SFx18** – Modifier un compte Étudiant | ✅ | ✅ | ❌ | ❌ |
| **SFx19** – Supprimer un compte Étudiant | ✅ | ✅ | ❌ | ❌ |
| **SFx20** – Postuler à une offre | ❌ | ❌ | ✅ | ❌ |
| **SFx21** – Voir ses propres candidatures | ❌ | ❌ | ✅ | ❌ |
| **SFx22** – Voir candidatures des élèves (Pilote) | ❌ | ✅ | ❌ | ❌ |
| **SFx23** – Afficher sa wish-list | ❌ | ❌ | ✅ | ❌ |
| **SFx24** – Ajouter à la wish-list | ❌ | ❌ | ✅ | ❌ |
| **SFx25** – Retirer de la wish-list | ❌ | ❌ | ✅ | ❌ |
| **SFx27** – Pagination (transversal) | ✅ | ✅ | ✅ | ✅ |
| **SFx28** – Mentions légales | ✅ | ✅ | ✅ | ✅ |

---

## Liste des tâches

### 🏗️ PHASE 1 — Architecture & Infrastructure

- [X] **TASK-01** Initialiser la structure MVC (dossiers `app/`, `public/`, `views/`, `config/`, `routes/`)
- [X] **TASK-02** Configurer Apache : vhost principal + vhost statique (CSS/JS/images)
- [X] **TASK-03** Configurer HTTPS (certificat SSL, redirection HTTP → HTTPS)
- [X] **TASK-04** Mettre en place le moteur de template (ex: Twig ou moteur maison)
- [X] **TASK-05** Créer le système de routage URL backend (router PHP natif, URLs lisibles)
- [X] **TASK-06** Créer le fichier `.htaccess` pour la réécriture d'URLs
- [X] **TASK-07** Mettre en place la connexion PDO à la base de données
- [ ] **TASK-08** Créer `sitemap.xml` et `robots.txt`
- [X] **TASK-09** Ajouter les balises SEO globales (title, meta description, meta keywords, alt images)

---

### 🗄️ PHASE 2 — Base de données

- [X] **TASK-10** Concevoir le MCD/MLD complet (toutes entités + relations)
- [X] **TASK-11** Créer la table `users` (id, nom, prénom, email, password_hash, role ENUM[admin,pilote,etudiant], created_at)
- [X] **TASK-12** Créer la table `entreprises` (id, nom, description, email, telephone, created_at)
- [X] **TASK-13** Créer la table `evaluations` (id, entreprise_id FK, user_id FK, note, commentaire, created_at)
- [X] **TASK-14** Créer la table `competences` (id, libelle) — liste fixe, pas de modification UI
- [X] **TASK-15** Créer la table `offres` (id, entreprise_id FK, titre, description, remuneration, date_offre, created_at)
- [X] **TASK-16** Créer la table `offre_competences` (offre_id FK, competence_id FK) — table pivot
- [X] **TASK-17** Créer la table `candidatures` (id, offre_id FK, etudiant_id FK, lettre_motivation, cv_path, created_at)
- [X] **TASK-18** Créer la table `wishlist` (id, offre_id FK, etudiant_id FK)
- [X] **TASK-19** Créer la table `pilote_etudiant` (pilote_id FK, etudiant_id FK) — relation pilote ↔ étudiants
- [ ] **TASK-20** Vérifier et poser toutes les clés étrangères avec `FOREIGN KEY ... ON DELETE CASCADE/RESTRICT`
- [ ] **TASK-21** Créer les seeds / fixtures de données de test

---

### 🔐 PHASE 3 — Authentification (SFx1)

- [ ] **TASK-22** Créer `AuthController` (login, logout)
- [ ] **TASK-23** Créer la vue `login.html.twig` avec formulaire (email + mot de passe)
- [ ] **TASK-24** Implémenter la vérification `password_verify()` + hachage `password_hash()`
- [ ] **TASK-25** Stocker la session dans un cookie sécurisé (httpOnly, Secure, SameSite=Strict)
- [ ] **TASK-26** Créer le middleware/guard de vérification des permissions par rôle
- [ ] **TASK-27** Validation front (HTML5 required, pattern) + back (PHP) sur le formulaire de connexion
- [ ] **TASK-28** Protection CSRF sur tous les formulaires (token CSRF)
- [ ] **TASK-29** Redirection post-login selon le rôle

---

### 🏢 PHASE 4 — Gestion des entreprises (SFx2–SFx6)

- [ ] **TASK-30** Créer `EntrepriseController` (index, show, create, store, edit, update, delete, evaluate)
- [ ] **TASK-31** Vue liste/recherche entreprises avec pagination (SFx2 — tous rôles)
  - Critères : nom, description
  - Afficher : nb stagiaires ayant postulé, moyenne évaluations
- [ ] **TASK-32** Vue détail entreprise : offres liées + évaluations (SFx2)
- [ ] **TASK-33** Vue + formulaire création entreprise (SFx3 — Admin/Pilote)
- [ ] **TASK-34** Vue + formulaire modification entreprise (SFx4 — Admin/Pilote)
- [ ] **TASK-35** Formulaire évaluation entreprise (SFx5 — Admin/Pilote)
- [ ] **TASK-36** Action suppression entreprise avec confirmation (SFx6 — Admin/Pilote)
- [ ] **TASK-37** Validation front + back sur tous les formulaires entreprise

---

### 📋 PHASE 5 — Gestion des offres (SFx7–SFx11)

- [ ] **TASK-38** Créer `OffreController` (index, show, create, store, edit, update, delete, stats)
- [ ] **TASK-39** Vue liste/recherche offres avec pagination (SFx7 — tous rôles)
  - Critères : entreprise, titre, compétences, rémunération, date
  - Afficher : nb candidatures par offre
- [ ] **TASK-40** Vue détail offre (SFx7)
- [ ] **TASK-41** Vue + formulaire création offre avec sélection compétences (SFx8 — Admin/Pilote)
- [ ] **TASK-42** Vue + formulaire modification offre (SFx9 — Admin/Pilote)
- [ ] **TASK-43** Action suppression offre (SFx10 — Admin/Pilote)
- [ ] **TASK-44** Page statistiques avec carrousel JS (SFx11 — tous rôles)
  - Répartition par durée de stage
  - Top offres les plus wish-listées
  - Nombre total d'offres
  - Moyenne de candidatures par offre
- [ ] **TASK-45** Validation front + back sur tous les formulaires offre

---

### 👨‍🏫 PHASE 6 — Gestion des pilotes (SFx12–SFx15)

- [ ] **TASK-46** Créer `PiloteController` (index, show, create, store, edit, update, delete)
- [ ] **TASK-47** Vue liste/recherche pilotes avec pagination (SFx12 — Admin uniquement)
- [ ] **TASK-48** Vue + formulaire création compte pilote (SFx13 — Admin)
- [ ] **TASK-49** Vue + formulaire modification compte pilote (SFx14 — Admin)
- [ ] **TASK-50** Action suppression compte pilote (SFx15 — Admin)
- [ ] **TASK-51** Validation front + back formulaires pilote

---

### 🎓 PHASE 7 — Gestion des étudiants (SFx16–SFx19)

- [ ] **TASK-52** Créer `EtudiantController` (index, show, create, store, edit, update, delete)
- [ ] **TASK-53** Vue liste/recherche étudiants avec pagination (SFx16 — Admin/Pilote)
  - Critères : nom, prénom, email
  - Afficher : état de la recherche de stage
- [ ] **TASK-54** Vue détail étudiant (SFx16)
- [ ] **TASK-55** Vue + formulaire création compte étudiant (SFx17 — Admin/Pilote)
- [ ] **TASK-56** Vue + formulaire modification compte étudiant (SFx18 — Admin/Pilote)
- [ ] **TASK-57** Action suppression compte étudiant (SFx19 — Admin/Pilote)
- [ ] **TASK-58** Assigner un étudiant à un pilote (table `pilote_etudiant`)
- [ ] **TASK-59** Validation front + back formulaires étudiant

---

### 📨 PHASE 8 — Gestion des candidatures (SFx20–SFx22)

- [ ] **TASK-60** Créer `CandidatureController` (store, indexEtudiant, indexPilote)
- [ ] **TASK-61** Formulaire de candidature : upload CV (PDF) + saisie LM (SFx20 — Étudiant)
- [ ] **TASK-62** Sécuriser l'upload (type MIME, taille max, nom aléatoire, stockage hors webroot)
- [ ] **TASK-63** Vue liste candidatures de l'étudiant connecté (SFx21 — Étudiant)
  - Afficher offre, LM, lien CV
- [ ] **TASK-64** Vue liste candidatures des élèves du pilote connecté (SFx22 — Pilote)
  - Afficher étudiant, offre, LM, lien CV

---

### ❤️ PHASE 9 — Gestion des wish-lists (SFx23–SFx25)

- [ ] **TASK-65** Créer `WishlistController` (index, add, remove)
- [ ] **TASK-66** Vue wish-list de l'étudiant connecté (SFx23 — Étudiant)
- [ ] **TASK-67** Action ajout offre en wish-list (SFx24 — Étudiant), bouton sur fiche offre
- [ ] **TASK-68** Action suppression offre de la wish-list (SFx25 — Étudiant)

---

### 🎨 PHASE 10 — Frontend & UX

- [ ] **TASK-69** Créer le layout principal (header, nav, footer) en template
- [ ] **TASK-70** Intégrer un menu burger pour mobile (CSS/JS natif)
- [ ] **TASK-71** Responsive design sur toutes les pages (media queries CSS)
- [ ] **TASK-72** Composant pagination réutilisable (fragment de template)
- [ ] **TASK-73** Carrousel JS pour les statistiques (SFx11, natif sans jQuery obligatoire)
- [ ] **TASK-74** Page Mentions Légales (SFx28)
- [ ] **TASK-75** Messages flash (succès / erreur) dans les vues
- [ ] **TASK-76** (Bonus) Manifest PWA + Service Worker pour mode hors-ligne

---

### 🛡️ PHASE 11 — Sécurité

- [ ] **TASK-77** Échapper toutes les sorties HTML (htmlspecialchars / auto-escape du moteur de template)
- [ ] **TASK-78** Utiliser des requêtes préparées PDO partout (anti-injection SQL)
- [ ] **TASK-79** Valider et sanitizer toutes les entrées côté PHP
- [ ] **TASK-80** Implémenter et vérifier les tokens CSRF sur chaque formulaire POST
- [ ] **TASK-81** Aucun mot de passe ou donnée sensible en clair (BDD + cookies)
- [ ] **TASK-82** Headers de sécurité HTTP (Content-Security-Policy, X-Frame-Options, etc.)
- [ ] **TASK-83** Rate-limiting sur le formulaire de login (anti brute-force basique)

---

### ✅ PHASE 12 — Qualité & Tests

- [ ] **TASK-84** Valider chaque page HTML via le validateur W3C
- [ ] **TASK-85** Vérifier le temps de chargement < 3s par page (optimiser images, minifier assets)
- [ ] **TASK-86** Écrire les tests unitaires PHPUnit pour au moins un contrôleur (ex: `OffreController` ou `EntrepriseController`)
  - Tester les méthodes CRUD
  - Tester les validations
  - Tester les permissions
- [ ] **TASK-87** Vérifier la conformité PSR-12 du code PHP (ex: avec PHP_CodeSniffer)
- [ ] **TASK-88** Tester le responsive sur mobile, tablette, desktop
- [ ] **TASK-89** Audit SEO final (balises Hn, alt, URLs, sitemap, robots.txt)

---

## Contraintes techniques à respecter en permanence

- **Pas de framework PHP** (pas de Laravel, Symfony, etc.)
- **Pas de framework JS** (pas de React, Vue, Angular) — jQuery autorisé
- **Pas de CMS**
- **Architecture MVC stricte** — le contrôleur ne contient pas de HTML, la vue ne contient pas de logique métier
- **POO PHP obligatoire** — chaque entité a son Model, chaque section son Controller
- **Moteur de template** utilisé pour tous les fragments (header, footer, pagination, flash messages)
- **Clés étrangères SQL** déclarées explicitement
- **Deux vhosts Apache** : appli principale + statique
- **HTTPS** obligatoire en production

---

## Ordre de développement recommandé

```
PHASE 1 (infra) → PHASE 2 (BDD) → PHASE 3 (auth) →
PHASE 4+5 (entreprises + offres, cœur métier) →
PHASE 6+7 (pilotes + étudiants) →
PHASE 8+9 (candidatures + wishlist) →
PHASE 10 (frontend) → PHASE 11 (sécurité) → PHASE 12 (tests + qualité)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vladimir-Maj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Vladimir-Maj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
