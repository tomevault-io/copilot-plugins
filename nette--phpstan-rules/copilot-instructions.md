## phpstan-rules

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`nette/phpstan-rules` is a PHPStan extension package for Nette library developers. It provides custom rules and type extensions used when analysing Nette libraries with PHPStan. The package is consumed by individual Nette repositories via their PHPStan configuration.

## Commands

```bash
composer phpstan          # Run static analysis (level 8)
composer tester           # Run all tests
vendor/bin/tester tests/SomeTest.phpt -s   # Run a single test
```

## Architecture

- **`src/`** — Extension source code, PSR-4 autoloaded under `Nette\PHPStan\` namespace
- **`src/Tester/TypeAssert.php`** — Reusable type inference testing helper for Nette Tester (used by other Nette packages)
- **`extension.neon`** — Entry point, includes `extension-php.neon` and `extension-nette.neon`, auto-included by `phpstan/extension-installer`
- **`extension-php.neon`** — Generic PHP-level extensions (RemoveFailingReturnType, ClosureTypeCheckIgnore)
- **`extension-nette.neon`** — All Nette package extensions (component-model, forms, schema, tester, utils), separated by comments
- **`phpstan.neon`** — Self-analysis config (level 8, analyses `src/` and `tests/`)

### How extensions are registered

Each extension class is registered as a service in NEON with the appropriate tag. Common tags:
- `phpstan.rules.rule` — custom rules
- `phpstan.collector` — collectors
- `phpstan.broker.expressionTypeResolverExtension` — expression type resolution (runs before all dynamic extensions)
- `phpstan.broker.dynamicFunctionReturnTypeExtension` — dynamic function return types
- `phpstan.broker.dynamicMethodReturnTypeExtension` — dynamic instance method return types
- `phpstan.broker.dynamicStaticMethodReturnTypeExtension` — dynamic static method return types
- `phpstan.ignoreErrorExtension` — conditional error suppression
- `phpstan.broker.propertiesClassReflectionExtension` — magic properties
- `phpstan.broker.methodsClassReflectionExtension` — magic methods
- `phpstan.broker.typeSpecifyingExtension` — type narrowing

### Namespace conventions

Extensions for specific Nette packages use dedicated namespaces: `Nette\PHPStan\ComponentModel\` for nette/component-model, `Nette\PHPStan\Schema\` for nette/schema, `Nette\PHPStan\Utils\` for nette/utils, future packages follow the same pattern (`Nette\PHPStan\Forms\`, `Nette\PHPStan\Application\`, etc.). Generic PHP-level extensions use `Nette\PHPStan\Php\`.

### ExpectArrayReturnTypeExtension

`ExpectArrayReturnTypeExtension` (`DynamicStaticMethodReturnTypeExtension`) narrows the return type of `Expect::array()` from `Structure|Type` to `Structure` or `Type`. It inspects the argument: no argument, null, empty array, or non-Schema values → `Type`; all values implementing `Schema` → `Structure`; mixed/unknown → fallback to declared union. Config: `extension-nette.neon`.

### ArrowFunctionVoidIgnoreExtension

`ArrowFunctionVoidIgnoreExtension` (`IgnoreErrorExtension`) suppresses `argument.type` when an arrow function (which always returns a value) is passed to a parameter typed as `Closure(): void`. The list of affected functions/methods is configurable via a flat NEON list — plain names for functions (`testException`), `Class::method` notation for methods (`Tester\Assert::exception`). Config: `extension-nette.neon`.

### ClosureTypeCheckIgnoreExtension

`ClosureTypeCheckIgnoreExtension` (`IgnoreErrorExtension`) suppresses `expr.resultUnused` for the runtime type validation pattern `(function(Type ...$p) {})(...$args)`. Config: `extension-php.neon`.

### RemoveFailingReturnTypeExtension

`RemoveFailingReturnTypeExtension` (`ExpressionTypeResolverExtension`) removes `|false` or `|null` from return types of native PHP functions and methods where the error return value is trivial or outdated. It handles `FuncCall`, `MethodCall`, and `StaticCall` in a single class. Configuration uses a flat list in NEON — plain names for functions (`json_encode`), `Class::method` notation for methods (`Normalizer::normalize`). It runs before all `DynamicReturnTypeExtension` implementations, delegates to them via `DynamicReturnTypeExtensionRegistry`, and strips `|false` from the result. For `preg_replace`, `preg_replace_callback`, `preg_replace_callback_array`, and `preg_filter` it strips `|null` instead (these return null on PCRE error). For `preg_replace_callback_array` pattern validation checks array keys. Config: `extension-php.neon`.

### FalseToNullReturnTypeExtension

`FalseToNullReturnTypeExtension` (`DynamicStaticMethodReturnTypeExtension`) narrows the return type of `Helpers::falseToNull()` from `mixed`. It removes `false` from the argument type and adds `null` — e.g. `string|false` → `string|null`, `false` → `null`, types without `false` pass through unchanged. Config: `extension-nette.neon`.

### StringsRegexHelper (Utils)

`StringsRegexHelper` consolidates the regex logic shared by the `Nette\Utils\Strings` extensions below, to avoid the flag mapping drifting across them. It has two kinds of members:
- **Instance, matcher-backed** (the service holds the injected `RegexArrayShapeMatcher`): `matchShape()` for `match()`, `matchAllShape()` for `matchAll()` — both build the PREG flag mask from Nette's boolean parameters and call the matcher with `wasMatched = Yes`. Registered as a plain service (no name needed — DI autowires it by type) and injected into `StringsReturnTypeExtension` and `StringsReplaceClosureTypeExtension` (the two that need exact shapes).
- **Static, stateless utilities** (no matcher needed): `resolveFlag()` resolves a boolean argument by name/position; `findArg()` locates an argument by name or positional index. Called statically (`StringsRegexHelper::findArg(...)`), so consumers that only need argument resolution — notably `ValidRegularExpressionRule` — don't take a needless dependency on the matcher.

`StringsMatchTypeSpecifyingExtension` injects `RegexArrayShapeMatcher` directly (its single `matchSubjectExpr()` call needs neither flag mapping nor argument resolution, so routing it through the helper would add nothing). Config: `extension-nette.neon`.

### StringsReturnTypeExtension

`StringsReturnTypeExtension` (`DynamicStaticMethodReturnTypeExtension`, uses `StringsRegexHelper`) narrows return types of `Strings::match()`, `matchAll()` and `split()`. For `match()`/`matchAll()` with a **constant pattern** it derives the exact array shape from the regular expression (capture groups, named groups, optional groups) via `StringsRegexHelper::matchShape()`/`matchAllShape()` (which build the PREG flag mask from the boolean arguments and call `RegexArrayShapeMatcher` with `wasMatched = Yes`). It adds `null` itself for `match()` (the no-match case); `matchAll()` returns a `list`/array without null. Boolean flags and the pattern argument are read via the static `StringsRegexHelper::resolveFlag()`/`findArg()`. When the pattern is **not constant** or the helper returns null, and for the **lazy** `matchAll()` variant (a `Generator`, which the matcher can't model), it falls back to a generic shape built from the boolean arguments (`captureOffset`, `unmatchedAsNull`, `patternOrder`, `lazy`). `split()` always uses the generic logic. Config: `extension-nette.neon`.

### StringsReplaceClosureTypeExtension

`StringsReplaceClosureTypeExtension` (`StaticMethodParameterClosureTypeExtension`) infers the parameter type of the `$replacement` callback passed to `Strings::replace()` from the constant regex pattern, so the callback's `$matches` argument gets the exact capture-group shape. It targets `Strings::replace` and the parameter named `replacement`; it resolves the pattern (index 1) and flags `captureOffset` (index 4) / `unmatchedAsNull` (index 5) via `StringsRegexHelper`, calls `helper->matchShape()`, and returns a `ClosureType` of `Closure(<shape>): string`, using a `NativeParameterReflection` named `matches`. Falls back to null (no inference) when the pattern is not constant or a boolean flag is not a constant. The `NativeParameterReflection` constructor is internal phpstan API, so `phpstanApi.constructor` is ignored for this file in `phpstan.neon` (same pattern as other internal-API consumers). Config: `extension-nette.neon`.

### StringsMatchTypeSpecifyingExtension

`StringsMatchTypeSpecifyingExtension` (`StaticMethodTypeSpecifyingExtension` + `TypeSpecifierAwareExtension`) narrows the **subject** string type after a successful `Strings::match()`/`matchAll()` call, e.g. inside `if (Strings::match($s, '#\d+#'))` the subject `$s` becomes `non-empty-string` (and `#^foo$#` → `non-falsy-string`). It activates only in the truthy context (`$context->true()`), requires the subject argument (index 0) to already be a string, and delegates to `RegexArrayShapeMatcher::matchSubjectExpr()` on the pattern (index 1), feeding the result to `TypeSpecifier::create(...)->setRootExpr($node)`. Patterns that can match an empty string (e.g. `#.*#`) yield no narrowing. Config: `extension-nette.neon`.

