## kt-start

> Gestionnaire de favoris web auto-hébergé. Refonte complète de l'ancienne version (~10 ans, fichiers `.ini`) avec la même stack que KT-Drop.

# KT-Start — Contexte projet pour Claude Code

## Présentation
Gestionnaire de favoris web auto-hébergé. Refonte complète de l'ancienne version (~10 ans, fichiers `.ini`) avec la même stack que KT-Drop.

## Stack technique
- PHP 8.3+ (`declare(strict_types=1)` partout)
- SQLite 3 via PDO
- Bootstrap 5.3 + Bootstrap Icons 1.11
- `vlucas/phpdotenv` ^5.6
- SortableJS (CDN) pour le drag & drop
- Architecture MVC, front controller unique (`public/index.php`), router maison
- Aucun framework PHP externe

## Architecture
```
src/
├── Config/
│   ├── BadgeStyles.php     # 12 styles de badge avec gradient() — deepBlue, turquoise, etc.
│   └── Config.php          # Accès à $_ENV
├── Controller/
│   ├── AdminController.php # index, usersPage, listsPage, foldersPage, settingsPage, backupPage, maintenancePage, tagsPage, statsPage, tagRename, tagDelete, userStore/Update/Delete, listStore/Rename/Delete/SetDefault, adminFolderStore/Rename/Delete/Reorder, settingUpdate, runMigration, exportBookmarks, exportFull, importBookmarks
│   ├── AuthController.php  # login, logout → redirige vers ?action=home
│   └── BookmarkController.php  # home(), index(), store(), update(), delete(), reorder(), fetchMeta(), checkDuplicate(), linksReport(), checkSingleLink(), deleteDeadLinks(), resetLinkStatus(), followRedirect(), folderStore(), folderRename(), folderDelete(), explorerReorder(), bookmarklet(), bookmarkletStore()
├── Core/
│   ├── Auth.php            # Session : ktstart_session (≠ ktdrop_session), isAdmin()
│   ├── Csrf.php
│   ├── Database.php        # Singleton PDO SQLite
│   ├── Flash.php
│   ├── Response.php
│   ├── Router.php          # Routes par ?action=xxx, non-auth → ?action=home
│   └── View.php            # render(), renderRaw() (sans layout — popup bookmarklet), e(), asset() → 'public/assets/...'
├── Repository/
│   ├── BookmarkRepository.php  # findPublic(), findPublicByList(), findFiltered(), findByUserAndListWithFolder(), CRUD, reorder(), setFolderAndPosition(), getAllTags(), getAllTagsAdmin(), renameTag(), deleteTag(), deleteTagsUsedOnce(), findByUrl(), findAllByUser(), updateCheckStatus(), resetCheckStatus(), updateUrl(), deleteMultiple(), countDeadLinksAll()
│   ├── FolderRepository.php    # findAllByUser(), findAllByUserInList(), findAllByListId(), findAllByListIdWithUser(), findById(), existsForUserInList(), create(), rename(), renameAdmin(), setParentAndPosition(), setParentAndPositionAdmin(), wouldCreateCycle(), wouldCreateCycleAdmin(), deleteAndLiftChildren(), deleteAndLiftChildrenAdmin(), countAll(), groupByParent()
│   ├── ListRepository.php      # findAll(), findByName(), create(), rename(), findAllWithCount(), findDefault(), setDefault(), clearDefault()
│   ├── SettingsRepository.php  # get(), set(), all() — table settings clé/valeur
│   ├── StatsRepository.php     # overview(), perUser(), perList(), perLinkStatus(), topTags(), perMonth(), perBadgeStyle()
│   └── UserRepository.php      # CRUD complet, emailExists()
└── Service/
    ├── ImportExportService.php # export() v1 favoris, exportFull() v2 backup, import() détection auto v1/v2
    ├── MigrationService.php    # Migrations idempotentes : PRAGMA + ALTER TABLE ADD COLUMN
    ├── UrlCheckService.php     # check() HEAD/GET cURL, statuts ok/redirect/error/timeout, getFinalUrl() suit les redirections, proxy DB→.env
    └── UrlMetaService.php      # curl + DOMDocument → title/host/description
```

## Schéma SQLite
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'admin',
    created_at TEXT NOT NULL
);

