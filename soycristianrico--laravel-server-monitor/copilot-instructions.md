## laravel-server-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Laravel package (`cristian-dev/laravel-server-monitor`) that provides server monitoring and security scanning capabilities. The package monitors server resources (CPU, memory, disk) and performs security checks including malware detection, suspicious process monitoring, and system file change tracking.

**Namespace:** `SoyCristianRico\LaravelServerMonitor`

## Development Commands

### Testing
```bash
# Run all tests using Pest
vendor/bin/pest

# Run with coverage
vendor/bin/pest --coverage

# Using composer script
composer test
composer test-coverage
```

### Package Testing
The package uses Orchestra Testbench for Laravel package testing. Tests are organized into:
- `tests/Unit/` - Unit tests for individual services and commands
- `tests/Feature/` - Integration tests for full workflows

## Architecture

### Core Services
1. **ServerMonitoringService** (`src/Services/ServerMonitoringService.php`)
   - Monitors system resources (CPU, memory, disk, MySQL)
   - Executes system commands to gather metrics
   - Compares against configurable thresholds

2. **SecurityScannerService** (`src/Services/Security/SecurityScannerService.php`)
   - Performs security scans for malware patterns, suspicious processes
   - Monitors system files, SSH keys, failed logins
   - Uses caching to prevent duplicate alerts

3. **SecurityNotificationService** (`src/Services/Security/SecurityNotificationService.php`)
   - Handles alert notifications to admin users
   - Manages alert cooldowns and deduplication
   - Integrates with Laravel's notification system

### Console Commands
- `server:monitor` - Main server resource monitoring
- `security:check` - Comprehensive security scanning
- `security:check-malware` - Malware pattern detection in PHP files
- `security:monitor-crontabs` - Crontab modification tracking

All commands are registered in `ServerMonitorServiceProvider`.

### Configuration
Configuration is published to `config/server-monitor.php` with sections for:
- Monitoring thresholds (disk, memory, CPU)
- Notification settings (admin role, user model)
- Security settings (whitelists, excluded paths, cache duration)
- Scheduling suggestions

### Dependencies
- **spatie/laravel-permission**: Role-based access control for admin notifications
- **illuminate/notifications**: Laravel notification system for alerts
- Uses system commands (`df`, `free`, `uptime`, `netstat`, etc.) for monitoring

### Notifications
The package uses Laravel's notification system with:
- `SecurityAlertNotification` for sending alerts
- `NotifiesSecurityAlerts` trait for shared notification logic
- Email notifications to users with configured admin role

### Key Design Patterns
- Service-oriented architecture with dependency injection
- Laravel package structure with service provider registration
- Command pattern for console operations
- Observer pattern for system monitoring and alerting
- Caching for performance and alert deduplication

## Security Considerations
This package executes system commands for monitoring. All commands are:
- Hardcoded (no user input)
- Read-only operations
- Limited to standard system utilities
- Used only for defensive monitoring purposes

## Package Structure
```
src/
├── Config/server-monitor.php          # Configuration file
├── Console/Commands/                  # Artisan commands
├── Services/                          # Core business logic
├── Notifications/                     # Laravel notifications
├── Traits/                           # Shared behavior
└── ServerMonitorServiceProvider.php  # Package registration
```

---
> Source: [soyCristianRico/laravel-server-monitor](https://github.com/soyCristianRico/laravel-server-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