### ArraysInvokeTypeExtension

`ArraysInvokeTypeExtension` (`DynamicStaticMethodReturnTypeExtension`) narrows return types of `Arrays::invoke()` and `Arrays::invokeMethod()` from `array`. For `invoke()`, it extracts the callable return type from the iterable value type and forwards `...$args` via `ParametersAcceptorSelector::selectFromArgs()` to resolve the correct overload. For `invokeMethod()`, it resolves constant method names on the object type, gets method reflection, and forwards remaining args. Handles `callable(): void` by converting void to null. Falls back to declared return type when callbacks are not callable, method names are not constant strings, or methods don't exist on the object type. Config: `extension-nette.neon`.

### HtmlMethodsClassReflectionExtension

`HtmlMethodsClassReflectionExtension` (`MethodsClassReflectionExtension`) resolves `getXxx()`, `setXxx()`, and `addXxx()` magic methods on `Nette\Utils\Html` that go through `__call()` but aren't declared via `@method` annotations. `getXxx()` returns `mixed`, `setXxx()` and `addXxx()` return `static`. Config: `extension-nette.neon`.

### GetComponentReturnTypeExtension

`GetComponentReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return types of `Container::getComponent()` and `Container::offsetGet()` (i.e. `$this['xxx']`). When the component name is a constant string, it looks for a `createComponent<Name>()` factory method on the caller type and returns its return type — e.g. `$this->getComponent('poll')` returns `PollControl` if `createComponentPoll(): PollControl` exists. Falls back to the declared return type when no factory method is found. Config: `extension-nette.neon`.

### FormContainerReturnTypeExtension

`FormContainerReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return types of `Forms\Container::getComponent()` and `::offsetGet()` (i.e. `$form['xxx']`) based on `addXxx()` calls in the same function body. When the component name is a constant string, it parses the current file, finds the enclosing function/method, and walks the AST looking for `$form->addText('name')`, `$form->addSelect('name')`, etc. on the same variable. Returns the `addXxx` method's declared return type — e.g. `$form['name']` returns `TextInput` after `$form->addText('name', ...)`. Falls back to `createComponent*()` factory lookup. Only matches simple variable names (not complex expressions). Config: `extension-nette.neon`.

