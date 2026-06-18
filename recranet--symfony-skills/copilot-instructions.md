## symfony-skills

> Idiomatic patterns and first-party components for Symfony 7.4+ projects - routing, Doctrine, security, forms, testing, and all core tasks


# Symfony 7.4+ Skill

Helps with any task in a Symfony 7.4+ project: idiomatic patterns, first-party
components, and configuration. All examples pin to the 7.4 documentation.

## Tooling — ddev when present

Before running any shell command:

- If the project has `.ddev/config.yaml`, prefix with `ddev`. Examples:
  `ddev php bin/console make:migration`, `ddev composer require lock`,
  `ddev php bin/console doctrine:migrations:migrate`.
- Otherwise use the host environment directly:
  `php bin/console …`, `composer …`. The Symfony CLI (`symfony console …`)
  is fine if installed.

This applies to every command in the references below — substitute the
prefix as needed.

## Version check — gate version-specific features

Some patterns in this skill require a minimum Symfony version. Detect the
installed version once per session before using them:

```bash
composer show symfony/framework-bundle --no-ansi | grep versions
# fallback: grep '"symfony/framework-bundle"' composer.json
```

| Installed version | What applies |
|-------------------|--------------|
| >= 8.1 | Everything, including [whats-new-8.1.md](references/whats-new-8.1.md) |
| 7.4 – 8.0 | Core references + [whats-new-7.4.md](references/whats-new-7.4.md); **no** 8.1 features |
| < 7.4 | Core references only; skip both what's-new pages |

Never suggest a feature from a what's-new page without confirming the
version meets its gate. When a newer feature would be the ideal fit but
the project's version is too old, use the classic pattern and mention the
upgrade option.

## 1. Working in a Symfony project

Standard layout:

```
.
├── bin/console               # CLI entrypoint
├── config/
│   ├── bundles.php
│   ├── packages/             # bundle configuration (one file per bundle)
│   ├── routes/               # route imports
│   ├── routes.yaml
│   └── services.yaml
├── migrations/               # DoctrineMigrationsBundle
├── public/                   # web root, contains index.php
├── src/                      # PSR-4 namespace App\
│   ├── Controller/
│   ├── Entity/
│   ├── Repository/
│   ├── Form/
│   ├── Security/
│   ├── EventListener/
│   ├── Command/
│   └── ...
├── templates/                # Twig
├── tests/                    # PSR-4 namespace App\Tests\
├── translations/
├── var/                      # cache, log (git-ignored)
├── vendor/                   # composer (git-ignored)
├── .env, .env.local          # env vars
├── composer.json
├── symfony.lock              # Flex lock
```

Conventions in `config/services.yaml`:

