## real-cargofind

> This project is a logistics platform based on Flask, SQLAlchemy, and SocketIO.

# CargoFind - Zencoder Guidelines

This project is a logistics platform based on Flask, SQLAlchemy, and SocketIO.

## Commands

### Running Locally
- Run the app: `python app.py` (Defaults to port 5000)
- Install dependencies: `pip install -r requirements.txt`

### Deployment
- Procfile: `web: gunicorn --worker-class eventlet -w 1 app:app`
- Requirements: Requires `eventlet` and `gevent-websocket` for SocketIO.

### Database
- Migrations: Uses standard SQLAlchemy. Init via `db.create_all()` in `app.py`.
- Default Admin: Created automatically if not exists (admin@cargofind.com / admin123).

## Style Preferences

### Code Style
- **Naming**: Use `snake_case` for Python functions and variables.
- **Indentation**: 4 spaces for Python, standard indentation for HTML/CSS.
- **Templates**: Uses Jinja2. Follow the role-based folder structure in `templates/`.
- **Styling**: Uses Bootstrap 5 utility classes wherever possible. Custom styles in `static/css/`.
- **JavaScript**: Prefer modern ES6+ syntax. Socket.io and Leaflet.js are used for core features.

### Conventions
- Always check the `base.html` for global styles and navbar/footer before editing.
- Ensure role-based access control is maintained for `/admin`, `/customer`, and `/driver` routes.
- Flash messages are used for user feedback across the system.
- SocketIO events follow the `delivery_<id>` room naming convention.
- Icons: Use FontAwesome 6 (e.g., `fas fa-truck-moving`).

## Project Structure
- `app.py`: Main routes, socket events, and app initialization.
- `models.py`: Database models (User, Delivery, Notification, Wallet).
- `templates/`:
  - `admin/`: Admin-specific pages.
  - `customer/`: Customer-specific pages.
  - `driver/`: Driver-specific pages.
  - `base.html`: Main layout.
- `static/`:
  - `css/style.css`: Main custom styles.
  - `uploads/`: User-uploaded documents and images.

---
> Source: [anya-awazi/Real_CargoFind](https://github.com/anya-awazi/Real_CargoFind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
