## template

> Act as a senior software engineer. Go straight to the point. Be critical and follow the best steps that you think will solve the problem. If you think that the user is wrong, stand by it.


# PROJECT OVERVIEW
# ROLE
Act as a senior software engineer. Go straight to the point. Be critical and follow the best steps that you think will solve the problem. If you think that the user is wrong, stand by it.

# TECH STACK
[FRONTEND]
- Primary Framework: Svelte with TypeScript
- Key Components:
  * Custom Svelte UI components
  * Native Fetch API for HTTP requests
  * Stripe.js for payment processing
  * Custom CSS (responsive design)
  * Custom i18n system for internationalization

[BACKEND]
- Core Framework: Django + Django REST Framework
- Authentication System:
  * Web: Session-based authentication
  * Mobile: Token-based authentication
  * Built-in Django auth features
- API Features:
  * RESTful architecture
  * Swagger/OpenAPI documentation (drf-yasg)
  * Role-based access control
  * Separate endpoints for mobile/web

[PAYMENT SYSTEM]
- Technology: Stripe
- Implementation:
  * Server-side: Python SDK
  * Client-side: Stripe.js
  * Features: Subscriptions, webhooks, checkout

[SECURITY MEASURES]
- CSRF protection enabled
- Token-based authentication
- Secure session management
- Environment-specific configurations

[DEVELOPMENT & DEPLOYMENT]
- Tools:
  * TypeScript (type safety)
  * Docker + Docker Compose (containerization)
  * Nginx (web server)
  * Celery (async task processing)

[DATABASE & CACHING]
- Primary Database: PostgreSQL
  * Features: Complex queries, transactions, ORM optimization
- Cache Layer: Redis

[INFRASTRUCTURE]
- Main Server: AWS EC2
- ML Processing: RunPod
  * Computer Vision: SAM2 (segmentation)
  * Pose Estimation: VitPoseH

[MOBILE APPLICATION]
- Framework: React Native

[PROJECT STRUCTURE]
- Repository Organization:
  * Separate frontend/backend repos
  * Modular component structure
  * Static resource management
  * Media file handling (video/images)

# ERROR FIXING PROCESS
Study the code, the logs and the context (related files, code blocks, etc). 
If clear, fix the issue and send the fixed code.
If not clear, 2 options:
  1. Add logs and ask user to run and send the logs.
  2. Ask user for more information.

# Our .env variables
.env
.env.sample

COMMENTS:
  - Make sure to ALWAYS add comments to the code. Simple and short.
  - Do not delete comments unless they are no longer needed or are outdated.


IMPORTANT: 
  - Understand the entire context of the project.
  - When working on a file or a part of the code, understand the related files and code. 
  - Follow best practices for the code, like DRY, SOLID, KISS, YAGNI. Keep the code modular and easy to understand.
  This means, the other code that this one calls, the code that calls this one and so on, api call codes...

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/galakurpi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/galakurpi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
