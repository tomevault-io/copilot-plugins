## gestion-des-stock-magasin-local

> - **Framework** : Laravel 12 (PHP 8.2+)

# Projet : Gestion de Stock AEB

## Stack technique
- **Framework** : Laravel 12 (PHP 8.2+)
- **Base de données** : MySQL via XAMPP
- **Frontend** : Blade templates + Vite
- **Import Excel** : maatwebsite/excel 3.1
- **Serveur local** : XAMPP (http://localhost/gestion-stock-AEB/public)

## Structure principale

```
app/
  Models/         Article, Emplacement, Metier, Operation, SousOperation,
                  Mouvement, Reapprovisionnement, Alerte, QrCode, Utilisateur
  Http/Controllers/
    ArticleController.php      — CRUD articles + recherche + entrée de stock
    ExcelImportController.php  — Import fichiers Excel
  Imports/        — Classes Maatwebsite pour lecture Excel

resources/views/
  articles/       index, show, create, edit, search
  layouts/        layout principal
  home.blade.php
  importer.blade.php

routes/web.php    — toutes les routes (pas d'API REST pour l'instant)
database/migrations/
```

## Modèle Article (table `articles`)
Champs : `mabec`, `designation`, `reference_fabricant`, `fabricant`, `criticite`,
`photo`, `stock_actuel`, `fk_emplacement`, `fk_metier`, `fk_operation`, `fk_sous_operation`

Relations : `emplacement`, `metier`, `operation`, `sousOperation`, `mouvements`,
`reapprovisionnements`, `alertes`, `qrCodes`

## Routes principales
- `GET /` — Dashboard (compteurs articles, métiers, emplacements, stock positif)
- `GET /articles` — Liste des articles
- `GET /articles/recherche` — Recherche avancée style GMAO
- `GET /articles/create` — Formulaire création
- `GET /articles/{id}` — Détail article
- `GET /articles/{id}/edit` — Formulaire édition
- `GET /articles/{id}/entree` — Entrée de stock
- `GET /articles/mabec/{mabec}` — Recherche par code MABEC
- `GET /importer-excel` — Import Excel (formulaire + analyse + confirmation)

## Conventions du projet
- Clés étrangères nommées `fk_*` (ex: `fk_emplacement`, `fk_metier`)
- Pas d'authentification en place pour l'instant
- Le champ `mabec` est l'identifiant métier unique d'un article

---
> Source: [AnassGE/Gestion-des-stock-magasin-local](https://github.com/AnassGE/Gestion-des-stock-magasin-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
