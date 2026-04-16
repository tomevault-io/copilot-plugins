## trustloan-capstone-project

> TrustLoan MVC and tech stack. Apply when editing or generating PHP, JS, or CSS.


# TrustLoan – MVC & Tech Stack (Mandatory)

- **Stack (consistent only):** Backend = plain PHP (no Laravel/Symfony). Frontend = HTML + plain CSS + vanilla JavaScript (no React, Vue, Angular, jQuery). JS libraries via CDN only (e.g. intl-tel-input, SweetAlert2). Do not introduce other frameworks or languages.
- **Redirects:** Use a helper that always includes `?page=...` so we never redirect to bare index.php.
- **Password:** Validation and hashing in js/password.js; server stores bcrypt(hash).

## Naming and structure (use going forward)

### Actions (`Actions/`)
- **Naming:** `{purpose}_action.php` (e.g. `register_customer_action.php`).
- **Role:** Receives data from the form/request, invokes the relevant method on the controller, and returns a message or redirect to the caller. No business logic; thin entry point only.

### Classes / Models (`Classes/` or `Models/`)
- **Naming:** `{entity}_class.php` (e.g. `customer_class.php`).
- **Role:** A class that extends the database connection and contains all entity methods: add, edit, delete, get by id, etc. All business logic and data access live here. No HTML, no HTTP.

### Controllers (`Controllers/`)
- **Naming:** `{entity}_controller.php` (e.g. `customer_controller.php`).
- **Role:** Creates an instance of the class and runs its methods. Has one method per action, e.g. `register_customer_ctr($kwargs)` which invokes `customer_class::add($args)`. Handles HTTP/request coordination only; no SQL, no HTML.

### Views
- Presentation only. No SQL, no financial calculations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicoletc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
