## joshcoast2026

> This is a WordPress 6.9.x website project ("JoshCoast 2026"). The repo tracks only custom themes/plugins under `wp-content/`; WordPress core is `.gitignore`-d and must be installed separately via WP-CLI.

# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

This is a WordPress 6.9.x website project ("JoshCoast 2026"). The repo tracks only custom themes/plugins under `wp-content/`; WordPress core is `.gitignore`-d and must be installed separately via WP-CLI.

### Stack

- **CMS:** WordPress 6.9.x
- **Language:** PHP 8.3
- **Database:** MariaDB 10.11
- **Web Server:** Apache 2 with `mod_rewrite`
- **CLI:** WP-CLI 2.12+

### Starting Services

After the update script runs, start services before doing any work:

```bash
sudo service mariadb start
sudo service apache2 start
```

### WordPress Admin

- URL: `http://localhost/wp-admin/`
- Username: `admin`
- Password: `admin123`

### Key Commands

| Task | Command |
|---|---|
| PHP lint | `php -l <file.php>` |
| DB health check | `wp db check --allow-root` |
| List posts | `wp post list --allow-root` |
| Create post | `wp post create --post_title="Title" --post_content="Content" --post_status=publish --allow-root` |
| WordPress version | `wp core version --allow-root` |
| Update WordPress | `wp core update --allow-root` |

### Gotchas

- WordPress core files live in `/workspace` but are `.gitignore`-d. Only `wp-content/` custom code is tracked.
- `wp-config.php` is also `.gitignore`-d. It is generated during setup with `wp config create`.
- Apache DocumentRoot is configured at `/etc/apache2/sites-available/wordpress.conf` pointing to `/workspace`.
- File ownership: `wp-content/` must be owned by `www-data` for uploads and plugin installs to work (`sudo chown -R www-data:www-data /workspace/wp-content/`).
- All `wp` CLI commands need `--allow-root` when running as root user.

---
> Source: [joshcoast/joshcoast2026](https://github.com/joshcoast/joshcoast2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
