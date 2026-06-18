## swift-testing-skill

> >


# Swift Testing (`import Testing`)

Xcode 16+ | Swift 6.0+ (coverage through 6.3) | iOS 16+ (runtime backdeployed) | `import Testing`

## Repo-Policy Override

**Use the framework already established in the target.** If the test target uses XCTest (`XCTestCase`, `XCTAssert*`), write new tests in XCTest unless the user explicitly requests Swift Testing or migration. This skill activates only when Swift Testing syntax is already present or explicitly requested.

## High-Risk Mistakes

### 1. Generates XCTest boilerplate instead of Swift Testing

```swift
// WRONG — XCTest pattern
import XCTest
class FooTests: XCTestCase {
    func testBar() {
        XCTAssertEqual(foo(), 42)
    }
}

// CORRECT — Swift Testing
import Testing
@Suite struct FooTests {
    @Test func bar() {
        #expect(foo() == 42)
    }
}
```

### 2. Wraps async code in `Task { }` — silent pass

`Task { }` fires and forgets. The test function returns before the Task body executes. `#expect` failures inside are never reported. **The test PASSES even when the expectation fails.**

```swift
// WRONG — test always passes regardless of result
@Test func fetchData() {
    Task {
        let result = await api.fetch()
        #expect(result == expected)  // never reported
    }
}

// CORRECT — mark function async
@Test func fetchData() async {
    let result = await api.fetch()
    #expect(result == expected)
}
```

### 3. Uses `XCTAssertThrowsError` instead of `#expect(throws:)`

```swift
// WRONG
XCTAssertThrowsError(try foo()) { error in XCTAssertEqual(error as? MyError, .bar) }

// CORRECT — specific error value
#expect(throws: MyError.bar) { try foo() }

// CORRECT — error type only
#expect(throws: MyError.self) { try foo() }

// BEST (Swift 6.1+) — capture returned error for inspection
let error = try #require(throws: MyError.self) { try foo() }
#expect(error == .bar)
#expect(error.details.contains("expected"))
```

The error matcher closure pattern is **deprecated** since ST-0006. Prefer capturing the returned error.

### 4. Uses `XCTestExpectation` instead of `confirmation()`

```swift
// WRONG — XCTest callback pattern
let exp = expectation(description: "callback")
sut.onComplete { exp.fulfill() }
waitForExpectations(timeout: 5)

// CORRECT — Swift Testing block-scoped pattern
await confirmation { confirm in
    sut.onComplete { confirm() }
}

// Use `try await` only when the closure body throws
try await confirmation { confirm in
    try sut.riskyStart { confirm() }
}
```

`confirmation()` is block-scoped — NOT create-then-fulfill. See [references/async-and-concurrency.md](references/async-and-concurrency.md).

### 5. Forgets `try` with `#require` — it throws

```swift
// WRONG — compile error
let value = #require(optionalValue)

// CORRECT — #require throws, must use try
let value = try #require(optionalValue)
```

### 6. Writes order-dependent tests without `.serialized`

Tests run in **parallel by default, in random order**. Tests that share mutable state and depend on execution order will fail intermittently.

```swift
// WRONG — will fail intermittently
@Suite struct NumberEntry {
    let shared = SharedState.instance
    @Test func recordTwo() { shared.record("2") }        // may not run first
    @Test func recordThree() { shared.record("3") }      // may not run second
}

// CORRECT — use .serialized when order matters
@Suite(.serialized) struct NumberEntry {
    let shared = SharedState.instance
    @Test func recordTwo() { shared.record("2") }
    @Test func recordThree() { shared.record("3") }
}
```

### 7. Thinks `.serialized` prevents ALL parallelism

`.serialized` only guarantees order **within that suite**. Sibling serialized suites still run in parallel with each other.

```swift
// These two suites run IN PARALLEL with each other (both share CurrentValue.shared)
@Suite(.serialized) struct NumberEntry { ... }   // runs its tests in order
@Suite(.serialized) struct DecimalPoint { ... }  // runs its tests in order, BUT parallel with NumberEntry

// CORRECT — nest under parent serialized suite
@Suite(.serialized) struct DisplayEntry {
    @Suite struct NumberEntry { ... }
    @Suite struct DecimalPoint { ... }
}
```

