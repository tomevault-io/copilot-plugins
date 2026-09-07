## distributorforklift

> Custom fullstack website for forklift distributor company.

# DISTRIBUTOR FORKLIFT CMS PROJECT

## Project Overview

Custom fullstack website for forklift distributor company.

This project is NOT an ecommerce platform.
Products are catalog/showcase only.

Customer inquiries are handled through:

- WhatsApp
- Email
- Contact Forms

This project focuses on:

- premium industrial visual design
- maintainable CMS architecture
- fast performance
- easy long-term maintenance
- scalable content management

---

# Core Technology Stack

## Backend

- Laravel (latest stable)
- PHP 8.3+

## Frontend

- Vue 3
- Inertia.js
- Tailwind CSS
- Vite

## Database

- MySQL / MariaDB
- DO NOT use SQLite for production

## Deployment Style

- Monolith Architecture
- Single Laravel App
- Frontend and Backend MUST NOT be separated

Reason:

- Easier maintenance
- Faster development
- Better CMS integration
- Easier deployment
- Better suited for medium-scale company profile CMS

---

# Design Reference System

## Primary Design Reference

The main visual reference for this project is:

https://www.still.co.uk/

This project should visually emulate the design language of STILL UK.

Important:
This is NOT a literal 1:1 clone.
The goal is to replicate:

- visual quality
- layout feel
- industrial premium atmosphere
- spacing
- typography hierarchy
- clean enterprise presentation

while adapting:

- company branding
- content structure
- simplified business requirements

---

# Visual Characteristics

## Design Style

Target visual identity:

- industrial premium
- enterprise-grade
- modern
- spacious
- minimal but strong
- machinery/heavy equipment oriented

---

# Layout Characteristics

Preferred layout characteristics:

- large whitespace
- wide containers
- large typography
- clean section separation
- oversized banners
- strong CTA hierarchy
- minimal visual clutter

---

# Typography Direction

Typography should feel:

- modern
- bold
- clean
- industrial
- professional

Recommended font style:

- sans-serif
- strong heading hierarchy
- medium-to-large text sizing

Potential font choices:

- Inter
- Montserrat
- Oswald (for accents/headings only)

---

# Color Direction

Primary visual palette:

- white backgrounds
- dark/slate typography
- industrial orange accents
- subtle gray sections

Avoid:

- excessive gradients
- flashy neon colors
- crowded UI
- overly playful design

---

# Animation Philosophy

Animations should be:

- subtle
- smooth
- professional
- lightweight

Recommended:

- fade transitions
- hover elevation
- image zoom hover
- smooth scrolling
- subtle card movement

Avoid:

- excessive motion
- overly flashy animations
- heavy JS animation libraries

---

# Product Presentation Philosophy

Products should feel:

- premium
- industrial
- high-value
- enterprise-grade

Product cards should emphasize:

- image quality
- clean spacing
- specifications readability
- strong CTA buttons

---

# Responsive Philosophy

Mobile responsiveness is mandatory.

Layouts should remain:

- spacious
- readable
- premium-looking

Avoid:

- cramped mobile layouts
- oversized text on small screens
- broken spacing

---

# Frontend Engineering Philosophy

Frontend should prioritize:

- clean component structure
- reusable sections
- maintainable Tailwind classes
- minimal CSS duplication

Use:

- reusable Vue components
- Tailwind utility classes
- consistent spacing scale

---

# Important Design Notes

This project references STILL UK for:

- design inspiration
- UX direction
- layout hierarchy

But should NOT:

- copy copyrighted assets
- duplicate company branding
- duplicate proprietary content
- replicate exact business structure

The implementation should remain:

- custom-built
- maintainable
- brand-adjusted
- client-specific

---

# Development Environment

## Required Versions

### Node.js

Use:

- Node 22 LTS

DO NOT use old Node versions.

### PHP

Use:

- PHP 8.3+

### Database

Use:

- MySQL 8+
  OR
- MariaDB

---

# Project Architecture

## Public Pages

Public-facing pages include:

- Home
- Products
- Product Detail
- About
- Contact
- News
- Dynamic Static Pages

---

## Admin Pages

Admin pages include:

- Dashboard
- Products CRUD
- Dynamic Pages CRUD
- News CRUD
- Hero Banner Management
- Website Settings
- Inquiry Management (optional future feature)

---

# Important Architectural Decisions

## 1. Monolith Architecture

This project intentionally uses monolith architecture.

DO NOT:

- Separate frontend/backend into different repositories
- Create unnecessary REST API architecture
- Create microservices

Reason:

- Project scope is medium-scale
- CMS-heavy architecture
- Faster development
- Easier maintenance
- Easier deployment

Use:

- Laravel + Inertia.js

---

## 2. Authentication Rules

Authentication is ADMIN ONLY.

Important:

- Public registration MUST be disabled in production
- Login page is only for internal admin use
- No customer authentication system needed

Future production rules:

