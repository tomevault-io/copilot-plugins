## unifi-portal-test

> - `yarn dev` - Start frontend dev server with HMR

# UniFi Portal Development Guide

## Build Commands
- `yarn dev` - Start frontend dev server with HMR
- `yarn prod` - Build frontend for production
- `composer dev` - Optimize autoloader for development
- `composer prod` - Install and optimize for production
- `composer test` - Run PHPUnit tests
- `composer test tests/SpecificTest.php` - Run single test
- `composer phpcs` - Run PHP CodeSniffer
- `composer phpcbf` - Fix PHP CodeSniffer errors

## Code Style Guidelines
- **PHP**: PSR-12 standard for most files, WordPress standard for WP-specific files
- **Vue**: Use Composition API, document components with comments
- **Naming**: PascalCase for classes/components, camelCase for methods/properties
- **Imports**: Use autoloader for PHP, auto-imports for Vue components
- **Types**: Use type hints and return types, document array shapes
- **Error Handling**: Try-catch blocks with consistent response format
- **CSS**: BEM naming convention, SCSS for styling
- **Security**: Always validate user input, check capabilities for admin actions
- **Architecture**: Follow namespace PSR-4 structure, organize by feature

## Project Structure
Frontend in `src/frontend/templates/unifi-portal`
PHP classes in PSR-4 namespaced directories under `src/`
Tests in `tests/` directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafa2701) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