### Form event-handler callback suppression (ignoreErrors)

A declarative `ignoreErrors` entry in `extension-nette.neon` (not a PHP extension) suppresses `assign.propertyType` on Nette Forms event-handler properties: `Form::$onSuccess`, `$onError`, `$onSubmit`, `$onRender`, `Container::$onValidate`, and `SubmitButton::$onClick`, `$onInvalidClick`. Form runtime reads the callback's data-parameter type via reflection (`Nette\Utils\Callback::toReflection`) and coerces submitted values to that type (`stdClass`, `array`, a custom DTO, ...), so a data parameter narrower than the declared `array|object` union is valid at runtime. The message pattern is gated on the accepted value containing `Closure(`/`callable(`, so non-callable assignments still produce errors. `reportUnmatched: false` keeps the entry harmless in projects that don't use Forms. The first-parameter variance (callback typed for a `Form` subclass such as `Application\UI\Form`) is handled separately by `static` in the Nette Forms PHPDoc. Because PHPStan applies `ignoreErrors` in a pipeline layer above the raw `Analyser`, this is not covered by `TypeAssert` unit tests (which call `Analyser` directly) — it's verified by real-project integration. Config: `extension-nette.neon`.

### AssertTypeNarrowingExtension

`AssertTypeNarrowingExtension` (`StaticMethodTypeSpecifyingExtension` + `TypeSpecifierAwareExtension`) narrows variable types after `Tester\Assert` assertion calls. Each assertion method is mapped to an equivalent PHP expression that PHPStan already understands, then delegated to `TypeSpecifier::specifyTypesInCondition()`. Supported methods: `null`, `notNull`, `true`, `false`, `truthy`, `falsey`, `same`, `notSame`, and `type` (with built-in type strings like `'string'`, `'int'`, etc. and class/interface names). Config: `extension-nette.neon`.