- Remove register route
- Keep login route only
- Optional: disable forgot password

---

## 3. CMS Philosophy

The client must be able to manage content independently.

Everything possible should be dynamic through admin dashboard.

DO NOT hardcode:

- hero texts
- hero images
- contact information
- WhatsApp numbers
- email addresses
- static page contents
- SEO metadata

---

# Dynamic Page System

## Dynamic Static Pages

Client requested mini CMS system.

Admin must be able to:

- create pages
- edit pages
- delete pages

Each page contains:

- title
- slug
- description/content
- featured image
- SEO metadata (optional future)
- publish status

Example:

- /tentang-kami
- /layanan
- /karir

Pages should be database-driven.

DO NOT create static Blade/Vue pages manually for each content page.

---

# Product System

## Product Catalog Only

Products are NOT ecommerce products.

There is:

- NO checkout
- NO cart
- NO payment gateway

Products are showcase/catalog only.

Each product includes:

- title
- slug
- short description
- full description
- specifications
- category
- featured image
- gallery images (optional future)

---

# Contact & Inquiry System

All CTA buttons should direct users to:

- WhatsApp
- Email

Forms may:

- send email
- redirect/send formatted WhatsApp message

No complex CRM needed.

---

# Frontend Philosophy

## Visual Direction

Design inspiration:

- STILL.de

Visual identity:

- premium industrial
- clean
- spacious
- modern
- high-end company profile

Main characteristics:

- strong typography
- large whitespace
- industrial orange accents
- elegant animations
- responsive layouts

---

# Tailwind Guidelines

Use:

- Tailwind utility-first approach
- reusable Vue components

Avoid:

- large custom CSS files
- inline duplicated styles

Preferred colors:

- slate
- zinc
- orange

---

# Vue Architecture

## Recommended Structure

resources/js/

- Pages/
- Components/
- Layouts/
- Composables/
- Utils/

---

# Recommended Components

Reusable components:

- Navbar
- Footer
- HeroSection
- ProductCard
- CTASection
- NewsCard
- PageBanner

---

# Backend Guidelines

## Controllers

Keep controllers thin.

Business logic should be separated into:

- Services
- Actions
- Helpers (if needed)

---

## Validation

Always use:

- Form Request Validation

Avoid:

- inline validation inside controllers

---

# Image Handling

Use:

- Intervention Image

Images should:

- resize automatically
- optimize automatically

Reason:

- better performance
- lower bandwidth
- faster loading

---

# Slug System

Use slugs for:

- products
- pages
- news

Recommended package:

- cviebrock/eloquent-sluggable

---

# Settings System

Create centralized settings system.

Recommended table:

- settings

Store:

- company name
- address
- WhatsApp
- email
- social links
- logos
- SEO defaults

DO NOT hardcode these values.

---

# Performance Philosophy

Target:

- fast loading
- lightweight frontend
- optimized images
- minimal JS bloat

Server Specs:

- 4 Core CPU
- 16GB RAM

This project is lightweight relative to server specs.

Performance should remain excellent.

---

# Deployment Notes

## Production Build

Use:
npm run build

DO NOT use:
npm run dev in production

---

## Web Server

Recommended:

- Nginx

Alternative:

- Apache

---

## Recommended Production Stack

- Ubuntu Server
- Nginx
- PHP-FPM
- MySQL
- Supervisor (optional future)
- SSL enabled

---

# Email System

Use third-party email provider.

Recommended:

- Zoho Mail
- Google Workspace

DO NOT self-host email server.

Responsibilities:

- Configure MX
- Configure SPF
- Configure DKIM
- Configure DMARC

Goal:

- proper inbox delivery
- avoid spam classification

---

# Long-Term Maintenance Philosophy

This project should remain:

- easy to update
- easy to deploy
- easy to scale
- easy to maintain

Avoid:

- overengineering
- unnecessary abstractions
- unnecessary packages
- overly complex architecture

Priority:

- maintainability
- readability
- reliability
- performance

---

# Current Scope Summary

Included:

- company profile
- product catalog
- admin dashboard
- dynamic static pages CMS
- news system
- server deployment
- mailbox integration

Excluded:

- ecommerce
- payment gateway
- customer login
- multi-vendor system
- ERP integration

---

# Development Priority Order

Recommended development order:

1. Settings System
2. Layout System
3. Authentication Cleanup
4. Products CRUD
5. Dynamic Pages CRUD
6. News CRUD
7. Frontend Slicing
8. Inquiry Forms
9. SEO Improvements
10. Deployment

---

# Notes For AI Assistants

When generating code:

- prioritize maintainability
- avoid overengineering
- prefer reusable components
- prefer clean Laravel conventions
- prefer scalable CMS patterns

Do not generate:

- unnecessary API architecture
- complex enterprise patterns
- microservices
- unnecessary state management libraries

This project is a maintainable monolith CMS.

---
> Source: [deannasution21/distributorforklift](https://github.com/deannasution21/distributorforklift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