CREATE TABLE lists (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    is_default INTEGER NOT NULL DEFAULT 0,  -- 1 seule liste par défaut max
    created_at TEXT NOT NULL
);

CREATE TABLE bookmarks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    url TEXT NOT NULL,
    host TEXT,
    title TEXT,
    description TEXT,
    badge_style TEXT NOT NULL DEFAULT 'deepBlue',
    badge_text TEXT NOT NULL DEFAULT '',
    tags TEXT,           -- virgule-séparé ex: "php,dev"
    visibility TEXT NOT NULL DEFAULT 'private',  -- 'public' | 'private'
    list_id INTEGER REFERENCES lists(id),
    folder_id INTEGER REFERENCES folders(id),  -- dossier optionnel dans la liste
    user_id INTEGER NOT NULL REFERENCES users(id),
    position INTEGER DEFAULT 0,
    created_at TEXT NOT NULL,
    last_check_status TEXT,   -- 'ok' | 'redirect' | 'error' | 'timeout' | NULL (non vérifié)
    last_check_at TEXT        -- datetime de la dernière vérification
);

CREATE TABLE folders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    user_id INTEGER NOT NULL REFERENCES users(id),
    list_id INTEGER NOT NULL REFERENCES lists(id),
    parent_id INTEGER REFERENCES folders(id),  -- hiérarchie parent/enfant
    position INTEGER NOT NULL DEFAULT 0,
    created_at TEXT NOT NULL
);

CREATE TABLE settings (
    key   TEXT PRIMARY KEY,
    value TEXT NOT NULL
    -- ex: bookmarks_per_page = '24'
);
```

## Routing
Toutes les routes passent par `?action=xxx` :
- `home` (public) → favoris publics, non connecté
- `login` / `login_submit` (public)
- `logout`
- `bookmarks` → liste complète (auth requise)
- `bookmark_store` / `bookmark_update` / `bookmark_delete` (POST, auth)
- `bookmark_reorder` (POST JSON, auth) → drag & drop SortableJS
- `bookmark_fetch_meta` (GET, auth) → JSON {title, host, description}
- `bookmark_check_duplicate` (GET, auth) → JSON {found, bookmark} — vérifie doublon URL avec exclude_id optionnel
- `bookmark_links_report` (GET, auth) → page rapport accessibilité des liens
- `bookmark_check_single` (POST, auth) → JSON — vérifie une seule URL, met à jour last_check_status/at
- `bookmark_reset_status` (POST, auth) → remet last_check_status/at à NULL pour tous les favoris du user
- `bookmark_delete_dead` (POST, auth) → supprime une liste d'ids (favoris morts sélectionnés)
- `bookmark_follow_redirect` (POST JSON, auth) → suit le 301 d'un favori, met à jour url/host en DB, remet last_check_status à NULL
- `bookmark_folder_store` (POST, auth) → crée un dossier dans une liste
- `bookmark_folder_rename` (POST, auth) → renomme un dossier
- `bookmark_folder_delete` (POST, auth) → supprime un dossier et remonte ses enfants (favoris + sous-dossiers) au niveau parent
- `bookmark_explorer_reorder` (POST JSON, auth) → réorganise favoris et dossiers dans la vue Explorateur
- `bookmarklet` (GET, public) → popup autonome d'ajout rapide, reçoit `url` et `title` en GET, auth gérée dans le controller (3 états : non connecté / formulaire / succès)
- `bookmarklet_store` (POST, public) → enregistre le favori depuis la popup, affiche l'état succès avec bouton `window.close()`
- `admin` (GET, admin) → dashboard avec 9 cartes de navigation
- `admin_users` (GET, admin) → gestion utilisateurs
- `admin_lists` (GET, admin) → gestion listes
- `admin_folders` (GET, admin) → gestion dossiers (arbre hiérarchique par liste, drag & drop)
- `admin_settings` (GET, admin) → paramètres applicatifs
- `admin_backup` (GET, admin) → sauvegarde (export/import)
- `admin_maintenance` (GET, admin) → maintenance (migration + log)
- `admin_tags` (GET, admin) → gestion des tags (renommer/supprimer, tous utilisateurs)
- `admin_stats` (GET, admin) → statistiques globales
- `admin_user_store` / `admin_user_update` / `admin_user_delete` (POST, admin)
- `admin_list_store` / `admin_list_rename` / `admin_list_delete` (POST, admin)
- `admin_list_set_default` (POST, admin) → toggle liste par défaut
- `admin_folder_store` (POST, admin) → crée un dossier (admin, any list)
- `admin_folder_rename` (POST, admin) → renomme un dossier (admin)
- `admin_folder_delete` (POST, admin) → supprime un dossier (admin, remonte les enfants)
- `admin_folder_reorder` (POST JSON, admin) → réorganise/niche les dossiers, debounce 600ms, anti-cycle
- `admin_setting_update` (POST, admin) → mise à jour table settings
- `admin_run_migration` (POST, admin) → MigrationService, résultat en session flash
- `admin_tag_rename` (POST, admin) → renomme un tag dans tous les favoris
- `admin_tag_delete` (POST, admin) → supprime un tag de tous les favoris
- `admin_tags_cleanup` (POST, admin) → supprime tous les tags utilisés par un seul favori
- `admin_export` (GET, admin) → export favoris v1 (user courant), téléchargement JSON
- `admin_export_full` (GET, admin) → backup complet v2 (toutes tables), téléchargement JSON
- `admin_import` (POST, admin) → import v1 ou v2 selon `version` dans le fichier ; `full_restore=1` → purge toutes les tables + session_destroy + redirect login

## Environnement
- `.env` — config de base, commité
- `.env.local` — surcharges locales (non commité)
- `.env.local.example` — template pour `.env.local`
- Session PHP : `ktstart_session` (isolée de KT-Drop)
- `BOOKMARKS_PER_PAGE` — nombre de favoris par page, priorité : DB (`settings`) → `.env` → défaut 24

## Installation sur une nouvelle machine
```bash
composer install
cp .env.local.example .env.local
# Éditer .env.local selon l'environnement
php scripts/init-db.php    # crée les tables + user admin@example.com / changeme
```

## Apache (local)
Accès via subdirectoire : `http://localhost:8080/KT-Start/`
- `.htaccess` racine → redirige vers `public/index.php`
- `.htaccess` dans `public/` → rewrite vers `index.php`
- DocumentRoot Apache : `/Users/.../Sites` (commun à tous les projets)
- Pas de VirtualHost dédié