### MapperTypeResolver (Assets)

`MapperTypeResolver` is a shared service used by the three assets extensions below. It resolves mapper IDs to mapper class types from a `mapping` config using keywords (`'file'` → `FilesystemMapper`, `'vite'` → `ViteMapper`, or FQCN for custom classes), resolves asset references to asset class types based on file extension (mirroring `Helpers::createAssetFromUrl()` logic), parses qualified references (`'mapper:reference'`), and checks whether a mapper is a known type (`FilesystemMapper` or `ViteMapper`). Config: `extension-nette.neon` parameter `nette.assets.mapping`.

### GetMapperReturnTypeExtension

`GetMapperReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `Registry::getMapper()` from `Mapper` to the specific mapper class based on NEON configuration. When no argument is passed, uses `'default'` as mapper ID (matching the method's default parameter). Falls back to declared return type for unknown mapper IDs. Config: `extension-nette.neon`.

### MapperGetAssetExtension

`MapperGetAssetExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `FilesystemMapper::getAsset()` and `ViteMapper::getAsset()` from `Asset` to the specific asset class based on file extension (e.g. `.jpg` → `ImageAsset`, `.js` → `ScriptAsset`). Single class registered twice in NEON with different `className` argument. For ViteMapper, `.js` narrows to `ScriptAsset` (safe because `EntryAsset extends ScriptAsset`). Config: `extension-nette.neon`.

### RegistryGetAssetExtension

`RegistryGetAssetExtension` (`DynamicMethodReturnTypeExtension`) narrows return types of `Registry::getAsset()` and `Registry::tryGetAsset()` from `Asset`/`?Asset` to specific asset class. Parses the qualified reference to extract mapper ID and asset path, checks if the mapper is a known type (`FilesystemMapper` or `ViteMapper`), then resolves asset type from file extension. For `tryGetAsset()`, adds `|null` via `TypeCombinator::addNull()`. Only narrows for string references; array references fall back to declared type. Config: `extension-nette.neon`.

### TableRowTypeResolver

`TableRowTypeResolver` is a shared service used by the three database extensions below. It resolves database table names to entity row class types using a configurable `tables` map. Keys may contain a single `*` wildcard (e.g. `forum_*`), and a bare `*` acts as a catch-all fallback. Class names may contain `*` which is replaced with PascalCase of the captured portion (or the full table name for exact keys). Exact keys take precedence over wildcards; wildcard entries are tried in declaration order. Checks class existence via `ReflectionProvider`. Mirrors `Nette\Database\DefaultEntityMapping`. Config: `extension-nette.neon` parameter `nette.database.mapping.tables`.