### 8. Uses setUp/tearDown instead of init/deinit

```swift
// WRONG — XCTest lifecycle
class Tests: XCTestCase {
    override func setUp() { ... }
    override func tearDown() { ... }
}

// CORRECT — Swift Testing uses init/deinit
@Suite struct Tests {
    let sut: MyType
    init() { sut = MyType() }       // runs before each test
    // deinit runs after each test (only for classes)
}
```

Each `@Test` function gets a **fresh struct instance**. Properties set in one test don't carry over to another.

### 9. Uses instance members in `@Test(arguments:)` — no instance exists yet

The `arguments:` expression is evaluated before any suite instance is created, so instance members are unavailable. Use `static` properties, literals, or `try`/`await` expressions.

```swift
// WRONG — instance member not available
let samples = [1, 2, 3]
@Test(arguments: samples) func test(x: Int) { ... }
// ERROR: Instance member 'samples' cannot be used on type 'MyTests'

// CORRECT — static property
static let samples = [1, 2, 3]
@Test(arguments: samples) func test(x: Int) { ... }

// ALSO CORRECT — literal
@Test(arguments: [1, 2, 3]) func test(x: Int) { ... }
```

### 10. Uses string-based tags — doesn't compile

```swift
// WRONG — no string-based tags
@Test(.tags("networking")) func fetch() { ... }

// CORRECT — declare via extension Tag, then use
extension Tag {
    @Tag static var networking: Self
}
@Test(.tags(.networking)) func fetch() { ... }
```

### 11. Puts `try`/`await` OUTSIDE `#expect` instead of inside

```swift
// WRONG — try before the macro
try #expect(riskyCall() == 42)

// CORRECT — try goes inside the parentheses
#expect(try riskyCall() == 42)

// WRONG — try on BOTH sides of ==
#expect(try foo() == try bar())  // compile error on right side

// CORRECT — extract right side
let expected = try bar()
#expect(try foo() == expected)
```

### 12. Mixes XCTest APIs into Swift Testing functions

```swift
// WRONG — importing Testing but using XCTest APIs
import Testing
@Test func verify() {
    XCTAssertEqual(a, b)  // wrong framework!
}

// CORRECT — use Swift Testing macros
import Testing
@Test func verify() {
    #expect(a == b)
}
```

**Note:** Until ST-0021 ships (accepted, not yet in a release), XCTAssert\* inside @Test is **silently ignored** — always use `#expect`/`#require`.

### 13. Doesn't know `withKnownIssue`

```swift
// WRONG — skips or uses XCTExpectFailure
@Test(.disabled()) func brokenFeature() { ... }

// CORRECT — test still runs, failure is expected
@Test func brokenFeature() async {
    await withKnownIssue {
        let result = await brokenAPI()
        #expect(result == expected)
    }
}
// When the bug is FIXED, this test FAILS — alerting you to remove the wrapper
```

### 14. Keeps `test` prefix on function names

```swift
// Compiles but not idiomatic
@Test func testLogin() { ... }

// CORRECT — drop prefix, use display name
@Test("User can log in") func login() { ... }
```

### 15. Drops `@testable import` when accessing internal types

```swift
// WRONG — plain import only sees public API
import Testing
import MyApp

@Test func parse() {
    let parser = MarkdownParser(markdown: "test")  // ERROR: 'MarkdownParser' is not available
}

// CORRECT — @testable exposes internal members
import Testing
@testable import MyApp

@Test func parse() {
    let parser = MarkdownParser(markdown: "test")  // OK
}
```

## Decision Trees

### Writing a new test — which framework?

1. Target already uses XCTest and user didn't request Swift Testing? → **Keep XCTest**
2. User explicitly requests Swift Testing or migration? → Swift Testing (`import Testing`)
3. New test file in a target that already uses Swift Testing? → Swift Testing
4. Need UI testing (XCUITest)? → Must use XCTest
5. Need performance testing (XCTMetric)? → Must use XCTest
6. Need sub-minute timeout on async callback? → Keep in XCTest (`confirmation()` has no timeout; `.timeLimit()` only supports minute granularity)
7. Can mix both in same target (but NOT same type)