## Templates
- `templates/layout.php` — navbar fixe glassmorphism, footer fixe, back-to-top, icône maison → `?action=bookmarks` (tous users connectés), lien Admin (admin only, visible sur mobile)
- `templates/auth/login.php`
- `templates/bookmarks/index.php` — 4 vues (badges/table/liste/explorer), modal add/edit `.ks-modal`, bouton poubelle `.ks-quick-delete` inline sur chaque favori (y compris vue Explorer), modal de confirmation suppression `#deleteConfirmModal`, drag & drop, contrôle taille badges, recherche, pagination + bouton ∞ "tout afficher", nuage de tags collapse, détection doublon URL inline, `$readOnly` pour vue publique ; vue Explorer avec navigation hiérarchique par dossiers (`.ks-folder-section-header`) ; sélecteur de dossier dans le modal rendu hiérarchiquement via `$renderFolderOption` (préfixe `└`) ; autocomplétion tags multi-valeur (`<datalist>` dynamique) ; raccourci clavier `N` pour ouvrir le modal d'ajout
- `templates/bookmarks/links.php` — page rapport des liens (sans layout propre, rendu dans layout) ; résumé 4 compteurs, barre de progression, sections par statut, vérification URL par URL (JS async/await), boutons Continuer/Stop/Tout revérifier, sélection en lot pour suppression/mise à jour
- `templates/bookmarks/bookmarklet.php` — page popup autonome (sans layout), 3 états : `$notLogged` / formulaire / `$saved` ; champs URL+titre pré-remplis, liste par défaut pré-sélectionnée (`is_default`), sélecteur de dossier hiérarchique reconstruit en JS selon la liste choisie (masqué si aucun dossier), autocomplétion tags multi-valeur via `<datalist>` dynamique, couleur badge, visibilité
- `templates/admin/index.php` — dashboard 9 cartes de navigation (dont "Vérification des liens" avec badge rouge si liens morts, "Dossiers" avec compteur)
- `templates/admin/users.php` — gestion utilisateurs (table + modal add/edit + delete + confirmation mot de passe)
- `templates/admin/lists.php` — gestion listes (table + modal + bouton ⭐ défaut + filtre live)
- `templates/admin/folders.php` — gestion dossiers : sélecteur de liste, arbre hiérarchique récursif (`$renderTree`), drag & drop SortableJS (réorganisation + nidification), modals créer/renommer/supprimer, debounce 600ms, badge email propriétaire, anti-cycle (`wouldCreateCycleAdmin`)
- `templates/admin/settings.php` — paramètres (bookmarks_per_page, proxy + case à cocher `check_proxy_enabled`) + section bookmarklet avec bouton à glisser et code copiable (généré depuis `APP_URL`)
- `templates/admin/backup.php` — export v1/v2 + import + restauration complète
- `templates/admin/maintenance.php` — migration runner + journal
- `templates/admin/tags.php` — table triée par fréquence, filtre live, modal renommer, bouton supprimer, bouton "Nettoyer (N unique(s))"
- `templates/admin/stats.php` — 6 cartes résumé, graphique barres mensuel (12 mois, CSS pur), répartitions par liste / statut liens / top tags / utilisateur / style de badge

