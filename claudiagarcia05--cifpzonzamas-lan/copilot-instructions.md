## cifpzonzamas-lan

> This project is a web-based Library Management System built with the **Laravel 12** framework. It is designed to manage books ("Libros") and users ("Usuarios"), providing functionalities for cataloging, searching, and administrative tasks.

# CIFP Zonzamas Library Management System

## Project Overview

This project is a web-based Library Management System built with the **Laravel 12** framework. It is designed to manage books ("Libros") and users ("Usuarios"), providing functionalities for cataloging, searching, and administrative tasks.

**Key Technologies:**

*   **Framework:** Laravel 12.x (PHP ^8.2)
*   **Frontend:** Blade Templates, Bootstrap 5.3.2, Vite, Alpine.js
*   **Admin Panel:** AdminLTE 4 (rc6)
*   **Authentication:** Laravel Breeze
*   **Permissions:** Spatie Laravel Permission

## Architecture

*   **MVC Pattern:** Follows the standard Model-View-Controller architecture of Laravel.
*   **Database:**
    *   `libros`: Stores book information (Title, Author, Year, Genre, Description).
    *   `usuarios`: Stores basic user information (Name, Email).
    *   `users`: Standard Laravel authentication table.
*   **Routing:** Defined in `routes/web.php`. Includes standard auth routes and specific resource routes for `LibroController`.

## Building and Running

### Prerequisites

*   PHP >= 8.2
*   Composer
*   Node.js & NPM

### Setup Command

The project includes a convenient setup script in `composer.json`:

```bash
composer run setup
```

This command performs the following:
1.  Installs PHP dependencies (`composer install`).
2.  Copies `.env.example` to `.env`.
3.  Generates the application key.
4.  Runs database migrations.
5.  Installs and builds frontend assets (`npm install && npm run build`).

### Development Server

To start the development server with all necessary services (server, queue, logs, vite):

```bash
composer run dev
```

Alternatively, you can run them separately:

*   **Backend:** `php artisan serve`
*   **Frontend (Vite):** `npm run dev`

### Running Tests

```bash
composer run test
```

## Key Directories & Files

*   `app/Http/Controllers/LibroController.php`: Manages Book CRUD operations. Note: It uses a combined approach for handling GET/POST requests and reuses the `libros.create` view for create, edit, show, and destroy actions.
*   `database/migrations/`: Contains database schema definitions, including `create_table_libros.php` and `create_usuarios_table.php`.
*   `resources/views/libros/`: Contains Blade templates for book management.
*   `routes/web.php`: Application entry points and route definitions.

## Development Conventions

*   **Code Style:** Follows standard PSR-12 coding standards.
*   **Frontend:** Uses Bootstrap 5.3.2. Assets are compiled using Vite.
*   **Controller Logic:** `LibroController` handles form submissions within the same method as the form display (checking `Request::isMethod('post')`).


#Estructura de urls
Crea siempre estructuras tipo amigables, por ejemplo usuarios.php?operacion=edicion&id=2 crealo como
/usuarios/edicion/2

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Claudiagarcia05)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Claudiagarcia05)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