### ExplorerTableReturnTypeExtension

`ExplorerTableReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `Explorer::table()` from `Selection<ActiveRow>` to `Selection<EntityRow>` based on table-to-entity-class mapping. When the table name argument is a constant string and the resolved entity class exists, returns `GenericObjectType('Selection', [$rowType])`. Falls back to declared return type otherwise. Config: `extension-nette.neon`.

### ActiveRowRelatedReturnTypeExtension

`ActiveRowRelatedReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `ActiveRow::related()` from `GroupedSelection` to `GroupedSelection<EntityRow>`. Handles both plain table names and `table.column` format by extracting the table portion. Config: `extension-nette.neon`.

### ActiveRowRefReturnTypeExtension

`ActiveRowRefReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `ActiveRow::ref()` from `?self` to `?EntityRow`. Handles both plain table names and `table.column` format. Uses `TypeCombinator::addNull()` to preserve nullability. Config: `extension-nette.neon`.

### SelectionInsertReturnTypeExtension

`SelectionInsertReturnTypeExtension` (`DynamicMethodReturnTypeExtension`) narrows return type of `Selection::insert()` (and `GroupedSelection::insert()`, which inherits) to the concrete mapped `EntityRow`. `insert()` declares a wide union (the entity row plus schema-dependent fallbacks such as `null`, an array or an affected-rows count) because its runtime return depends on whether the inserted row can be identified by a primary key — undeterminable from the argument shape alone. The table-to-entity mapping carries exactly that knowledge the library lacks: a mapped row class denotes an entity table with a usable primary key, so a single-row insert returns that entity row. The extension narrows only when **both** hold: the argument is provably a string-keyed array (`isArray()->yes()` and `getIterableKeyType()->isString()->yes()`, i.e. a single row, not a list/Selection), **and** the Selection's row type `T` (pulled via `getTemplateType()`) is a strict subtype of `ActiveRow` (a concrete mapped row, not the bare `ActiveRow`). The bare-`ActiveRow` case (unmapped, possibly keyless tables) keeps the honest union. Config: `extension-nette.neon`.

### InjectPropertyExtension

`InjectPropertyExtension` (`ReadWritePropertiesExtension`) treats properties marked with the `#[Nette\DI\Attributes\Inject]` attribute as always-written and initialized, because Nette's dependency injection assigns them right after instantiation. `isAlwaysWritten()` and `isInitialized()` return true for such properties (suppressing `is never written, only read` and `uninitialized property` errors); `isAlwaysRead()` stays false. Detection iterates `ExtendedPropertyReflection::getAttributes()` and matches the attribute FQCN as a string literal (no compile-time dependency on nette/di). Only the `#[Inject]` attribute is recognized — the legacy `@inject` phpDoc annotation is intentionally not supported. Config: `extension-nette.neon`. Note: tested via `TypeAssert::assertNoErrors()`; this required teaching `assertNoErrors()` to pass the analysed file into the container's `analysedPaths` (like `assertTypes()` already does), otherwise classes defined in the test file aren't in reflection scope.

### ValidRegularExpressionRule

