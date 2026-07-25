## pfsense-pkg-restapi

> Contributor and AI-agent guide for **pfSense-pkg-RESTAPI** (https://pfrest.org).

# AGENTS.md

Contributor and AI-agent guide for **pfSense-pkg-RESTAPI** (https://pfrest.org).

This file is the **single source of truth** for how code in this repository is structured and how new contributions must be authored. It is intentionally opinionated. Read it end-to-end before proposing changes.

For deeper, file-scoped guidance see:

- `.github/skills/` — task-oriented playbooks (endpoints/models/fields, validators/commands/dispatchers, writing tests, full feature walkthroughs).
- `.github/instructions/` — GitHub Copilot path-scoped instructions that auto-apply when you edit specific directories (Models, Endpoints, Validators, Dispatchers, Tests).
- `docs/CONTRIBUTING.md` — human-facing contribution and build guide.
- `docs/BUILDING_CUSTOM_*` — long-form references for each subsystem.
- `https://pfrest.org/php-docs/` — generated PHPDoc for every class.

---

## 1. What this project is

`pfSense-pkg-RESTAPI` adds a fully featured REST and GraphQL API to **pfSense CE**. It is implemented as a FreeBSD package that installs PHP code under `/usr/local/pkg/RESTAPI` on a pfSense host.

The framework is **declarative and metadata-driven**: Endpoints, Models, Fields, Validators, Dispatchers, Auth, ContentHandlers, and Responses describe the system, and the runtime generates:

- HTTP endpoint PHP files in the pfSense webroot
- pfSense privileges (ACL entries) per endpoint and method
- OpenAPI 3 / Swagger documentation
- A GraphQL schema
- Background dispatcher cron jobs
- Optional webConfigurator forms

If you bypass the framework's metadata (e.g. by hand-writing handlers or hard-coding shapes), you silently break documentation, schema generation, privileges, HA sync, and tests. **Don't.**

---

## 2. Repository layout (only the parts you'll touch)

```
pfSense-pkg-RESTAPI/files/usr/local/pkg/RESTAPI/
├── Auth/              # Auth methods (BasicAuth, JWTAuth, KeyAuth)
├── Caches/            # Scheduled JSON dataset producers
├── ContentHandlers/   # Request/response Content-Type and Accept handlers
├── Core/              # Framework base classes (do not modify without maintainer approval)
│   ├── Auth.inc  Cache.inc  Command.inc  ContentHandler.inc
│   ├── Dispatcher.inc  Endpoint.inc  Field.inc  Form.inc
│   ├── Model.inc  ModelSet.inc  QueryFilter.inc
│   ├── Response.inc  Schema.inc  TestCase.inc  Tools.inc  Validator.inc
├── Dispatchers/       # Long-running / background workers
├── Endpoints/         # URL → Model adapters (thin)
├── Fields/            # Field types (StringField, IntegerField, ForeignModelField, ...)
├── Forms/             # webConfigurator form pages
├── Models/            # Feature logic (config schema + apply behavior)
├── ModelTraits/       # Cross-Model mixins (e.g. log file traits)
├── QueryFilters/      # Query operators (e.g. exact, regex, gt, lt)
├── Responses/         # Throwable HTTP responses (Success, ValidationError, ...)
├── Schemas/           # Schema generators (OpenAPISchema, GraphQLSchema, NativeSchema)
├── Tests/             # Custom *TestCase.inc files (see §10)
├── Validators/        # Reusable Field-level validation classes
├── autoloader.inc
└── .resources/
    ├── cache/         # Cache file output
    ├── schemas/       # Generated schema artifacts
    └── scripts/       # dispatch.sh, manage.php (build/runtime CLI)
```

Other top-level items:

- `tools/make_package.py` — builds the FreeBSD package.
- `vagrant-build.sh` + `Vagrantfile` — builds via a FreeBSD VM on your local machine.
- `docs/` — MkDocs source for https://pfrest.org.
- `composer.json` / `package.json` — runtime PHP deps and dev tooling (Prettier + plugin-php, Spectral).
- `.github/workflows/` — CI: Prettier, Black, phplint, phpdoc, package build, OpenAPI lint, runtests on real pfSense VMs.

---

## 3. Mental model: how a request flows

```
HTTP → generated PHP file in webroot
     → Endpoint (Auth → ACL → method dispatch → ContentHandler)
        → Model (validate Fields → run validators → validate_FIELD_NAME / validate_extra
                 → write to pfSense config (or call internal_callable) → apply())
           → apply() may spawn a Dispatcher process (filter reload, service restart, ...)
        ← Model returns representation
     ← Endpoint serializes via ContentHandler → Response
```

Two implications:

1. The Endpoint layer is essentially a router/adapter. Everything that _does_ something belongs in the Model (or in a Dispatcher invoked from the Model).
2. Field metadata is the schema. OpenAPI types, GraphQL types, validation, defaults, choices, sensitivity, pagination, and HATEOAS links all derive from Field/Model/Endpoint properties.

---

## 4. Non-negotiable rules

These are enforced by maintainers in code review.

1. **Model-first.** Every Endpoint sets `model_name` and exposes a Model. Endpoints **must not** override `get()`, `post()`, `patch()`, `put()`, `delete()` unless a maintainer has explicitly approved an exception.
2. **Field-driven schema.** Object shape, types, defaults, choices, constraints, sensitivity, conditional visibility, and help text live on `Field` objects in the Model constructor. Do not hand-write OpenAPI/GraphQL.
3. **Reusable validation goes in `Validators/`.** Use `validate_FIELD_NAME()` / `validate_extra()` in Models only for logic that is genuinely Model-specific.
4. **Shell execution uses `RESTAPI\Core\Command`.** Avoid bare `exec()` / `shell_exec()` / `passthru()` in feature code.
5. **Long-running work uses a Dispatcher.** Anything that reloads filters, restarts services, syncs HA peers, applies configuration, or otherwise can take >1s belongs in `Dispatchers/` and is invoked from the Model's `apply()` via `spawn_process()`.
6. **No pfSense Plus code.** Only reference functionality and source available in pfSense **CE**. If you cannot find the symbol/feature in CE source, you cannot use it here.
7. **No hand-written endpoint PHP in the webroot.** The webroot files are generated by `manage.php buildendpoints`. Do not check them in or edit them.
8. **Privileges are auto-generated.** Don't hard-code privilege names; they are derived from `$url` + method.
9. **Secrets are `sensitive: true` Fields.** Never log secrets. Never echo them through generic responses.

---

## 5. Coding conventions

- **PHP version:** 8.2 (CI matrix). Use named arguments for clarity (the codebase does this consistently, e.g. `new StringField(required: true, ...)`).
- **File extension:** `.inc` for all PHP class files in `pfSense-pkg-RESTAPI/files/usr/local/pkg/RESTAPI/`.
- **Top of every class file:**

  ```php
  <?php

  namespace RESTAPI\<Subnamespace>;

  require_once 'RESTAPI/autoloader.inc';

  use RESTAPI\Core\<...>;
  ```

- **Comments:** Use `#` for short inline notes (matches the existing codebase). Use `/** ... */` PHPDoc on every public method and class — PHPDoc is checked in CI.
- **Naming:**
  - Models: `PascalCase`, single noun (singular). Example: `FirewallAlias`, `DNSResolverHostOverrideAlias`.
  - Endpoints: `<Area><Resource>Endpoint` for singular, plural form (e.g. `FirewallAliasesEndpoint`) for `many: true` collection endpoints. Apply endpoints: `<Area>ApplyEndpoint`.
  - Validators: `<Thing>Validator` (e.g. `IPAddressValidator`).
  - Dispatchers: `<Thing>Dispatcher` (e.g. `FirewallApplyDispatcher`).
  - Tests: `API<Subnamespace><ClassUnderTest>TestCase` (e.g. `APIModelsFirewallAliasTestCase`, `APIValidatorsIPAddressValidatorTestCase`, `APICoreEndpointTestCase`).
  - `response_id` values: `UPPER_SNAKE_CASE`, prefixed by the class or feature (e.g. `INVALID_FIREWALL_ALIAS_NAME`, `IP_ADDRESS_VALIDATOR_FAILED`).
- **Constructor signature for Models is fixed:**
  ```php
  public function __construct(
      mixed $id = null,
      mixed $parent_id = null,
      mixed $data = [],
      mixed ...$options,
  ) {
      # 1. Set Model attributes (config_path, many, subsystem, ...)
      # 2. Define Field objects on $this
      # 3. parent::__construct() MUST be the last statement
      parent::__construct($id, $parent_id, $data, ...$options);
  }
  ```
- **Endpoint constructors** set `$this->url`, `$this->model_name`, `$this->request_method_options`, optional `$this->many`, optional help text, then call `parent::__construct();` last.
- **Formatting:** Prettier with `@prettier/plugin-php` is authoritative. Run before pushing:
  ```bash
  npm install
  ./node_modules/.bin/prettier --write ./pfSense-pkg-RESTAPI/files
  ```
  Python uses Black: `pip install -r requirements.txt && black .`

---

## 6. Authoring an Endpoint (the thin layer)

An Endpoint is _only_ a URL → Model adapter plus auth/method exposure. Most endpoints are 10–25 lines.

### Single-object endpoint

```php
<?php

namespace RESTAPI\Endpoints;

require_once "RESTAPI/autoloader.inc";

use RESTAPI\Core\Endpoint;

class FirewallAliasEndpoint extends Endpoint
{
  public function __construct()
  {
    $this->url = "/api/v2/firewall/alias";
    $this->model_name = "FirewallAlias";
    $this->request_method_options = ["GET", "POST", "PATCH", "DELETE"];
    parent::__construct();
  }
}
```

### Collection (`many`) endpoint

```php
class FirewallAliasesEndpoint extends Endpoint
{
  public function __construct()
  {
    $this->url = "/api/v2/firewall/aliases";
    $this->model_name = "FirewallAlias";
    $this->many = true;
    $this->request_method_options = ["GET", "PUT", "DELETE"];
    parent::__construct();
  }
}
```

Method semantics (enforced in `Core/Endpoint.inc::check_construct`):

| `many`  | Method | Calls Model method |
| ------- | ------ | ------------------ |
| `false` | GET    | `read()`           |
| `false` | POST   | `create()`         |
| `false` | PATCH  | `update()`         |
| `false` | DELETE | `delete()`         |
| `true`  | GET    | `read_all()`       |
| `true`  | PUT    | `replace_all()`    |
| `true`  | DELETE | `delete_many()`    |

Other useful Endpoint properties (set when needed, leave default otherwise): `requires_auth`, `auth_methods`, `ignore_read_only`, `ignore_interfaces`, `ignore_enabled`, `ignore_acl`, `tag`, `*_help_text`, `deprecated`, `limit`, `offset`, `sort_by`, `encode_content_handlers`, `decode_content_handlers`, `resource_link_set`. See `Core/Endpoint.inc` for the full annotated list.

---

## 7. Authoring a Model (where features live)

Use Field objects to express **what** the resource is; use methods only for **how** to validate/apply the model-specific bits.

```php
<?php

namespace RESTAPI\Models;

require_once "RESTAPI/autoloader.inc";

use RESTAPI\Core\Model;
use RESTAPI\Dispatchers\FirewallApplyDispatcher;
use RESTAPI\Fields\StringField;
use RESTAPI\Responses\ValidationError;
use RESTAPI\Validators\FilterNameValidator;

class FirewallAlias extends Model
{
  public StringField $name;
  public StringField $type;
  public StringField $address;

  public function __construct(
    mixed $id = null,
    mixed $parent_id = null,
    mixed $data = [],
    mixed ...$options,
  ) {
    # Model attributes describe storage + behavior
    $this->config_path = "aliases/alias";
    $this->subsystem = "aliases";
    $this->many = true;

    # Fields describe the schema
    $this->name = new StringField(
      required: true,
      unique: true,
      editable: false,
      maximum_length: 31,
      validators: [new FilterNameValidator()],
      verbose_name: "Name",
      help_text: "Sets the name for the alias.",
    );
    $this->type = new StringField(
      required: true,
      choices: ["host", "network", "port"],
      help_text: "Sets the alias type.",
    );
    $this->address = new StringField(
      default: [],
      allow_empty: true,
      many: true,
      delimiter: " ",
      help_text: "Entries belonging to the alias.",
    );

    parent::__construct($id, $parent_id, $data, ...$options);
  }

  # Model-specific validation only — reusable rules belong in Validators/
  public function validate_name(string $name): string
  {
    if (!is_validaliasname($name)) {
      throw new ValidationError(
        message: "Invalid firewall alias name '$name'.",
        response_id: "INVALID_FIREWALL_ALIAS_NAME",
      );
    }
    return $name;
  }

  # Apply side effects via a Dispatcher (do not block the request)
  public function apply(): void
  {
    new FirewallApplyDispatcher(async: $this->async)->spawn_process();
  }
}
```

### When to use which override

| Need                                                      | Mechanism                                                                            |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Validate one field with reusable logic                    | `Validators/` class via `validators: [...]`                                          |
| Validate one field with model-specific logic              | `validate_<field_name>(<value>): <type>` returning the validated value               |
| Cross-field validation                                    | `validate_extra()`                                                                   |
| Replace how the Model is read (no pfSense config backing) | `internal_callable = 'method_name'` returning array(s)                               |
| Replace how the Model is created/updated/deleted          | `_create()` / `_update()` / `_delete()` (rare; only when not config-backed)          |
| Run something after a successful write                    | `apply()` — typically `(new XApplyDispatcher(async: $this->async))->spawn_process()` |

### Useful Model attributes

`config_path`, `internal_callable`, `subsystem`, `many`, `many_minimum`, `many_maximum`, `id_type`, `parent_model_class`, `parent_id_type`, `cache_class`, `unique_together_fields`, `protected_model_query`, `packages`, `package_includes`, `sort_by`, `sort_order`, `sort_flags`, `placement`, `always_apply`, `update_strategy`, `verbose_name`, `verbose_name_plural`. See `Core/Model.inc` PHPDoc for full semantics.

---

## 8. Fields, Validators, Commands, Dispatchers — at a glance

- **Fields** (`Fields/`): `StringField`, `IntegerField`, `FloatField`, `BooleanField`, `Base64Field`, `DateTimeField`, `UnixTimeField`, `PortField`, `InterfaceField`, `FilterAddressField`, `SpecialNetworkField`, `ForeignModelField`, `NestedModelField`, `ObjectField`, `UIDField`. Common constructor args: `required`, `unique`, `default`, `default_callable`, `choices`, `choices_callable`, `allow_empty`, `allow_null`, `editable`, `read_only`, `write_only`, `sensitive`, `representation_only`, `many`, `many_minimum`, `many_maximum`, `delimiter`, `internal_name`, `internal_namespace`, `conditions`, `validators`, `verbose_name`, `verbose_name_plural`, `help_text`. See `Core/Field.inc` PHPDoc.
- **Validators** (`Validators/`): each extends `RESTAPI\Core\Validator` and implements `validate(mixed $value, string $field_name = ''): void`, throwing `ValidationError` with a stable `response_id`. Reuse before adding new ones. Existing: `EmailAddressValidator`, `FilterNameValidator`, `HexValidator`, `HostnameValidator`, `IPAddressValidator`, `LengthValidator`, `MACAddressValidator`, `NumericRangeValidator`, `RegexValidator`, `SubnetValidator`, `URLValidator`, `UniqueFromForeignModelValidator`, `X509Validator`.
- **Command** (`Core/Command.inc`): `new Command('/sbin/pfctl -sr')` exposes `->output`, `->result_code`. Use this anywhere you previously would have called `exec()`.
- **Dispatcher** (`Core/Dispatcher.inc`): subclass and implement `protected function _process(mixed ...$arguments): void`. Trigger from a Model with `(new MyDispatcher(async: $this->async))->spawn_process()`. Optional properties: `timeout` (default 300s), `max_queue` (default 10), `schedule` (5-field cron string), `required_packages`, `package_includes`.

See `.github/skills/validation-command-dispatcher.md` for full examples.

---

## 9. Responses

Throw, don't return. The Endpoint base class catches Response exceptions and serializes them.

| Class                       | HTTP code | Use when                                                                           |
| --------------------------- | --------- | ---------------------------------------------------------------------------------- |
| `Success`                   | 200       | The Model returned successfully (handled by base classes — you rarely throw this). |
| `ValidationError`           | 400       | Bad input; required by validators and `validate_*` methods.                        |
| `AuthenticationError`       | 401       | Auth failed.                                                                       |
| `ForbiddenError`            | 403       | Auth succeeded but no privilege / ACL denied.                                      |
| `NotFoundError`             | 404       | Resource does not exist.                                                           |
| `MethodNotAllowedError`     | 405       | Method not in `request_method_options`.                                            |
| `NotAcceptableError`        | 406       | Requested Accept content type not supported.                                       |
| `ConflictError`             | 409       | State conflict (e.g. attempting to delete a referenced object).                    |
| `MediaTypeError`            | 415       | Unsupported Content-Type.                                                          |
| `UnprocessableContentError` | 422       | Semantic error in well-formed request.                                             |
| `FailedDependencyError`     | 424       | Required pfSense package or include missing.                                       |
| `ServerError`               | 500       | Unexpected internal failure.                                                       |
| `ServiceUnavailableError`   | 503       | Dispatcher queue full / temporarily unavailable.                                   |

Always pass a stable `response_id` so clients can switch on it.

---

## 10. Tests (custom framework — not PHPUnit)

The framework is `RESTAPI\Core\TestCase`. Tests live in `pfSense-pkg-RESTAPI/files/usr/local/pkg/RESTAPI/Tests/`.

### File and class naming

- File: `API<Subnamespace><ClassUnderTest>TestCase.inc`
- Class: same as filename (without `.inc`)
- Methods that begin with `test` are auto-discovered and run

### Discovery and lifecycle

- The runner calls `setup()` once, runs every `test*` method (with config snapshot/restore between methods), then calls `teardown()`.
- Mark flaky/timing-sensitive tests with `#[TestCaseRetry(retries: 3, delay: 1)]`.
- Tests run on a real pfSense instance via `pfsense-restapi runtests` (optionally filter: `pfsense-restapi runtests <keyword>`).
- Always undo any explicit external state (e.g. files written outside `$config`) in the test or `teardown()`.

### Assertion helpers (from `Core/TestCase.inc`)

`assert_equals`, `assert_not_equals`, `assert_throws`, `assert_does_not_throw`, `assert_throws_response`, `assert_is_true`, `assert_is_false`, `assert_str_contains`, `assert_str_does_not_contain`, `assert_is_greater_than`, `assert_is_greater_than_or_equal`, `assert_is_less_than`, `assert_is_less_than_or_equal`, `assert_is_empty`, `assert_is_not_empty`, `assert_type`.

`assert_throws_response(response_id, code, callable)` is the canonical way to verify a `Response`-derived exception.

### Minimal example

```php
<?php

namespace RESTAPI\Tests;

use RESTAPI\Core\TestCase;
use RESTAPI\Models\FirewallAlias;

class APIModelsFirewallAliasTestCase extends TestCase
{
  public function test_reject_invalid_alias_name(): void
  {
    $this->assert_throws_response(
      response_id: "INVALID_FIREWALL_ALIAS_NAME",
      code: 400,
      callable: function () {
        $alias = new FirewallAlias(data: ["name" => "!!bad", "type" => "host"]);
        $alias->validate();
      },
    );
  }
}
```

See `.github/skills/writing-tests.md` for richer examples (async apply waits, pfctl assertions, fixtures).

---

## 11. Build, run, regenerate

### Build the package (FreeBSD required)

```bash
# Easiest path: VirtualBox + Vagrant on your dev machine
sh vagrant-build.sh
# Output: pfSense-pkg-RESTAPI-<version>.pkg in the repo root
```

Then `scp` the resulting `.pkg` to a pfSense test instance and `pkg add` it.

### Useful runtime CLIs (run on the pfSense host)

```bash
pfsense-restapi runtests                 # run the full test suite
pfsense-restapi runtests <keyword>       # filter by keyword
pfsense-restapi buildschemas             # regenerate OpenAPI + GraphQL artifacts
# manage.php exposes more: buildendpoints, buildforms, buildprivs, notifydispatcher, ...
```

### Local-only checks (no pfSense required)

```bash
./node_modules/.bin/prettier --check ./pfSense-pkg-RESTAPI/files   # style
phplint -vvv --no-cache                                             # syntax
black --check .                                                     # python style
pylint $(git ls-files '*.py')
./phpdoc                                                            # PHPDoc render check
```

CI runs all of the above on every push, plus a real-pfSense matrix that builds the package, installs it on a pfSense VM, generates and lints OpenAPI, and runs `pfsense-restapi runtests`.

---

## 12. Pull request workflow

Branches (see `docs/CONTRIBUTING.md` for full text):

- `next_patch` — small bug fixes, docs typos. Goes into the next patch release.
- `next_minor` — new features, enhancements, minimal-impact breaking changes.
- `next_major` — large structural changes; rarely accepted.
- `master` — reflects the current stable release. Do not target directly except for security hotfixes, dependency bumps, or release-independent docs.

Before opening a PR:

1. Format (`prettier`, `black`).
2. Run `phplint` locally if you can.
3. If you touched `Endpoints/`, `Models/`, `Fields/`, `Validators/`, `Dispatchers/`, or `Auth/`, add or update tests under `Tests/`.
4. If you added a new resource, the OpenAPI/GraphQL/privilege artifacts will regenerate on install — you do not commit them by hand.
5. Verify on a pfSense test instance whenever feasible. CI's pfSense VM matrix is the authoritative gate.

Security vulnerabilities: do **not** open a public issue or PR. Contact a maintainer privately (see `docs/index.md`).

---

## 13. AI contribution checklist

Before proposing code, confirm:

- [ ] The Endpoint is thin (URL, `model_name`, `request_method_options`, `many`, optional auth/help) and does not override method handlers.
- [ ] All schema/typing/defaults/choices/constraints are expressed via `Field` objects.
- [ ] Reusable validation lives in a `Validators/` class; only model-specific logic is in `validate_*` / `validate_extra`.
- [ ] Shell calls use `RESTAPI\Core\Command`.
- [ ] Apply work goes through a Dispatcher invoked by `apply()` with `async: $this->async`.
- [ ] All thrown errors use a `Responses/` class with a stable `UPPER_SNAKE_CASE` `response_id`.
- [ ] Sensitive Fields are marked `sensitive: true` and never logged.
- [ ] Tests added/updated under `RESTAPI/Tests/` extending `RESTAPI\Core\TestCase`.
- [ ] No pfSense Plus-only symbols, paths, or behaviors are referenced.
- [ ] PHP files end in `.inc`, start with the standard `<?php` + `namespace` + `require_once 'RESTAPI/autoloader.inc';` preamble.
- [ ] Code passes Prettier (`@prettier/plugin-php`).
- [ ] Comments use `#` style; public symbols have PHPDoc.

When in doubt, mirror the closest existing class. The codebase is intentionally repetitive so patterns stay obvious.

---
> Source: [pfrest/pfSense-pkg-RESTAPI](https://github.com/pfrest/pfSense-pkg-RESTAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
