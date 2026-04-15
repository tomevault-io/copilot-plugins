## blog-web

> CMS Symfony pret a vendre. Un site propre, securise, avec SEO integre, clonable en 30 minutes pour chaque nouveau client. On vend du service (installation + personnalisation), pas du SaaS.

# Blog & Web — CMS Symfony

## Objectif

CMS Symfony pret a vendre. Un site propre, securise, avec SEO integre, clonable en 30 minutes pour chaque nouveau client. On vend du service (installation + personnalisation), pas du SaaS.

## Stack technique

- **Backend** : PHP 8.4 / Symfony 7.4 LTS
- **ORM** : Doctrine ORM 3.3 + Migrations
- **Admin** : EasyAdmin Bundle 4.12
- **Frontend** : Webpack Encore + Bootstrap 5.3 + Stimulus/Hotwire
- **Templates** : Twig 3
- **BDD** : MariaDB 11
- **Infra** : Docker (PHP-FPM 8.4 + Nginx + MariaDB 11 + Mailpit)
- **Mailer** : Brevo (via `symfony/brevo-mailer`)

## Documentation detaillee

La spec technique complete, l'architecture, les phases de dev et la roadmap sont dans `.claude/docs/` :

| Fichier | Contenu |
|---------|---------|
| `.claude/docs/SPEC.md` | Spec technique complete (architecture, phases, conventions, structure) |
| `.claude/docs/CLAUDE2.md` | Extension spec — dette technique, modules, corrections |
| `.claude/docs/CLAUDE_FULL.md` | Roadmap long terme (multi-tenant, Stripe, IA, provisioning) |
| `.claude/docs/DESIGN_THEME.md` | Specs design des themes |

> **Ces fichiers sont supprimes automatiquement chez les clients** par `app:client:setup` et `scripts/deploy.sh`.

## Conventions de code

### General
- PHP 8.4, typed properties, readonly, enums
- Symfony 7.4 LTS, attributs PHP 8 (`#[Route]`, `#[ORM\Entity]`, `#[Assert\...]`)
- PascalCase classes, camelCase methodes/variables, snake_case BDD

### Architecture
- **Jamais** `->find(1)` → `SiteContext::getCurrentSite()`
- **Jamais** `findAll()` dans les controllers → methodes Repository
- **Toujours** ownership check avant modif d'une ressource utilisateur
- **Toujours** `HtmlSanitizer` sur contenu rendu avec `|raw`
- **Toujours** verifier `site.owner` pour les actions ROLE_FREELANCE

### Securite
- CSRF active globalement
- `denyAccessUnlessGranted()` sur toute route sensible
- Password min 12 caracteres, hash `auto`
- reCAPTCHA v3 optionnel sur le formulaire de contact
- Verification email obligatoire (ROLE_ADMIN+ exempt)

### Roles

```
ROLE_USER < ROLE_AUTHOR < ROLE_ADMIN < ROLE_FREELANCE < ROLE_SUPER_ADMIN
```

| Role | Qui | Acces |
|------|-----|-------|
| `ROLE_USER` | Visiteur inscrit | Lecture, commentaires, profil |
| `ROLE_AUTHOR` | Redacteur | Creation/edition articles et pages |
| `ROLE_ADMIN` | Admin client | Gestion complete du site |
| `ROLE_FREELANCE` | Freelance revendeur | Themes, apparence, gestion multi-site |
| `ROLE_SUPER_ADMIN` | David | Acces total, modules, infrastructure |

### Front
- SCSS avec CSS custom properties (pas de couleurs hardcodees)
- Stimulus pour le JS interactif
- Bootstrap 5 personnalise via custom properties
- `loading="lazy"` systematique sur les images
- Images responsives WebP auto (480w, 800w, 1200w)

## Commandes utiles

```bash
make up              # Lance Docker
make down            # Stop
make sh              # Shell PHP
make db              # Reset BDD (drop + create + migrate)
make migrate         # Migrations seulement
make assets          # npm run dev
make assets-build    # npm run build (prod)
make cc              # Cache clear
```

## Installation client

Voir `SETUP.md` pour le process complet. En resume :

```bash
git clone git@repo:blog_web.git /var/www/clients/client-x
cd /var/www/clients/client-x
cp .env.local.example .env.local   # Editer BDD, APP_SECRET, MAILER_DSN
make up && make db && make assets
docker compose exec php php bin/console app:client:setup
```

## Structure

```
blog_web/
├── docker/                  # Dockerfile, nginx, php.ini
├── scripts/                 # deploy.sh, backup.sh
├── assets/                  # JS/SCSS (app + admin entries)
├── src/
│   ├── Command/             # CLI (client:setup, module:enable, etc.)
│   ├── Controller/Admin/    # 19 CrudControllers EasyAdmin
│   ├── Entity/              # Doctrine entities
│   ├── Service/             # SiteContext, ThemeService, SeoService, etc.
│   └── EventSubscriber/     # PageViewSubscriber, ContentSanitize, etc.
├── templates/
│   ├── themes/              # 6 themes (default, corporate, artisan, etc.)
│   ├── admin/               # Dashboard, guide, menu manager
│   └── ...                  # Front templates
├── .claude/docs/            # Spec technique (dev only, supprime chez clients)
├── SETUP.md                 # Process installation
└── README.md                # Presentation du projet
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laigneletdavid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