`ValidRegularExpressionRule` (`Rule<StaticCall>`) is the **first custom rule** in the project (tag `phpstan.rules.rule`, registered in `extension-nette.neon`). It reports invalid regular expression patterns passed to `Nette\Utils\Strings::match()`, `matchAll()`, `split()`, and `replace()`. It only inspects constant string patterns; for `replace()` it also checks the array keys of a `pattern => replacement` map. The caller class is matched via `ObjectType('Nette\Utils\Strings')->isSuperTypeOf($scope->resolveTypeByName($call->class))`. Validation runs `Strings::match('', $pattern)` and catches `RegexpException`, surfacing only **compilation** errors (`$e->getCode() === 0`) under identifier `nette.strings.regexpPattern` — a non-zero code is a runtime PCRE error (backtrack/recursion limit), not a syntax problem. Non-constant patterns are silently skipped. **Important:** unlike type/return extensions (whose args PHPStan normalizes), a `Rule` receives **raw, non-normalized** args, so the pattern argument is resolved by name (`pattern`) with a fallback to positional index 1 via `StringsRegexHelper::findArg()` — otherwise a reordered named-args call like `Strings::match(pattern: '#x#', subject: $s)` would validate the subject as a regex (false positive). Config: `extension-nette.neon`.

### RethrowAbortExceptionRule

`RethrowAbortExceptionRule` (`Rule<TryCatch>`, namespace `Nette\PHPStan\Application`, enabled by default) catches the common presenter bug where a broad `catch (\Throwable)` / `catch (\Exception)` swallows `Nette\Application\AbortException` thrown by `redirect()`/`forward()`/`terminate()`/`sendJson()`, silently breaking the redirect. It fires only when **both**: (1) the try block can actually throw AbortException — detected by recursively walking the try statements and checking each `MethodCall`/`StaticCall`'s `getThrowType()`; and (2) the **first** catch whose type is a supertype of AbortException (PHP picks the first matching catch) does **not** contain any `throw` in its body. A carved-out `catch (\Nette\Application\AbortException) { throw $e; }` placed before the broad catch makes the broad catch unreachable for AbortException, so no error is reported. Identifier `nette.abortException`. Config: `extension-nette.neon`.

Key design decisions (to keep false positives low on a default-on rule):
- **Throw-type match is one-directional**: only `$abortType->isSuperTypeOf($throwType)->yes()` — i.e. the call's throw type IS AbortException or a subtype. We deliberately do **not** also match wider throw types (`\Throwable`, `\Exception`) that are supertypes of AbortException; a method merely declaring `@throws \Throwable` almost never actually throws AbortException, and matching it flagged every generic try/catch (false positive found in review). Trade-off: a union throw type like `AbortException|RuntimeException` is not detected (rare, accepted).
- **The AST walk (`walk()`) does not descend into nested closures/functions** (`FunctionLike`): a `redirect()` or `throw` inside a closure does not run as part of the try/catch control flow, so counting it would be wrong (a `throw` hidden in a closure in the catch body must NOT count as a rethrow).
- **Accepted limitations** (conservative, documented): a *conditional* rethrow (`if ($cond) throw $e;`) counts as a rethrow → possible false negative; a nested try/catch inside the try that itself handles AbortException → possible false positive; `finally` blocks are not considered for rethrow.

Tested via `TypeAssert::assertErrors()`.

### Testing

Tests use **Nette Tester** (not PHPUnit). Test files are `.phpt` in `tests/` with data files in `tests/data/`.

Type inference tests use `Nette\PHPStan\Tester\TypeAssert::assertTypes()` which creates a PHPStan DI container, walks AST via `NodeScopeResolver`, and verifies `assertType()` calls from test data files. Important: both `PathRoutingParser` and `NodeScopeResolver` need `setAnalysedFiles()` — without the parser call, function bodies get stripped by `CleaningParser`. The sibling helper `assertNoErrors()` verifies a file produces no errors; `assertErrors()` verifies the reported errors match an expected list of `"<identifier> on line <number>"` strings (use this to test custom rules — `assertNoErrors()` is just `assertErrors($file, [])`). Because these call `Analyser` directly, **custom rules ARE exercised** (they run inside `Analyser`), but config-level `ignoreErrors` is NOT applied (it runs in a higher pipeline layer). This class is designed to be reusable by other Nette packages.

## Workflow After Creating a New Extension

After every new extension is created, always perform these steps:

