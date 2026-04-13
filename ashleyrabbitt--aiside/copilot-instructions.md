## aiside

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MagicAI v8.70 is a Laravel-based AI SaaS platform that integrates multiple AI providers to offer text generation, image creation, voice synthesis, and chat functionality. It uses a modular extension system for features and supports multiple payment gateways with subscription management.

## Common Development Commands

### Frontend Development
```bash
npm install              # Install frontend dependencies
npm run dev              # Start Vite dev server with HMR
npm run build            # Build production assets
npm run lint             # Run ESLint on resources/views/page
```

### Backend Development
```bash
composer install         # Install PHP dependencies
php artisan serve        # Start development server
php artisan migrate      # Run database migrations
php artisan db:seed      # Seed the database
php artisan storage:link # Create storage symlink
```

### Testing
```bash
composer test            # Run Pest tests with parallel execution
composer test:coverage   # Run tests with code coverage
php artisan test         # Run custom test command
```

### Code Quality
```bash
composer lint            # Format PHP code with Laravel Pint
composer test:lint       # Test code style without changes
npm run lint             # Lint JavaScript files
```

### Cache Management
```bash
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan optimize:clear
```

### Queue & Scheduling
```bash
php artisan queue:work   # Process queue jobs
php artisan schedule:run # Run scheduled tasks
```

## Architecture Overview

### Extension System
The application uses a modular extension architecture located in `/app/Extensions/`. Each extension:
- Has its own ServiceProvider in `/app/Providers/`
- Contains routes, controllers, views, and migrations
- Can be enabled/disabled independently
- Examples: AI tools (Plagiarism, RealtimeImage), integrations (Midjourney, Mailchimp), UI features

### AI Provider Integration
Multiple AI providers are integrated through a unified service layer:
- Service classes in `/app/Services/AI/` handle provider-specific logic
- Configuration files in `/config/` for each provider
- Supports OpenAI, Google AI, Anthropic, DeepSeek, xAI, and more

### Payment System
- Multiple payment gateways configured in `/config/payment.php`
- Laravel Cashier for subscription management
- Custom artisan commands for checking payment statuses
- Credit-based usage system with team support

### Frontend Architecture
- Blade templates with component-based structure
- Alpine.js for reactive UI components
- TailwindCSS with custom configuration per section
- Vite for asset bundling
- Livewire for real-time interactions

### Key Models and Relationships
- User → Teams → Members (multi-tenancy)
- User → Subscriptions → Plans
- User → OpenAIGenerators (AI content)
- User → UserOpenaiChatMessages (conversations)
- Extension system with dynamic model loading

### Custom Artisan Commands
Important maintenance commands:
- `app:check-subscription-end` - Monitor expired subscriptions
- `app:check-[gateway]` - Check payment gateway statuses
- `app:clear-*` - Various data cleanup commands

### Environment Configuration
Key environment variables to configure:
- Database connection
- Redis for caching/queues
- AI provider API keys
- Payment gateway credentials
- Mail configuration
- Storage configuration (local/S3)

## Development Tips

### Working with Extensions
When modifying or creating extensions:
1. Check the extension's ServiceProvider for route and view registration
2. Extensions have their own migration files
3. Use the existing extension structure as a template

### AI Provider Integration
When adding new AI features:
1. Create service class in `/app/Services/AI/`
2. Add configuration in `/config/`
3. Update relevant controllers and views
4. Consider credit consumption logic

### Testing Approach
- Use Pest PHP for testing
- Tests are located in `/tests/`
- Run with parallel execution for speed
- Check coverage reports in `/coverage/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ashleyrabbitt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ashleyrabbitt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
