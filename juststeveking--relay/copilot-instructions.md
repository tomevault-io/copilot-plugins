## relay

> 0. Context & Philosophy

0. Context & Philosophy

Relay is a zero-bloat, fiber-native transport layer. It treats HTTP requests as immutable Signals flowing through a Conduit.

Target Environment: PHP 8.5+ (using Native Fibers).

Core Directive: Never block the main thread. Never use Exceptions for expected HTTP failures.

1. Architectural Constraints for AI

When generating code for this repository, you must adhere to these rules:

A. Immutability (The Nouns)
All Request/State objects must be readonly class.

Modifications must use the `with()` pattern which returns a new instance.

B. Functional Flow (The Verbs)
Actions should be static methods or first-class callables.

Terminal methods (like send) must be tagged with #[NoDiscard].

C. Error Handling
Forbidden: Throwing exceptions for 4xx or 5xx status codes.

Mandatory: Return the Signal interface (Success|Failure).

Use `match` or `instanceof` for result processing.

2. Core Type Manifest

```PHP
namespace JustSteveKing\Relay\Http;

use JustSteveKing\Relay\Contracts\Signal;

/**
 * @template T
 */
readonly class Success implements Signal {
    public function __construct(
        public mixed $data, 
        public Status $status, 
        public array $headers = []
    ) {}
}

readonly class Failure implements Signal {
    public function __construct(
        public string $reason, 
        public Status $status, 
        public array $context = []
    ) {}
}

interface Signal {}
```

3. The WarpCore (Fiber Scheduler)

All I/O must be orchestrated by the Scheduler. It uses curl_multi_get_handles() to poll without blocking.

```php
namespace JustSteveKing\Relay\Http;

use Fiber;

final class Scheduler {
    /**
     * Suspends the current Fiber until cURL I/O is ready.
     * @return Signal
     */
    public static function await(\CurlHandle $ch): Signal {
        // 1. Add to multi-handle
        // 2. Fiber::suspend()
        // 3. On resume: return Success|Failure
    }

    /**
     * Runs the fiber loop.
     */
    public static function run(callable $main): void {
        // ...
    }
}
```

4. Example Implementation Pattern

If asked to create a "Starbase" resource, generate it like this:

```php
namespace JustSteveKing\Relay\Actions;

use JustSteveKing\Relay\Http\{Request, Transceiver, Method, Status};
use JustSteveKing\Relay\Contracts\Signal;

class Starbase {
    public static function hail(string $baseId): Signal {
        $request = new Request(
            uri: "https://api.starfleet.org/v1/starbase/{$baseId}",
            method: Method::GET
        );
        
        return Transceiver::send($request);
    }
}
```

5. Concurrency Pattern

To run multiple requests in parallel, use the `Concurrent::run` helper. This handles fiber management automatically.

```php
use JustSteveKing\Relay\Http\Concurrent;

$results = Concurrent::run([
    'users' => fn() => Transceiver::send($usersReq),
    'posts' => fn() => Transceiver::send($postsReq),
]);
```

For limiting concurrency, use `Pool::work()`:

```php
use JustSteveKing\Relay\Http\Pool;
$results = Pool::work($tasks, concurrency: 5);
```

For racing requests, use `Race::any()`:

```php
use JustSteveKing\Relay\Http\Race;
$winner = Race::any([$taskA, $taskB]);
```

6. Metadata for Gemini CLI
Project Name: Relay

Vendor: juststeveking

Primary Extension: ext-curl, ext-fiber

Star Trek Reference Level: Subtle (Relay, Transceiver, Signal).

---
> Source: [JustSteveKing/relay](https://github.com/JustSteveKing/relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
