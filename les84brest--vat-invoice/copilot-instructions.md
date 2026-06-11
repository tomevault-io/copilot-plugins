## vat-invoice

> This project is a web application designed to train students in creating VAT invoices.

# Project Context

## Overview

This project is a web application designed to train students in creating VAT invoices.

The system simulates real-world invoice workflows and enforces correct structure and validation of VAT-related documents.

---

## Application Architecture

The application is divided into two main parts:

### 1. Admin Panel

* is available by `/admin` route only
* is used for managing all system (users, settings,  handbooks)

### 2. User Workspace

* is available from the root `/` route
* this is main working zone for application users
* functionality:

  * invoices creation
  * invoices editing
  * view invoices
  * invoices submitting and canceling

---

## Tech Stack

### Backend

* PHP 8.3
* Laravel 11
* Laravel Sanctum (authentication)

### Frontend

#### Admin Panel

* Blade templates
* AdminLTE (Bootstrap-based admin template)
* Bootstrap components
* DataTables (table management)

#### Client Application

* Vue 3 (SPA) composition API
* Pinia (state management)
* Vue Router 4
* Element Plus (UI library)

#### Landing / Main Page

* TailwindCSS

### Build Tools

* Vite

---

## Backend Architecture Rules

* Controllers must be thin
* Business logic must be placed in Service classes
* Use Repository pattern for data access abstraction (when complexity increases)
* Use FormRequest classes for validation
* Use DTOs for structured data transfer when needed

### Dependency Rules

* Use Dependency Injection everywhere
* Avoid using Facades inside Services
* Prefer constructor injection

---

## Code Style Rules

* Strict typing required
* All methods must have return types
* Follow PSR-12
* Use meaningful naming (no abbreviations)
* Keep methods small and focused

---

## Frontend Rules

### Vue Application

* Use Composition API
* Use Pinia for global state only
* Keep components small and reusable
* Business logic should not be inside UI components (move to composables/services)

### UI

* Use Element Plus components consistently
* Do not mix multiple UI libraries inside SPA

---

## Admin Panel Rules

* Blade is used only in admin panel
* jQuery plugins (like DataTables) are allowed ONLY in admin
* Do not introduce Vue into admin unless explicitly required

---

## Routing Rules

* `/admin/*` → admin panel (Blade)
* `/` → Vue SPA
* API endpoints should be versioned if extended (`/api/v1/...`)

---

## Testing

* PHPUnit is used
* Feature tests for endpoints
* Unit tests for Services

---

## When Generating Code

ALWAYS:

* Create Service layer for business logic
* Use FormRequest for validation
* Return JSON responses for API
* Follow existing architecture

NEVER:

* Put business logic in Controllers
* Mix Blade and Vue in the same feature
* Use raw SQL unless necessary
* Modify unrelated files

---

## Domain Notes

* The system is focused on VAT invoices
* Accuracy and validation of financial data is critical
* Data consistency is more important than performance optimizations

---

## Agent Behavior Guidelines

* Prefer small, incremental changes
* Ask before making large refactors
* Preserve backward compatibility
* Follow existing patterns in the codebase

---

---
> Source: [Les84Brest/vat-invoice](https://github.com/Les84Brest/vat-invoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
