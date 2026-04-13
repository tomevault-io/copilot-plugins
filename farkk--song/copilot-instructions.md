## song

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SONGLY is a PHP-based song lyrics website that displays information about singers, songs, albums, and song lyrics. The application uses a MySQL database and serves dynamic content through PHP pages.

## Database Connection

All PHP files use centralized database configuration from `config/db_config.php`:
- Host: `songbooks.asmart-test-dev.ru`
- Username: `root`
- Password: (empty)
- Database: `j27119254_song`

The connection is automatically established and closed using the config file. All pages use `require_once('config/db_config.php')` to connect.

## Database Schema

The application uses five main tables:

### Core Tables
- `singer`: Singer information (id_singer, name_singer, followers, contributions, description, image_path)
- `songs`: Song information (id_songs, name_songs, id_singer, number_of_auditions, image_path)
- `albums`: Album information (id_albums, name_albums, id_singer, year_of_release, image_path)
- `textsong`: Song lyrics (id_songs, producer, textsong)

### Admin Table
- `admin_users`: Admin authentication (id, username, password_hash, created_at)

**Note**: Image paths are now stored in the database as relative paths (e.g., `images/singer/1.png`) instead of being constructed from IDs.

## Core Pages and Architecture

### Main Entry Point
- `index.php`: Homepage displaying news and top charts
  - Dynamically loads songs from database sorted by number_of_auditions DESC
  - Shows all songs with Load More/Hide functionality (first 3 visible by default)
  - Formats view counts (M for millions, k for thousands)
  - Uses database image paths with fallback to default cover

### Dynamic Pages
- `singer.php?id={id}`: Singer detail page showing:
  - Singer information and description
  - Top 4 most popular songs (by number_of_auditions)
  - 4 most recent albums (by year_of_release)

- `text_song.php?id={id}`: Song lyrics page displaying:
  - Song cover image
  - Song name, artist, producer
  - Full lyrics text
  - Joins `songs` and `textsong` tables to get complete information

- `singers_list.php`: List of all singers from database
- `songs_list.php`: List of all songs from database

### Image Organization

Images are stored in the filesystem with paths saved in the database:
- Singer images: `images/singer/` directory
- Song covers: `images/song/` directory
- Album covers: `images/album/` directory
- Default fallback images exist for missing covers

Image paths are stored in database columns (`image_path`) as relative paths. The admin panel handles image uploads and updates the database accordingly.

## Admin Panel Architecture

### Location
All admin files are in the `/admin/` directory.

### Authentication System
- `admin_login.php`: Login page with session-based authentication
- `admin_logout.php`: Logout handler
- `auth_check.php`: Authentication middleware (included at the top of all admin pages)
- Session timeout: 30 minutes of inactivity
- Passwords hashed using `password_hash()` with PASSWORD_DEFAULT

### Admin Pages Structure
Each entity (singers, songs, albums) has a consistent CRUD pattern:
- `admin_{entity}.php`: List/view all records
- `admin_{entity}_add.php`: Add new record
- `admin_{entity}_edit.php?id={id}`: Edit existing record
- `admin_{entity}_delete.php?id={id}`: Delete record (with validation)

**Lyrics Management**: Special case - `admin_lyrics.php` and `admin_lyrics_edit.php` manage the `textsong` table.

### Image Upload Handler
- Location: `admin/includes/upload_image.php`
- Functions: `uploadImage()`, `deleteImage()`, `imageExists()`
- Validates file type (JPG, PNG, GIF), size (max 5MB)
- Returns associative array with success status and relative path
- Used by all entity add/edit pages

### Bootstrap Integration
All admin pages use Bootstrap 5.3.0 via CDN for UI components, with custom styles in `admin/css/admin_style.css`.

## Development Commands

This is a simple PHP project with no build system. To run locally:

```bash
# Start PHP built-in server
php -S localhost:8000

# Or use MAMP/XAMPP and navigate to the project directory
```

### Accessing the Admin Panel

After running the SQL setup (`database_full_setup.sql`):
- Admin URL: `https://songbooks.asmart-test-dev.ru/admin/admin_login.php`
- Default credentials: `admin` / `admin123`
- Change the default password after first login!

**Note:** Database `j27119254_song` must exist before running the SQL script.

## Security Notes

All PHP files use prepared statements with parameterized queries to prevent SQL injection. Input validation checks that IDs are numeric before querying the database.

### Admin Panel Security Features
- Password hashing with `password_hash()` and `password_verify()`
- Session-based authentication with 30-minute timeout
- XSS protection via `htmlspecialchars()` on all user input
- File upload validation (type, size, MIME type)
- CSRF protection considerations for production deployment

## Common Modification Patterns

### Adding Content (Recommended Method)
Use the admin panel at `/admin/` to:
1. Add singers, songs, albums, and lyrics through web interface
2. Upload images via the admin forms (automatic validation and path management)
3. All database operations handled with proper validation

### Direct Database Modification
If adding content directly to the database:
1. Insert records into the appropriate table
2. Manually upload image to correct directory (`images/singer/`, `images/song/`, `images/album/`)
3. Update the `image_path` column with the relative path (e.g., `images/singer/1.png`)
4. Frontend will automatically display new content

### Modifying Database Queries
- All queries use prepared statements (`$conn->prepare()`)
- Remember to check both success path and error handling
- Close statements after use (`$stmt->close()`)
- When adding new queries, include `image_path` in SELECT statements for images

### Adding New Admin Users
Execute SQL to add a new admin (hash password first with `password_hash()`):
```sql
INSERT INTO admin_users (username, password_hash) VALUES ('newuser', 'hashed_password');
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Farkk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Farkk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
