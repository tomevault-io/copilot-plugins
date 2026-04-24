## dashboard-fraumueller-ch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a hybrid Laravel 11 + Vue.js application for building voting/rating in Zurich (gutebauten2025.stadt-zuerich.ch). The architecture combines traditional Blade templates for the public site with a Vue.js SPA for the admin dashboard.

## Essential Commands

### Development Setup
```bash
# Backend setup
composer install
php artisan migrate
php artisan db:seed
php artisan serve

# Frontend setup  
npm install
npm run dev          # Vite development server
npm run build        # Production build
```

### Testing
```bash
php artisan test     # Run all tests
./vendor/bin/pest    # Alternative test runner
```

### Code Quality
```bash
./vendor/bin/pint    # Laravel Pint code formatting
```

## Architecture

### Backend (Laravel 11)
- **Actions/**: Core business logic (Building, Comment, Vote operations)
- **Models/**: Eloquent models (Building, Comment, User, Voter)  
- **Http/Controllers/**: Request handling
- **Http/Resources/**: API resource transformations

### Frontend Structure
- **resources/js/spa/**: Vue.js SPA for admin dashboard
- **resources/js/interaction/**: Vue components for public voting interface
- **resources/js/modules/**: Vanilla JS modules
- **resources/views/**: Blade templates for public site

### Key Technologies
- **Backend**: Laravel 11, PHP 8.2, MySQL, Laravel Sanctum, Livewire 3.5, Pest testing
- **Frontend**: Vue.js 3, Vue Router 4, Pinia 2.3, Tailwind CSS 3.1, Alpine.js 3.13, Vite 4.5
- **Maps**: Leaflet with Swiss coordinate system (Proj4)
- **Voting**: FingerprintJS for voter tracking

### Database Structure
Core entities: Buildings (slug, coordinates, year), Comments (with publishing workflow), Voters (fingerprint tracking), Users (admin access).

### Build Tools
Uses Vite as primary build tool with Vue.js support. Legacy Laravel Mix commands still available for compatibility.

### Custom Design System
Tailwind CSS with custom color palette (limora, crimora, mintara, steelor) and granular spacing scale (1-250).

### Multilingual Support
German Swiss (de_CH) and English localization with Laravel's translation system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marceli-to) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