## Conventions CSS
Fichier : `public/assets/css/app.css`
- Variables : `--app-blue: #0288D1`, `--app-blue-soft`, `--app-blue-hover`
- Boutons : `--bs-btn-border-radius: 10px` global, overrides Bootstrap variables sur `.btn-primary`
- Classes badge : `.ks-badge`, `.ks-badge-thumb` (3 couches Liquid Glass : `::before` reflet, `::after` overlay, `inset box-shadow`), `.ks-badge-footer`
- Taille badge : propriété CSS custom `--ks-badge-width` sur `.ks-badges-grid`, 6 paliers XS (80px) → XXL (240px), défaut L (160px), persisté en `localStorage`
- Hover badge : `--ks-badge-color` injecté inline sur `.ks-badge` (couleur `bg` du style) ; `box-shadow` et `border-color` au survol utilisent `color-mix(in srgb, var(--ks-badge-color) …)` — ombre colorée propre à chaque badge
- Classes liste compacte : `.ks-compact-item`
- Dropdown liste : `.ks-list-dropdown-items` (max-height + scroll) — remplace les anciens pills `.ks-list-tab`
- Modaux unifiés : `.ks-modal` (style Apple/Tesla, fond flouté, coins arrondis, séparateurs subtils)
- Drag & drop : `.ks-drag-handle` (visible uniquement en sort=position)
- Admin : `.ks-admin-card`, `.ks-admin-icon`, `.ks-migration-log`
- Pagination : `.ks-pagination`
- Recherche : `.ks-search-input`
- Vue tableau et vérification des liens : classe `ks-table` sur `<table>` → `thead th` reçoit `background:#f8f9fa` (clair) / `#333740` (sombre), cohérent avec les tables admin
- Barre d'outils (Tri, Liste, Tags, vérification liens, sélecteur de vue) : bordures unifiées sur `--app-border` dans tous les états (neutre/hover/actif) — seule la couleur de l'icône change en bleu au survol/sélection
- Indicateur statut lien : `.ks-link-dot` — position de base sans `position` ni coordonnées ; `.ks-badge-thumb .ks-link-dot` → `position:absolute; bottom:6px; right:6px; z-index:3` (règle plus spécifique que `.ks-badge-thumb span { position:relative }`) ; vues tableau/liste → `position:static` via `.ks-compact-item .ks-link-dot, td .ks-link-dot`
- **Badge visibilité "Privé"** : classe `.ks-badge-private` (pas `text-bg-light`) — fond gris clair en mode clair, `#2e3139` en mode sombre via `[data-theme="dark"] .ks-badge-private`
- **Bouton poubelle inline** : `.ks-quick-delete` sur les 3 vues — gris atténué par défaut (opacity .3), rouge au survol ; handler délégué sur `document` ; ouvre `#deleteConfirmModal` (jamais `confirm()` natif)
- **Modal de confirmation suppression** : `#deleteConfirmModal` Bootstrap initialisé en lazy (`bootstrap.Modal.getOrCreateInstance()`) — JAMAIS en top-level IIFE car Bootstrap JS est chargé après le template dans le layout
- **Mode sombre** : `[data-theme="dark"]` sur `<html>` + `data-bs-theme="dark"` pour Bootstrap ; script inline `<head>` applique le thème avant le rendu (évite le flash) ; toggle `🌙/☀️` dans la navbar ; persisté dans `localStorage` sous `ks-theme` ; fond sombre uni (`#22242a`, pas de gradient — évite les bandes au scroll) ; palette bleu-grisée ~`#22242a`→`#3a3e4a`
- **Toggle liste dropdown/boutons** : `data-list-nav` sur `<html>` (valeurs `dropdown` | `buttons`) — script inline synchrone avant la toolbar pose l'attribut depuis `localStorage['ks-list-nav']` (défaut `buttons`) avant le premier rendu (zéro flash, même technique que le dark mode) ; CSS `html[data-list-nav="..."]` masque/affiche `#ks-list-filter-dropdown` et `#ks-list-nav-buttons` avec `!important` pour surcharger `d-flex` ; le JS du toggle met à jour `dataset.listNav` + localStorage et anime la transition (fade + translateY, 220 ms) côté JS pour que la sortie et l'entrée aient exactement le même ressenti ; bouton `#btnListNavToggle` positionné entre Tri et Liste dans la toolbar
- **Bootstrap JS chargé après le template** : `bootstrap.bundle.min.js` est en bas du layout (après `</main>`) — tout appel à `new bootstrap.Modal()` au top-level d'un script inclus dans le template lève `ReferenceError`. Utiliser `bootstrap.Modal.getOrCreateInstance()` ou `bootstrap.Modal.getInstance()` uniquement dans des handlers d'événements
- **Bookmarklet** : `View::renderRaw()` pour les pages sans layout ; routes `bookmarklet`/`bookmarklet_store` publiques, auth gérée dans le controller ; code JS généré dans `admin/settings.php` depuis `APP_URL` (`$_ENV['APP_URL']`)
- Même structure visuelle que KT-Drop (dominante bleue au lieu d'orange)

## Points techniques importants
- **LIMIT/OFFSET** : interpolés directement dans la requête SQL (pas de `bindValue`) car SQLite rejette les strings pour LIMIT — les valeurs viennent du calcul interne, pas de l'utilisateur
- **getAllTags()** : `$tags = array_keys($tags); sort($tags);` sur deux lignes — `sort()` attend une référence, impossible en expression compacte
- **Migration log** : `$_SESSION['_migration_log']` utilisé pour passer le résultat de migration entre redirect et affichage
- **Drag & drop** : SortableJS uniquement actif en mode `sort=position`, handles `.ks-drag-handle` masqués sinon
- **Liquid Glass** : `.ks-badge-thumb::before` (radial-gradient spéculaire) + `::after` (linear-gradient directionnel) + `inset box-shadow` — ne pas utiliser `overflow: hidden` sur `.modal-content` (blur des textes)
- **Liste par défaut** : `is_default` sur `lists`, une seule liste à 1 max — `findDefault()` retourne null si colonne absente (migration pas encore lancée) via try/catch
- **Sentinel list=0** : dans l'URL, `list=0` signifie "Toutes" explicitement choisi (pas de filtre). Absence du paramètre `list` → appliquer la liste par défaut. `$listRaw` est transmis au template pour construire les URLs de pagination/recherche sans perdre ce choix
- **Tout afficher** : `perpage=all` dans l'URL désactive la pagination (PHP_INT_MAX en LIMIT). `$showAll` transmis au template — préservé dans le formulaire de recherche et les liens de pagination
- **Compteur** : affiche `X / Y favoris` quand X (page courante) < Y (total filtré), sinon juste `Y favoris`
- **Priorité bookmarks_per_page** : DB (`settings.bookmarks_per_page`) → `$_ENV['BOOKMARKS_PER_PAGE']` → 24
- **SettingsRepository** : utilise `INSERT ... ON CONFLICT(key) DO UPDATE` (upsert SQLite) — `get()` retourne `''` si table absente (try/catch)
- **Tri badge_text** : `orderBy()` dans `BookmarkRepository` — cas `badge_text` → `LOWER(b.badge_text) ASC`
- **Import/Export** : `ImportExportService` — deux formats JSON :
  - v1 (`ktstart-bookmarks-*.json`) : lists + bookmarks user courant — à l'import : DELETE bookmarks (user) + DELETE lists + reset `sqlite_sequence` → réinsère
  - v2 (`ktstart-backup-*.json`) : users + settings + lists (avec `is_default`) + bookmarks tous users — import normal = merge ; `full_restore=1` = purge totale + reset `sqlite_sequence` + session_destroy
  - `badge_style` non validé à l'import (préservation des anciens styles hérités)
  - `is_default` exporté en v2 en objet `{name, is_default}`, restauré via `setDefault()` après création des listes
  - `sqlite_sequence` réinitialisée via `DELETE FROM sqlite_sequence WHERE name IN (...)` pour repartir à id=1
- **Nuage de tags** : collapse Bootstrap `#tagCloud` dans la vue bookmarks, triés par fréquence décroissante (`arsort`), taille proportionnelle 0.78–1.6rem, max-height 140px + overflow-y auto
- **Détection de doublons URL** : `bookmark_check_duplicate` → GET JSON `{found, bookmark}`, `findByUrl()` avec `exclude_id` pour le mode édition — alerte inline sous le champ URL en mode add/edit
- **Persistance liste sélectionnée** : `$_SESSION['ktstart_list']` mémorisé à chaque sélection explicite de liste, restauré si retour sur `?action=bookmarks` sans paramètre `list`
- **Tags cleanup** : `deleteTagsUsedOnce()` dans `BookmarkRepository` — supprime d'un coup tous les tags apparaissant dans un seul favori ; bouton "Nettoyer (N unique(s))" sur la page `admin_tags`
- **Vérification des liens** : `UrlCheckService::check()` — HEAD (fallback GET si 405), `getFirstHttpCode()` sans suivre les redirections pour détecter les 301 ; statuts `ok/redirect/error/timeout` stockés dans `last_check_status` + `last_check_at` ; vérification URL par URL depuis le JS (async/await enchaîné) ; indicateurs `.ks-link-dot` sur les 4 vues (point bas-droite dans le badge coloré) ; rapport groupé par statut avec suppression en lot (morts) et mise à jour URL (redirigés) ; barres d'action `position:fixed` au-dessus du footer (`z-index:1025`, `bottom:calc(var(--app-footer-height)+.5rem)`), fond opaque : `#fff5f5` (rouge) / `#fffdf0` (jaune)
- **Mise à jour URL redirigées** : `UrlCheckService::getFinalUrl()` suit les redirections (`CURLOPT_FOLLOWLOCATION`, `CURLINFO_EFFECTIVE_URL`) ; `BookmarkRepository::updateUrl()` met à jour url + host et remet last_check_status à NULL ; endpoint `bookmark_follow_redirect` traite un favori à la fois via JS fetch
- **Dashboard admin — liens morts** : `BookmarkRepository::countDeadLinksAll()` compte les favoris en statut `error` ou `timeout` tous utilisateurs confondus ; `$deadLinkCount` passé au template ; badge rouge affiché sur la carte "Vérification des liens" si > 0
- **Proxy vérification liens** : `CHECK_PROXY` — priorité DB (`settings.check_proxy`) → `.env` → vide ; configurable depuis Admin → Paramètres sans redémarrage ; `check_proxy_enabled` (DB, `0`/`1`) — case à cocher pour désactiver sans effacer la valeur
- **Robustesse migration** : `countDeadLinksAll()`, `StatsRepository::overview()` et `perLinkStatus()` protégés par `try/catch` — retournent 0 si `last_check_status` absent (migration non encore lancée), évite le fatal error sur `?action=admin`
- **StatsRepository** : utilise `Database::connection()` (pas `getInstance()` qui n'existe pas) ; tous les chiffres calculés côté SQL sauf `topTags()` qui agrège les chaînes virgule-séparées en PHP
- **Mode sombre — fond** : `background-attachment: fixed` conservé en mode clair seulement ; en mode sombre, fond couleur unie sans gradient (les `radial-gradient` fixes créent des bandes répétées visibles au scroll sur fond sombre)

- **Dossiers** : table `folders` avec hiérarchie parent/enfant (`parent_id` nullable) ; `FolderRepository::wouldCreateCycle()` prévient les boucles (user) / `wouldCreateCycleAdmin()` sans contrainte user (admin) ; `deleteAndLiftChildren()` remonte les sous-dossiers et favoris d'un niveau (pas de suppression en cascade) ; Vue Explorer (`view=explorer`) — 4ème vue avec navigation par dossiers, `foldersByParent` + `bookmarksByFolder` passés au template ; `bookmark_explorer_reorder` reçoit un tableau JSON `{folders:[…], bookmarks:[…]}` et met à jour les positions + folder_id via `setParentAndPosition()` et `setFolderAndPosition()`
- **Hiérarchie vues badges/tableau/liste** : rendu récursif via `$renderFolderLevel` / `$renderTableFolderRows` / `$renderListFolderSections` ; indentation par `.ks-folder-nested-content` (bordure gauche bleue + padding) en badges/liste ; `border-left` + `padding-left` inline en tableau ; compteur récursif `$countBmRecursive` sur les badges dossiers ; `$countExplorerRecursive` (même logique, utilise `$bookmarksByFolder`) sur les dossiers de la vue Explorer ; titre du dossier masqué sous taille L via `clamp(0px, (width-145)*9999, 1.5rem)` ; bouton `📁+` dans le footer badge pour créer un sous-dossier
- **Responsive badges dossiers** : `.ks-folder-group` suit les mêmes breakpoints que `.ks-badge` (`calc(50% - .4rem)` à < 576px, `calc(33.33% - .5rem)` à 576–767px) — le `.ks-badge` interne passe à `width:100%`
- **Cache-busting assets** : `View::asset()` ajoute `?v={filemtime}` — le navigateur invalide le cache automatiquement à chaque modification
- **Page admin dossiers** (`admin_folders`) : SortableJS chargé dans le template (pas dans le layout) ; debounce 600ms sur `saveOrder` pour fusionner les drops rapides en une seule requête ; `serializeTree()` lit `data-parent-id` sur chaque `<ul>` pour reconstruire la hiérarchie complète
- **Debounce D&D badges** : `scheduleBadgeSave(container)` accumule les conteneurs dans un `Set` + timer 300ms avant d'appeler `saveBadgesContainer()` — évite les requêtes parallèles en cas de déplacements rapides entre conteneurs
- **Sélecteur de dossier hiérarchique** : `$renderFolderOption(int $pk, int $depth)` fermeture récursive dans le modal bookmark — options préfixées `└ ` avec indentation `str_repeat('  ', $depth)` selon la profondeur ; `$byParentForSelect` construit localement dans le modal
- **Autocomplétion tags multi-valeur** : `<datalist>` reconstruit à chaque événement `input`/`focus` — le préfixe (tout avant la dernière virgule) est recalculé et injecté dans les `value` des options pour que la sélection complète la valeur correctement ; tags déjà saisis exclus de la liste
- **Bookmarklet — sélecteur de dossier JS** : `buildOptions(parentId, listId, depth, selectEl)` construit récursivement les `<option>` en JS depuis `allFolders` (JSON injecté depuis PHP) ; appelé par `refreshFolders()` à chaque changement de liste ; le bloc `#bmlFolderWrap` est masqué si la liste n'a aucun dossier
- **Raccourci clavier `N`** : `keydown` sur `document` — ignoré si `activeElement` est `input`, `textarea`, `select` ou `contentEditable` ; simule un clic sur le bouton `[data-mode="add"]`
## Ce qui reste à faire (non implémenté)
- ~~Page de statistiques (répartition par liste, tag, visibilité)~~ ✓ implémenté
- ~~Vérification automatique des favoris inaccessibles (lien mort)~~ ✓ implémenté
- ~~Bookmarklet (ajout rapide depuis le navigateur)~~ ✓ implémenté
- ~~Sous-dossiers dans les vues badges/tableau/liste~~ ✓ implémenté
- ~~Gestion des dossiers dans l'administration~~ ✓ implémenté
- Rôle `editor` (multi-utilisateurs sans accès admin)
- Import de favoris au format HTML (Netscape bookmarks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aKelleter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
