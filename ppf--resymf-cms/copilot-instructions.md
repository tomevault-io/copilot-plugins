## resymf-cms

> ReSymf-CMS is a Symfony 7.3 CMS with Vue 3 islands and Tailwind CSS.

# Claude Code Instructions for ReSymf-CMS

## Project Overview

ReSymf-CMS is a Symfony 7.3 CMS with Vue 3 islands and Tailwind CSS.

**Stack:** PHP 8.5 | Symfony 7.3 | Doctrine ORM 3 | Vue 3 | Vite | Tailwind CSS | MariaDB 11

## Code Patterns

### Entity Pattern
- Location: `src/Entity/`
- Use Doctrine attributes (not annotations)
- Table names: `resymf_*` prefix
- Include: `createdAt` (immutable), `updatedAt` (nullable with PreUpdate)
- Fluent interface (return `static`)
- Reference: `src/Entity/Category.php`

### Admin Controller Pattern
- Location: `src/Controller/Admin/`
- Route prefix: `/admin/{plural}`
- Security: `#[IsGranted('ROLE_ADMIN')]`
- Methods: index, new, show, edit, delete, toggleActive
- CSRF: Always validate on POST actions
- Reference: `src/Controller/Admin/CategoryController.php`

### Vue Island Pattern
- Entry point: `assets/vue/{feature}-app.js`
- Component: `assets/vue/components/{Feature}.vue`
- Composable: `assets/vue/composables/use{Feature}.js`
- Mount: `<div data-vue-island="{feature}" data-api-url="...">`
- Register in `vite.config.mjs` rollupOptions.input
- Reference: `assets/vue/category-form-app.js`

### Form Type Pattern
- Location: `src/Form/`
- Bootstrap classes: `form-control`, `form-check-input`
- Include `help` for field descriptions
- Reference: `src/Form/CategoryType.php`

## Docker Commands

```bash
# Start environment
docker-compose up -d --build

# Run Symfony commands
docker-compose exec php php bin/console <command>

# Run tests
docker-compose exec php bin/phpunit

# Frontend dev (HMR)
npm run dev
```

## Testing

Run all tests before committing:
```bash
docker-compose exec php bin/phpunit
docker-compose exec php vendor/bin/phpstan analyse
docker-compose exec php vendor/bin/php-cs-fixer fix --dry-run
```

## Commit Format

```
{issue-number} - Brief description

Example: 31 - Add user authentication
```

Do not include Claude attribution in commits.

## Key Files

| Purpose | File |
|---------|------|
| Docker | `docker-compose.yml` |
| Vite config | `vite.config.mjs` |
| Security | `config/packages/security.yaml` |
| Routes | `config/routes/` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ppf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ppf)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
