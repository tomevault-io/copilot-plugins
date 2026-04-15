## medisola-ona

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Godo Shopping Mall** aggregated project for Medisola, a Korean e-commerce platform focused on health food and nutrition. This repository contains **4 separate repositories** that work together to form a complete e-commerce platform:

1. **godo-dev-module/**: Development environment with complete PHP framework
2. **godo-module/**: Production environment with custom extensions only
3. **godo-skin/**: Frontend themes and skins for desktop and mobile interfaces
4. **admin-skin/**: Administrative interface customizations

## Repository Architecture

### Multi-Repository Structure

The project follows a distributed architecture with 4 interconnected repositories:

```
godo-dev-module/         # Development Environment (2,062 PHP files)
├── Bundle/              # Core GodomMall framework (READ-ONLY)
├── Component/           # Custom business logic extensions
├── Controller/          # MVC Controllers (Admin/Front/Mobile)
└── Widget/              # UI Widget components

godo-module/             # Production Environment (147 PHP files)
├── Component/           # Custom extensions only (inherits from platform Bundle)
├── Controller/          # Production controllers
└── Widget/              # Production widgets

godo-skin/              # Frontend Themes (1,984 HTML files)
├── front/              # Desktop themes (drorganic_24_renewal, medisola_dev)
├── mobile/             # Mobile themes (dorganic_24_renewal, medisola_dev)
└── backup/             # Theme version backups

admin-skin/             # Admin Interface (851 PHP files)
└── medisola_admin_skin_original/  # Live admin panel customizations
```

### Repository Relationships

**Development Workflow:**

```
Development: godo-dev-module → Testing → Production: godo-module
                    ↓
                godo-skin (themes) + admin-skin (admin UI)
```

**Key Differences Between Modules:**

-   **godo-dev-module**: Complete framework with Bundle core for development
-   **godo-module**: Production-ready extensions only, inherits from platform Bundle
-   **Shared Components**: Both use identical custom business logic with same inheritance patterns

### Technology Stack

-   **Backend**: PHP with CakePHP-style architecture
-   **Frontend**: HTML, CSS, JavaScript (jQuery-based)
-   **Platform**: Godo Shopping Mall (Korean e-commerce platform)
-   **Database**: MySQL with Korean language support
-   **Payment**: Korean payment gateways (Naver Pay, KakaoPay, etc.)

## Development Workflow

### Development Workflow Process

**⚠️ IMPORTANT: Follow this exact development sequence:**

**Step 1: Frontend Development (Development Themes)**

1. Develop new features in `godo-skin/front/medisola_dev/` (desktop)
2. Develop new features in `godo-skin/mobile/medisola_dev/` (mobile)
3. Test on development site using medisola_dev skin

**Step 2: Backend Development (Development Module)**

1. Develop new features in `godo-dev-module/`
2. Test backend functionality on development site

**Step 3: Production Distribution (After All Features Confirmed)**

1. Apply frontend changes to `godo-skin/front/drorganic_24_renewal/`
2. Apply mobile changes to `godo-skin/mobile/dorganic_24_renewal/`
3. Apply backend changes to `godo-module/`

**🚨 DO NOT modify production folders during development:**

-   `godo-skin/front/drorganic_24_renewal/` (production frontend)
-   `godo-skin/mobile/dorganic_24_renewal/` (production mobile)
-   `godo-module/` (production backend)

### Repository-Specific Deployment

**godo-dev-module:**

-   Direct SFTP deployment to `gdadmin.medisola2.godomall.com` on save
-   VSCode with SFTP extension configured
-   Primary development environment

**godo-skin (medisola_dev themes):**

-   Development themes for testing new features
-   Live deployment to development site
-   Template syntax testing environment

**godo-module & drorganic_24_renewal (dorganic_24_renewal) themes:**

-   Production deployment only after feature confirmation
-   Final distribution target after development completion

**admin-skin:**

-   Direct FTP deployment (live environment)
-   Real-time admin interface updates
-   Korean administrative interface

### No Build Process Required

-   **Direct Deployment**: Files are deployed directly via SFTP/FTP on save
-   **No Dependencies**: No composer, npm, or package managers
-   **No Compilation**: No build steps needed
-   **Live Development**: Changes reflect immediately in development environment

### Key Development Rules

#### **MANDATORY DUAL IMPLEMENTATION RULE**

**⚠️ ALWAYS implement features in BOTH desktop AND mobile versions during development:**

**Development Phase:**

1. **Desktop Development**: `/godo-skin/front/medisola_dev/`
2. **Mobile Development**: `/godo-skin/mobile/medisola_dev/`

**Production Distribution (After confirmation):**

1. **Desktop Production**: `/godo-skin/front/drorganic_24_renewal/`
2. **Mobile Production**: `/godo-skin/mobile/dorganic_24_renewal/`

When developing ANY feature:

-   ✅ **HTML templates**: Implement in both `medisola_dev` desktop and mobile first
-   ✅ **JavaScript logic**: Sync both `medisola_dev` front and mobile JS files
-   ✅ **CSS styling**: Apply to both `medisola_dev` desktop and mobile stylesheets
-   ✅ **Template variables**: Ensure data consistency across development themes
-   ✅ **Backend logic**: Develop in `godo-dev-module` first

#### **Development-to-Production Flow**

**⚠️ Follow this exact sequence:**

**Development Phase (DO NOT touch production):**

1. **Frontend**: Work only in `medisola_dev` themes (front + mobile)
2. **Backend**: Work only in `godo-dev-module`
3. **Testing**: Verify functionality on development site

**Production Distribution (After all features confirmed):**

1. **Frontend(Front)**: Copy confirmed changes from `medisola_dev` → `drorganic_24_renewal`
1. **Frontend(Mobile)**: Copy confirmed changes from `medisola_dev` → `dorganic_24_renewal`
1. **Backend**: Copy confirmed changes from `godo-dev-module` → `godo-module`

#### **File Synchronization Patterns**

**Development Phase:**

-   **Frontend**: `godo-skin/front/medisola_dev/` ↔ `godo-skin/mobile/medisola_dev/`
-   **Backend**: `godo-dev-module/Component/` (development only)

**Production Distribution:**

-   **Frontend(Front)**: `medisola_dev` themes → `drorganic_24_renewal` themes
-   **Frontend(Mobile)**: `medisola_dev` themes → `dorganic_24_renewal` themes
-   **Backend**: `godo-dev-module/Component/` → `godo-module/Component/`

### Core Modules

**E-commerce Core (All Repositories)**

-   `Order/` - Order processing and management
-   `Cart/` - Shopping cart functionality
-   `Goods/` - Product catalog and management
-   `Member/` - User accounts and authentication

**Medisola-Specific Features**

-   `GiftOrder/` - Gift card and gift ordering system
-   `Wm/EarlyDelivery` - Express delivery services
-   `Wm/FirstDelivery` - First-time delivery logic
-   `OurMenu/` - Custom menu management
-   `SelfCancel/` - Self-cancellation functionality

**Integration Services**

-   `Sms/` - SMS notifications and marketing
-   `Mail/` - Email systems
-   `Marketing/` - Third-party integrations (Kakao, Naver, Apple Login)

**Repository-Specific Components**

**godo-dev-module & godo-module:**

-   Complete business logic implementations
-   Database access objects (DAO)
-   Validation classes
-   Trait-based shared functionality

**godo-skin:**

-   Frontend templates and themes
-   JavaScript UI interactions
-   CSS styling and responsive design
-   Analytics integration (Amplitude)

**admin-skin:**

-   Administrative interface components
-   Backend management tools
-   Live admin panel functionality

## Template System

### Godo Template Syntax

**Template Tags**

-   **Delimiters**: Use `{ }` for template tags
-   **Variables**: `{variable_name}` or `{=variable_name}`
-   **Loops**: `{@loop_id}...{/@}`
-   **Conditionals**: `{?expression}...{:}...{/?}`
-   **Includes**: `{#file_id}`

**Common Variables**

-   `{=gMall.mallNm}`: Mall name
-   `{=gSess.memId}`, `{=gSess.memNm}`: Member info
-   `{goods.*}`: Product data
-   `{order.*}`: Order data

### File Naming Conventions

-   Templates use `.html` extension but contain PHP-style template syntax
-   Partial templates prefixed with `_` (e.g., `_header.html`)
-   Godo-specific JavaScript files prefixed with `gd_`
-   Layer/modal popups prefixed with `layer_`

## Key Features

### Korean E-commerce Specific

-   **Product Categories**: 새벽배송 (dawn delivery), 샐러드 (salads), 건강케어 (health care), 질환케어 (disease care)
-   **Payment Systems**: Integration with Korean payment gateways (Naver Pay, KakaoPay, etc.)
-   **Delivery Options**: Scheduled delivery, express delivery, first-time delivery
-   **Membership System**: Tiered benefits and rewards with Korean language support

### Custom Extensions

-   **Gift Ordering**: Custom gift card selection and ordering (across all repositories)
-   **Differential Pricing**: Complex pricing rules based on quantity and menu types
-   **Review System**: Photo-based review system with rewards (Plus Review)
-   **SMS Marketing**: Automated SMS campaigns and notifications
-   **Early Delivery**: Express delivery services for health food products
-   **Self-Cancellation**: Customer-initiated order cancellation system

### Multi-Repository Integration

-   **Cross-Repository Data Flow**: Orders created in frontend (godo-skin) → processed in backend (godo-dev-module/godo-module) → managed in admin (admin-skin)
-   **Shared Database Schema**: Custom tables with `wm_` prefix used across all repositories
-   **Consistent API**: Same controller and component interfaces across development and production modules

## Database Conventions

-   Tables use Korean comments for business logic
-   Custom tables prefixed with `wm_` (e.g., `wm_giftSet`)
-   Extends standard GodomMall database schema

## Development Guidelines

### Code Organization

**Repository-Specific Guidelines:**

**godo-dev-module:**

-   **Bundle directory**: Core components - DO NOT MODIFY (read-only framework)
-   **Component directory**: Custom business logic extensions
-   **Controller directory**: MVC controllers for different interfaces
-   **Widget directory**: Reusable UI components

**godo-module:**

-   **Component directory**: Production-ready custom extensions only
-   **Controller directory**: Production controllers (inherit from platform Bundle)
-   **Widget directory**: Production widgets

**godo-skin:**

-   **front/medisola_dev/**: Development desktop theme (work here first)
-   **mobile/medisola_dev/**: Development mobile theme (work here first)
-   **front/drorganic_24_renewal/**: Production desktop theme (distribute here after confirmation)
-   **mobile/dorganic_24_renewal/**: Production mobile theme (distribute here after confirmation)
-   **backup/**: Theme version control and backups

**admin-skin:**

-   **medisola_admin_skin_original/**: Live admin interface (direct FTP deployment)

### Inheritance Pattern

Custom components extend Bundle classes (different sources by repository):

**godo-dev-module:**

```php
class Order extends \Bundle\Component\Order\Order
// Extends from local Bundle directory
```

**godo-module:**

```php
class Order extends \Bundle\Component\Order\Order
// Extends from platform Bundle (not in repository)
```

### Namespace Structure

Components organized by functionality (consistent across repositories):

```php
namespace Component\GiftOrder;
namespace Controller\Admin\Goods;
namespace Widget\Front\FirstDelivery;
```

### Common Traits

Reusable functionality via traits (shared across development and production):

```php
use Component\Traits\SendSms;
use Component\Traits\GoodsInfo;
use Component\Traits\Common;
use Component\Traits\Security;
```

### Repository Integration Patterns

**Development to Production Flow:**

1. Develop in `godo-dev-module` with complete Bundle framework
2. Test with `godo-skin` themes (desktop and mobile)
3. Deploy custom components to `godo-module` (production)
4. Update `admin-skin` for administrative features

**File Synchronization:**

-   Backend logic files should be nearly identical between `godo-dev-module` and `godo-module`
-   Frontend templates must be synchronized between desktop and mobile in `godo-skin`
-   Admin interface updates in `admin-skin` require careful testing due to live deployment

## Architecture Documentation

For detailed architecture documentation, refer to: http://doc.godomall5.godomall.com/Getting_Started/Architecture

## Critical Development Notes

### Repository-Specific Warnings

**godo-dev-module:**

-   Contains complete development framework
-   Auto-deploys to development server on save
-   Safe for experimentation and testing

**godo-module:**

-   Production environment with selective deployment
-   Contains only custom extensions
-   Requires careful testing before deployment

**godo-skin:**

-   Multiple theme variations available
-   **MANDATORY**: Implement changes in both desktop and mobile versions
-   Backup system available for version control

**admin-skin:**

-   **LIVE ENVIRONMENT**: Direct FTP deployment
-   Changes reflect immediately in production admin interface
-   Requires extra caution and testing

### Inter-Repository Dependencies

-   **Database Schema**: Shared across all repositories with `wm_` prefix for custom tables
-   **Template Variables**: Must be consistent between frontend themes and backend data
-   **API Interfaces**: Controllers must maintain compatibility across development and production modules
-   **Korean Language Support**: Consistent across all repositories for proper localization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kino-ona)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kino-ona)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
