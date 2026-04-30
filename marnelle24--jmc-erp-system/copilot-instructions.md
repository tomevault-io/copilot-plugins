## jmc-erp-system

> Laravel, Livewire, PHP, DB, naming, and security rules for JMC ERP


# Laravel & PHP

- PHP: PSR-12, typed properties.
- Laravel 13: Form Requests for validation, Policies for authorization, Eloquent relationships used correctly.
- Avoid N+1 queries; eager load as needed. Use database transactions for critical operations.
- Use DTOs where they clarify boundaries.

# Livewire

- Livewire 4 single-file components; keep UI logic minimal; delegate heavy work to services.

# Database

- Foreign keys and indexes where appropriate.

# Naming

- Tables: plural `snake_case`. Models: singular `PascalCase`. Services: action-oriented names.

# SaaS & security

- Validate all inputs. Never expose `.env` or secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marnelle24) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