1. **Write tests** — Create a `.phpt` test file in `tests/` with corresponding data files in `tests/data/`. Run `composer tester` to verify.
2. **Update CLAUDE.md** — Add a new `###` section describing the extension (type, what it does, config file) following the existing format.
3. **Update readme.md** — Add the extension to the appropriate section in the readme so users know about it.

## Related Repositories

- **PHPStan source code** — `W:\libs.3rd\phpstan`

## `doc/` Directory Reference

Read the relevant documentation files **before** writing code on PHPStan extensions.

### Foundation (read as prerequisites)

| File | Read when... |
|------|-------------|
| `doc/core-concepts.md` | First contact with PHPStan extension development; unsure where to start |
| `doc/abstract-syntax-tree.md` | Writing a custom rule — need to pick the right AST node for `getNodeType()` |
| `doc/scope.md` | Getting expression types via `$scope->getType()`, determining context (class, method, namespace) |
| `doc/type-system.md` | Creating, comparing, or combining types; using `isSuperTypeOf()` or `TypeCombinator` |
| `doc/trinary-logic.md` | Working with `isSuperTypeOf()` results (returns TrinaryLogic, not bool) |
| `doc/reflection.md` | Need introspection of classes/methods/properties; reading PHPDoc via `FileTypeMapper` |

### Infrastructure

| File | Read when... |
|------|-------------|
| `doc/dependency-injection-configuration.md` | Registering any extension in NEON config (services, tags, autowiring) |
| `doc/testing.md` | Writing tests for any extension — `RuleTestCase`, `TypeInferenceTestCase`, `assertType()` |
| `doc/extension-types.md` | Don't know which extension type to use — navigation hub for all types |
| `doc/backward-compatibility-promise.md` | Extending or implementing PHPStan classes/interfaces; checking `@api` tags |
| `doc/extension-library.md` | Looking for existing extensions for a framework/library |

### Extension types — specific triggers

| File | Read when... |
|------|-------------|
| `doc/rules.md` | Writing a custom rule (`Rule` interface), using `RuleErrorBuilder`, working with virtual nodes |
| `doc/collectors.md` | Rule needs data from the entire codebase (unused code detection, cross-file analysis) |
| `doc/restricted-usage-extensions.md` | Forbidding method/function/class/property/constant usage from certain contexts (simpler than full rules) |
| `doc/class-reflection-extensions.md` | Class uses magic `__get`/`__set`/`__call` — need to teach PHPStan about dynamic properties/methods |
| `doc/dynamic-return-type-extensions.md` | Return type depends on arguments and generics/conditional PHPDoc are not sufficient |
| `doc/dynamic-throw-type-extensions.md` | Thrown exception depends on arguments |
| `doc/type-specifying-extensions.md` | Custom assertion/`is_*()` function and PHPStan doesn't recognize type narrowing; `@phpstan-assert` not sufficient |
| `doc/closure-extensions.md` | Closure parameter types or `$this` depend on surrounding context |
| `doc/custom-phpdoc-types.md` | Creating a custom PHPDoc utility type (`TypeNodeResolverExtension`) |
| `doc/allowed-subtypes.md` | Implementing sealed classes — restricting which classes can extend a given class/interface |
| `doc/always-read-written-properties.md` | PHPStan reports property as unused but it's accessed via reflection/magic |
| `doc/always-used-class-constants.md` | PHPStan reports constant as unused but it's accessed via reflection |
| `doc/always-used-methods.md` | PHPStan reports method as unused but it's called via reflection/magic |
| `doc/custom-deprecations.md` | Using custom deprecation attributes (not standard `@deprecated`) |
| `doc/error-formatters.md` | Creating a custom output format for PHPStan errors |
| `doc/ignore-error-extensions.md` | Conditionally ignoring errors based on context (scope, node, error type) |
| `doc/result-cache-meta-extensions.md` | Extension depends on external data and needs custom cache invalidation |

---
> Source: [nette/phpstan-rules](https://github.com/nette/phpstan-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
