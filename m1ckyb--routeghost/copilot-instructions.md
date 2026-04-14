## routeghost

> - The user allows the use of tools like `docker` and `gh` without requiring manual confirmation for every session once initially allowed.

# Gemini Added Memories
- The user allows the use of tools like `docker` and `gh` without requiring manual confirmation for every session once initially allowed.

# Gemini Onboarding - RouteGhost

## Project Overview
**RouteGhost** is a Python/Flask application designed to securely expose local services (like Jellyfin, Sonarr, Radarr) to the internet temporarily. It uses a combination of:
- **Traefik** (via Redis provider) for dynamic routing.
- **Cloudflare** for DNS records and Origin Rules (port rewrites).
- **UniFi UDM Pro** for firewall control (opening/closing ports).
- **Home Assistant** for state updates.

## Architecture
- **Backend**: Flask (`main.py`) handles the web UI and API.
- **Database**: SQLite (`database.py`) stores configuration, users, services, and auth credentials.
- **Frontend**: HTML/CSS/JS (`templates/`, `static/`).
- **Background Tasks**: Threading in `main.py` handles health checks.
- **External Integrations**:
    - **Redis**: Stores Traefik routing configuration.
    - **Cloudflare API**: Manages DNS A records and Origin Rules.
    - **UniFi Controller API**: Toggles port forwarding rules.
    - **Home Assistant API**: Updates entity states.

## Key Files
- `main.py`: Core application logic, API endpoints, background threads, and external API interactions.
- `database.py`: Database schema, migrations, and CRUD operations.
- `templates/`: Jinja2 HTML templates.
    - `base.html`: Layout and common JS/CSS.
    - `index.html`: Main dashboard.
    - `settings.html`: Configuration page (tabbed).
    - `login.html`: Authentication page.
- `requirements.txt`: Python dependencies.

## Development Guidelines

### Security & Vulnerability Testing
Every new feature or code change must be evaluated for security risks.
- **CSRF Protection**: All new state-changing endpoints (POST, PUT, DELETE) **must** include `@login_required` (or API key auth) and ensure the `X-CSRFToken` header is checked and provided by the frontend.
- **Input Validation**: Use `ipaddress` library for IPs, `urlparse` for URLs, and regex for subdomains. Never trust user input.
- **Database**: Always use parameterized queries (e.g., `WHERE key = ?`). Never use string formatting for SQL.
- **Permissions**: Ensure the app continues to run correctly as a non-root user. Test volume persistence using named volumes.
- **Audit**: After significant changes, run a manual grep-based audit for hardcoded secrets or unsafe function usage (e.g., `shell=True`).

### Infrastructure & Persistence
- **Named Volumes**: Use the `routeghost_data` named volume for all persistent data. Avoid host-path mounts in production to prevent permission translation issues (especially on Windows/WSL2).
- **Non-Root Execution**: The container must run as `appuser`. Use `PUID` and `PGID` environment variables to manage file ownership via the `entrypoint.sh` script.

### Database Migrations
- The database schema is versioned in `database.py` inside `init_db()`.
- When adding columns or tables, increment the schema version and add a migration block checking `current_version`.

### Authentication
- Supports **Passkeys** (WebAuthn) and **Password** login.
- **2FA (TOTP)** is supported and can be enforced globally.
- **API Keys** allow programmatic access (`X-API-Key` header).
- Use `@login_required` for UI routes and `@api_key_or_login_required` for API endpoints.

### Configuration
- Settings are stored in the `settings` table in SQLite.
- Accessed via `db.get_setting(key)`.
- Configured via the Web UI (`/settings`) or the `/api/settings` endpoint.

### Logging
- Logs are written to `app.log` in the `DATA_DIR`.
- Output is handled via the `Tee` class in `main.py` to ensure logs reach both the console and the log file.

### Error Handling
- API endpoints should return JSON with `error` keys on failure.
- UI routes should use `flash()` messages.

## Common Tasks

### Adding a New Setting
1. Add the input field in `templates/settings.html`.
2. The `settings` route in `main.py` automatically saves all form data to the DB, so no backend change is usually needed for simple string settings unless validation logic is required.
3. Use `db.get_setting('KEY')` in `main.py` to use the value.

### Adding a New Feature
1. If it requires DB changes, update `database.py`.
2. Add backend logic/routes in `main.py`.
3. Update templates if UI is needed.

### Security
- Always check permissions.
- Validate inputs (especially URLs and IP addresses).
- Ensure sensitive data (passwords, tokens) is handled securely.

