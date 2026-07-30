## pfsense-pkg-restapi

> Dispatchers run work in the background so HTTP requests stay responsive. Use a Dispatcher for any operation that can take more than ~1 second: filter reloads, service restarts, certificate issuance, HA sync, etc.


# Dispatchers — `RESTAPI\Dispatchers\*`

Dispatchers run work in the background so HTTP requests stay responsive. Use a Dispatcher for any operation that can take more than ~1 second: filter reloads, service restarts, certificate issuance, HA sync, etc.

For full guidance and worked examples see:

- `.github/skills/validation-command-dispatcher.md`
- `docs/BUILDING_CUSTOM_DISPATCHER_CLASSES.md`

## Required shape

```php
<?php

namespace RESTAPI\Dispatchers;

use RESTAPI\Core\Dispatcher;

/**
 * Defines a Dispatcher that <does the long-running thing>.
 */
class <Area>ApplyDispatcher extends Dispatcher {
    public int $timeout = 300;          # optional override
    public int $max_queue = 10;         # optional override
    public string $schedule = '';       # 5-field cron string; set to register a CronJob
    protected array $required_packages = [];
    protected array $package_includes = [];

    protected function _process(mixed ...$arguments): void {
        # Do the actual work here. This runs in a forked PHP process.
    }
}
```

## Conventions

- Class name ends in `Dispatcher` (e.g. `FirewallApplyDispatcher`, `DNSResolverApplyDispatcher`, `RESTAPISettingsSyncDispatcher`).
- Implement `_process(mixed ...$arguments): void`. Never expose work via a public method other than the inherited `process()`/`spawn_process()`.
- Honor `$this->async` when the underlying pfSense function offers both sync and async forms (e.g. `filter_configure()` vs `filter_configure_sync()`).
- For Dispatchers that depend on a pfSense package, list it in `$required_packages` (full `pfSense-pkg-...` name) and any required PHP includes in `$package_includes`. Missing dependencies throw `FailedDependencyError`.
- Use `$this->schedule = '*/5 * * * *';` to register a `CronJob` automatically (handled by `manage.php` at install time).

## Triggering from a Model

```php
public function apply(): void {
    (new <Area>ApplyDispatcher(async: $this->async))->spawn_process();
}
```

`spawn_process()` returns a PID, writes a PID file under `/tmp/`, and respects `$max_queue` (overflow throws `ServiceUnavailableError` with `DISPATCHER_TOO_MANY_QUEUED_PROCESSES`).

## Hard rules

- **Never** call `filter_configure*()`, `services_*_configure()`, or other long-running pfSense functions directly from a Model — always wrap them in a Dispatcher.
- **Never** use bare `exec()` / `shell_exec()` / `passthru()`. Use `RESTAPI\Core\Command`.
- **Never** rely on `_process()` running synchronously in tests unless you instantiate with `async: false`.
- The PID file lifecycle is owned by `Core/Dispatcher.inc`. Do not write your own locking.

## Manual invocation (debugging)

On the pfSense host:

```bash
pfsense-restapi notifydispatcher <DispatcherClassShortName>
```

This runs the dispatcher synchronously, queues behind any existing instance, and writes `/tmp/<DispatcherName>.lock` while running.

---
> Source: [pfrest/pfSense-pkg-RESTAPI](https://github.com/pfrest/pfSense-pkg-RESTAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
