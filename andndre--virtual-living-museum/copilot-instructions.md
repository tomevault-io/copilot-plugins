## virtual-living-museum

> Virtual Living Museum is an interactive web platform designed for virtual heritage exploration through Augmented Reality (AR) and a gamified e-learning system. Built with Laravel 11 and PHP 8.2+.

# Virtual Living Museum

Virtual Living Museum is an interactive web platform designed for virtual heritage exploration through Augmented Reality (AR) and a gamified e-learning system. Built with Laravel 11 and PHP 8.2+.

## Project Overview

-   **Core Purpose:** Facilitate immersive heritage education and exploration.
-   **Main Technologies:**
    -   **Backend:** PHP 8.2+, Laravel 11.
    -   **Frontend:** Blade templates, Tailwind CSS, Alpine.js (for reactive state).
    -   **AR & 3D:** Three.js, AR.js, A-Frame, n8ao (AO), postprocessing.
    -   **Build System:** Vite.
    -   **Database:** MySQL with Indonesian naming conventions.
-   **Architecture:**
    -   Standard Laravel MVC.
    -   Business logic resides directly in Controllers.
    -   Three-tier access control: Guest (auth only), User (learning/AR), Admin (management).
    -   Token-based authentication for cross-device AR sessions (`ArTokenAuth`).

## Building and Running

### Prerequisites
- PHP 8.2 or higher
- Composer
- Node.js and NPM
- MySQL

### Installation & Setup
1.  **Clone and Install:**
    ```bash
    composer install
    npm install
    ```
2.  **Environment:**
    ```bash
    cp .env.example .env
    php artisan key:generate
    ```
3.  **Database:**
    ```bash
    php artisan migrate --seed
    ```
4.  **Development Server:**
    ```bash
    composer run dev
    ```
    *This runs `php artisan serve`, `queue:listen`, `pail`, and `npm run dev` concurrently.*

### Testing
-   Run tests using Pest:
    ```bash
    php artisan test
    ```

## Development Conventions

### PHP/Laravel
-   **Naming:** Use Indonesian for models, tables, and columns (e.g., `materi`, `situs_peninggalan`, `laporan`).
-   **Primary Keys:** Use explicit custom primary keys (e.g., `materi_id`, `situs_id`) instead of standard `id`.
-   **Type Safety:** Prefer type hints and return types in new code.
-   **Utilities:** Place shared logic in `app/Helper/` (e.g., `TokenHelper.php`).
-   **Relationships:** Always use `onDelete('cascade')` for foreign keys.

### Frontend
-   **State Management:** Use Alpine.js (`x-data`, `x-show`, etc.) for UI reactivity; avoid jQuery.
-   **Styling:** Utility-first CSS with Tailwind. Custom primary color is `#2D8BEB`.
-   **AR Implementation:**
    -   Marker-based AR uses AR.js + A-Frame.
    -   WebXR uses Three.js (r153).
    -   AR JavaScript modules are located in `public/assets/js/` and served directly (not bundled by Vite).

### Progressive Learning System
Users progress through defined stages:
1. `PRE_TEST` (1)
2. `EBOOK` (2)
3. `VIRTUAL_LIVING_MUSEUM` (3)
4. `POST_TEST` (4)
Logic for this is handled in `User.php` and `Materi.php`.

## Key Files & Directories
-   `app/Models/`: Eloquent models with Indonesian naming.
-   `app/Http/Controllers/`: Logic for Admin, Guest, and AR features.
-   `app/Helper/TokenHelper.php`: HMAC token generation for AR access.
-   `public/assets/js/`: Direct-served AR and 3D modules.
-   `resources/views/`: Blade components and pages.
-   `routes/web.php`: Categorized routing.
-   `.github/instructions/`: Detailed subsystem instructions (AR, Assessment, Migrations).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Andndre)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Andndre)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