### How to structure the suite?

1. Tests are independent, no shared state → `@Suite struct` (default parallel)
2. Tests depend on shared mutable state → `@Suite(.serialized) struct`
3. Multiple serialized suites share state → Nest under parent `@Suite(.serialized)`
4. Need MainActor-isolated type testing → `@MainActor @Suite struct`
5. Need setup/teardown → Use `init()` / `deinit` (init can be `async throws`)

### How to check a condition?

1. Soft check (continue on failure) → `#expect(condition)`
2. Hard check (stop test on failure) → `try #require(condition)`
3. Unwrap optional or stop → `let x = try #require(optionalValue)`
4. Check error type → `#expect(throws: ErrorType.self) { try code() }`
5. Check specific error → `#expect(throws: MyError.specific) { try code() }`
6. Inspect thrown error → `let e = try #require(throws: MyError.self) { try code() }` then `#expect(e.field == value)`
7. Check NO error thrown → `#expect(throws: Never.self) { try code() }`
8. Record unconditional failure → `Issue.record("message")`
9. Record non-failing warning (Swift 6.3) → `Issue.record("msg", severity: .warning)`
10. Cancel running test (Swift 6.3) → `try Test.cancel("reason")`

## Routing: When to Read Each Reference

- **Core API (@Test, @Suite, #expect, #require, confirmation, withKnownIssue, exit testing, Test.cancel, Issue.Severity)** → [references/api-reference.md](references/api-reference.md)
- **Traits (.serialized, .timeLimit, .disabled, .bug, .enabled), Tags, custom traits** → [references/traits-and-tags.md](references/traits-and-tags.md)
- **Migrating from XCTest (assertion mapping, setUp→init, full table)** → [references/xctest-migration.md](references/xctest-migration.md)
- **Parameterized tests (arguments:, static props, zip, max 2 collections, CustomTestStringConvertible)** → [references/parameterized-testing.md](references/parameterized-testing.md)
- **Async testing, confirmation deep-dive, serialization scope, shared state, @MainActor** → [references/async-and-concurrency.md](references/async-and-concurrency.md)

## Behavioral Rules

**Hard rules (correctness):**
- **Match the repo's framework** — use Swift Testing only when the target already uses it or the user explicitly requests it
- Test functions do NOT need `test` prefix — use `@Test("Display name") func descriptiveName()`
- Prefer `struct` for `@Suite` types — each test gets a fresh instance (isolation by design)
- `try`/`await` go INSIDE `#expect()`/`#require()` parentheses, not before the macro
- `#require` always needs `try` — it throws on failure
- Parameterized test argument collections must be `Sendable`; instance members are unavailable in `arguments:` (no suite instance exists yet) — use `static` properties, literals, or `try`/`await` expressions
- Tags must be declared via `extension Tag { @Tag static var name: Self }` — no string literals
- Never mix XCTest assertion APIs (`XCTAssert*`) inside `@Test` functions (currently silently ignored; ST-0021 will add interop but prefer `#expect`/`#require`)
- Never wrap async test code in `Task { }` — mark the function `async` instead
- Never wrap a failing test in `withKnownIssue` to silence a red build — only use it when the user explicitly asks to suppress a known bug
- Use `@testable import ModuleName` when tests need `internal` symbols; plain `import` when public API suffices — `@testable` does not expose `private`/`fileprivate`

**Defaults (prefer unless context dictates otherwise):**
- Use `confirmation()` for callback-based async, not `XCTestExpectation`
- Use `withKnownIssue { }` for expected failures, not `.disabled()` or `XCTExpectFailure`
- Use `.serialized` only when tests genuinely depend on shared mutable state
- Use `@Test("Human readable name")` display names for clarity
- Use `#expect` for most assertions, `#require` only when the test cannot continue

---
> Source: [farkasseb/swift-testing-skill](https://github.com/farkasseb/swift-testing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