- `autowire: true`, `autoconfigure: true`, `public: false`
- Classes under `App\` are auto-registered as services
- Add new services by just creating the class — no XML/YAML wiring needed
  unless tags or arguments need overriding

For details, see [`references/project-layout.md`](references/project-layout.md).

## 2. Common tasks

| Task | Reference | Docs |
|------|-----------|------|
| Use a Symfony 7.4 feature (requires >= 7.4) | [whats-new-7.4.md](references/whats-new-7.4.md) | https://symfony.com/blog/symfony-7-4-curated-new-features |
| Use a Symfony 8.1 feature (requires >= 8.1) | [whats-new-8.1.md](references/whats-new-8.1.md) | https://symfony.com/blog/symfony-8-1-curated-new-features |
| Add a route / controller | [controllers-and-routing.md](references/controllers-and-routing.md) | https://symfony.com/doc/7.4/controller.html |
| Add an entity / repository / migration | [doctrine.md](references/doctrine.md) | https://symfony.com/doc/7.4/doctrine.html |
| Add a console command | [console-commands.md](references/console-commands.md) | https://symfony.com/doc/7.4/console.html |
| Debug routes / services / config / DB | [debugging.md](references/debugging.md) | — |
| React to an event | [events.md](references/events.md) | https://symfony.com/doc/7.4/event_dispatcher.html |
| Secure an endpoint / firewall / voter | [security.md](references/security.md) | https://symfony.com/doc/7.4/security.html |
| Write a unit / functional / browser test | [testing.md](references/testing.md) | https://symfony.com/doc/7.4/testing.html |
| Configure the framework (`framework:` tree) | [framework-config.md](references/framework-config.md) | https://symfony.com/doc/current/reference/configuration/framework.html |
| Validate input | [validator.md](references/validator.md) | https://symfony.com/doc/7.4/validation.html |
| Send email | [mailer.md](references/mailer.md) | https://symfony.com/doc/7.4/mailer.html |
| Send notification (chat/SMS/push) | [notifier.md](references/notifier.md) | https://symfony.com/doc/7.4/notifier.html |
| Run background / queued work | [messenger.md](references/messenger.md) | https://symfony.com/doc/7.4/messenger.html |
| Schedule recurring work | [scheduler.md](references/scheduler.md) | https://symfony.com/doc/7.4/scheduler.html |
| Make HTTP calls outbound | [http-client.md](references/http-client.md) | https://symfony.com/doc/7.4/http_client.html |
| Cache expensive work | [cache.md](references/cache.md) | https://symfony.com/doc/7.4/cache.html |
| Acquire a lock | [lock.md](references/lock.md) | https://symfony.com/doc/7.4/lock.html |
| Rate-limit something | [rate-limiter.md](references/rate-limiter.md) | https://symfony.com/doc/7.4/rate_limiter.html |
| Model a state machine | [workflow.md](references/workflow.md) | https://symfony.com/doc/7.4/workflow.html |
| Serialize/deserialize | [serializer.md](references/serializer.md) | https://symfony.com/doc/7.4/serializer.html |
| Run an external process | [process.md](references/process.md) | https://symfony.com/doc/7.4/components/process.html |
| Walk files / find files | [finder.md](references/finder.md) | https://symfony.com/doc/7.4/components/finder.html |
| Filesystem operations | [filesystem.md](references/filesystem.md) | https://symfony.com/doc/7.4/components/filesystem.html |
| String manipulation / slugs | [string.md](references/string.md) | https://symfony.com/doc/7.4/components/string.html |
| Generate UUIDs / ULIDs | [uid.md](references/uid.md) | https://symfony.com/doc/7.4/components/uid.html |
| Get "now" testably | [clock.md](references/clock.md) | https://symfony.com/doc/7.4/components/clock.html |
| Sanitize user-submitted HTML | [html-sanitizer.md](references/html-sanitizer.md) | https://symfony.com/doc/7.4/html_sanitizer.html |
| Build a form | — | https://symfony.com/doc/7.4/forms.html |
| Render with Twig | — | https://symfony.com/doc/7.4/templates.html |
| Translate strings | — | https://symfony.com/doc/7.4/translation.html |
| Manage env / secrets | — | https://symfony.com/doc/7.4/configuration/env_var_processors.html |

## 3. Component catalog

Quick scan of Symfony components grouped by problem area. Use this when
deciding between a Symfony component and a third-party library — for any
row whose "Replaces" column matches what you're about to reach for, use
the Symfony component.

### Network / I/O

| Component | Replaces | Reference |
|-----------|----------|-----------|
| `symfony/http-client` | Guzzle, curl wrappers, file_get_contents for HTTP | [http-client.md](references/http-client.md) |
| `symfony/mailer` | PHPMailer, SwiftMailer, vendor SMTP SDKs | [mailer.md](references/mailer.md) |
| `symfony/notifier` | Twilio/Slack/Discord/Telegram SDKs for notifications | [notifier.md](references/notifier.md) |
| `symfony/mime` | nesbot/MIME builders, custom MIME assembly | — |

### Data & state

| Component | Replaces | Reference |
|-----------|----------|-----------|
| `symfony/cache` | Direct Redis/Memcached/APCu clients | [cache.md](references/cache.md) |
| `symfony/lock` | Redis SETNX wrappers, flock helpers, custom mutexes | [lock.md](references/lock.md) |
| `symfony/semaphore` | Hand-rolled concurrency limits | — |
| `symfony/rate-limiter` | Hand-rolled rate limit code, third-party limiters | [rate-limiter.md](references/rate-limiter.md) |
| `symfony/messenger` | php-amqplib direct use, vendor queue SDKs | [messenger.md](references/messenger.md) |
| `symfony/scheduler` | Cron-based packages, hand-rolled scheduling | [scheduler.md](references/scheduler.md) |
| `symfony/workflow` | Hand-rolled state machines | [workflow.md](references/workflow.md) |

### Filesystem & process

| Component | Replaces | Reference |
|-----------|----------|-----------|
| `symfony/filesystem` | Raw `mkdir`, `rename`, `copy`, `unlink` | [filesystem.md](references/filesystem.md) |
| `symfony/finder` | `glob`, `scandir`, `RecursiveIteratorIterator` | [finder.md](references/finder.md) |
| `symfony/process` | `exec`, `shell_exec`, `proc_open` | [process.md](references/process.md) |

### Data transformation & validation

| Component | Replaces | Reference |
|-----------|----------|-----------|
| `symfony/serializer` | JMS Serializer, hand-rolled (de)serialization | [serializer.md](references/serializer.md) |
| `symfony/validator` | Hand-rolled validation, respect/validation | [validator.md](references/validator.md) |
| `symfony/string` | Ad-hoc `mb_*` usage, custom slug functions | [string.md](references/string.md) |
| `symfony/uid` | `ramsey/uuid` for most cases | [uid.md](references/uid.md) |
| `symfony/html-sanitizer` | HTMLPurifier | [html-sanitizer.md](references/html-sanitizer.md) |
| `symfony/clock` | Direct `new \DateTimeImmutable()` (for testability) | [clock.md](references/clock.md) |
| `symfony/expression-language` | `eval`, custom rule DSLs | — |

### HTTP / framework essentials

Already idiomatic when working in Symfony — listed for completeness, no
reference files needed.

`symfony/http-foundation`, `symfony/routing`, `symfony/security-bundle`,
`symfony/form`, `symfony/twig-bundle`, `symfony/translation`,
`symfony/console`, `symfony/dependency-injection`,
`symfony/event-dispatcher`, `symfony/yaml`, `symfony/config`,
`symfony/runtime`, `symfony/asset`, `symfony/asset-mapper`,
`symfony/web-link`.

### Testing / crawling

`symfony/panther`, `symfony/dom-crawler`, `symfony/browser-kit`,
`symfony/css-selector`. See [testing.md](references/testing.md).

## 4. Decision rule

When picking an approach:

1. **Task-shaped work** (routes, entities, console commands, events,
   security, tests, config) — open the matching task reference and follow
   its patterns. Don't invent a new convention.
2. **Component choice** — before `composer require`-ing a third-party
   library, check the catalog above. If a Symfony component covers the
   use case, use it; only fall back to a third-party library when the
   Symfony component genuinely doesn't fit (call that out explicitly).
3. **Configuration** — for any `framework:` key, consult
   [framework-config.md](references/framework-config.md) before
   guessing the option name or value.
4. **All examples target Symfony 7.4.** Patterns from older versions
   (`MessageHandlerInterface`, route YAML when attributes work, public
   service ids) are out of scope.
5. **Version-gated features** — anything from
   [whats-new-7.4.md](references/whats-new-7.4.md) or
   [whats-new-8.1.md](references/whats-new-8.1.md) requires the version
   check from "Version check" above to pass first.

## Flex aliases

Many components are installable by short name when Flex is present:

```
composer require lock           # symfony/lock
composer require http-client    # symfony/http-client
composer require mailer         # symfony/mailer
composer require cache          # already in symfony/framework-bundle
composer require messenger      # symfony/messenger
composer require notifier       # symfony/notifier
composer require scheduler      # symfony/scheduler
composer require workflow       # symfony/workflow
composer require uid            # symfony/uid
composer require validator      # symfony/validator
composer require serializer     # symfony/serializer-pack
composer require security       # symfony/security-bundle
composer require form           # symfony/form
composer require twig           # symfony/twig-bundle
composer require translation    # symfony/translation
composer require test           # symfony/test-pack
composer require maker --dev    # symfony/maker-bundle
composer require profiler --dev # symfony/web-profiler-bundle
```

Prefer aliases — they install the matching recipe (config, env vars, services).

---
> Source: [recranet/symfony-skills](https://github.com/recranet/symfony-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
