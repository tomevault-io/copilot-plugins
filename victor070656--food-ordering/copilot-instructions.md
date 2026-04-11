## food-ordering

> This project is a home-cooked meal delivery management system called "FoodSys". It is a PHP-based web application that allows customers to order food online, and for the business to manage orders, customers, and riders.

# Project Overview

This project is a home-cooked meal delivery management system called "FoodSys". It is a PHP-based web application that allows customers to order food online, and for the business to manage orders, customers, and riders.

The application is built with a classic PHP structure, using a combination of procedural and object-oriented programming. It uses a MySQL database to store data, and the front-end is built with Tailwind CSS and JavaScript.

## Key Features

*   **Customer-facing website:** A public-facing website where customers can browse the menu, add items to their cart, and place orders.
*   **Admin dashboard:** A comprehensive dashboard for administrators to manage all aspects of the system, including users, customers, orders, menu items, and payments.
*   **Staff dashboard:** A dashboard for staff members to view and manage orders.
*   **Rider dashboard:** A dashboard for riders to view and manage their deliveries.
*   **API:** A simple API for placing orders.

# Building and Running

This is a PHP application, so there is no build process. To run the project, you will need a web server with PHP and MySQL installed (like XAMPP, WAMP, or MAMP).

1.  **Database Setup:**
    *   Create a new MySQL database named `foodsys`.
    *   Import the `database.sql` file to create the necessary tables and seed the database with initial data.

2.  **Configuration:**
    *   Open the `config/config.php` file and update the database credentials (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`) to match your local environment.

3.  **Running the Application:**
    *   Place the project files in the web root of your server (e.g., `htdocs` for XAMPP).
    *   Access the project in your web browser (e.g., `http://localhost/food-sys`).

# Development Conventions

*   **Database:** The application uses a `Database` class (in `classes/Database.php`) that implements the singleton pattern for database connections. It uses prepared statements to prevent SQL injection.
*   **Authentication:** The application has a `User` class (in `classes/User.php`) that handles user authentication. However, it is important to note that the application stores passwords in plain text, which is a major security vulnerability. This should be addressed by implementing a password hashing mechanism (e.g., `password_hash()` and `password_verify()`).
*   **Routing:** The application does not use a formal routing library. Instead, it relies on individual PHP files for each page.
*   **Styling:** The application uses Tailwind CSS for styling.
*   **API:** The application has a simple API for placing orders. The API endpoints are located in the `api/` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Victor070656)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Victor070656)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