## Context for Gemini

When working on this project, assume the user wants robust, production-ready code. Pay attention to:

- **Error handling**: External APIs (Cloudflare, UniFi) can fail.

- **User Experience**: The UI should be responsive and provide feedback (toasts/alerts).

- **Security**: This app controls firewall rules, so security is paramount.



## Workflow & Changelog

**CRITICAL**: Every time you make a change to the codebase that affects functionality, user experience, or configuration (features, bug fixes, refactoring, style updates), you **MUST** follow these steps:

1.  **Update `unreleased.md`**: 
    - **Format**: Follow the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) convention.
    - **Categories**: Use sub-headers like `### Added`, `### Changed`, `### Fixed`, `### Removed`.
    - **Content**: Be concise but descriptive. Explain *what* changed and *why*.
    - **Process**: Perform the `unreleased.md` update in the same turn as the code changes.

2.  **Rebuild Local Container**:
    - After making changes, always rebuild and restart the local development container to verify the fix/feature.
    - Command: `docker-compose -f docker-compose-dev.yml up -d --build`

3.  **Git Push Restriction**:
    - **NEVER** push code to GitHub (e.g., `git push`) unless the user explicitly instructs you to do so.
    - Only commit changes locally unless told otherwise.

## Experimental: Non-Root & Permission Handling (test-code branch)

The `test-code` branch contains an experimental setup for running the container as a non-root user while maintaining volume write access.

- **Non-Root User**: The image creates an `appuser` and `appgroup`.
- **Configurable IDs**: Supports `PUID` and `PGID` environment variables (defaulting to 1000) to match host file ownership.
- **Runtime Permission Fix**: Uses an `entrypoint.sh` script and `su-exec`. On startup, the container:
    1. Runs as `root`.
    2. Adjusts `appuser` UID/GID to match the environment.
    3. Recursively `chown`s `/app/data` and `/app`.
    4. Drops privileges and executes the app as `appuser`.

### ⚠️ WSL2 Permission Caveat
When running this setup on a **Windows host via WSL2 mount** (`/mnt/c/...`), you may encounter `sqlite3.OperationalError: attempt to write a readonly database`. This is a known issue where Windows host mounts do not correctly support Linux permission changes or SQLite file locking.

**Recommendation**: This setup is designed for **Native Linux** environments with native volumes (e.g., ext4). For WSL2 development, stick to the `dev` branch or move the project into the native WSL filesystem (e.g., `~/code/`) rather than the mounted Windows drive.

## Release Process

When requested to "Make a dev release":

1.  **Push to Dev**: Push the current code to the `dev` branch.

When requested to "Make a release", where `<type>` is Patch, Minor, or Major, the following steps must be performed on the **`dev` branch** based on Semantic Versioning:

1.  **Determine New Version**: Read the current version from `VERSION.txt` (e.g., X.Y.Z).
    - For a Patch release, the new version will be X.Y.(Z+1).
    - For a Minor release, the new version will be X.(Y+1).0.
    - For a Major release, the new version will be (X+1).0.0.

2.  **Update `CHANGELOG.md`**:
    - Create a new version heading with the new version number and current date (e.g., `## [1.0.0] - YYYY-MM-DD`).
    - Move **only** the content from `unreleased.md` that hasn't been released yet into this new section.
    - **CRITICAL**: Ensure you do not duplicate entries already present in older versions of `CHANGELOG.md`.
    - Do not add an `[Unreleased]` section back to the top of `CHANGELOG.md`.

3.  **Clear `unreleased.md`**: After moving the content, reset `unreleased.md` to an empty state (or just the sub-headers) to prevent those changes from being included in the next release.

4.  **Update `VERSION.txt`**: Change the content of `VERSION.txt` to the new version number.

5.  **Update `docker-compose.yml`**: Update the image tags for the services to the new version number.

6.  **Update Documentation**: Review `README.md` and other docs to reflect new features or significant changes.

7.  **Push to Dev**: Commit and push all release-related changes to the `dev` branch.

8.  **Merge to Main**: Checkout `main`, merge `dev`, and push to `origin main`.

9.  **GitHub Release**: Use the `gh` CLI to create a release/pre-release on GitHub, using the notes extracted from the changelog.
    - Command: `gh release create vX.Y.Z --title "vX.Y.Z - Description" --notes "content from changelog" --prerelease`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/m1ckyb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/m1ckyb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
