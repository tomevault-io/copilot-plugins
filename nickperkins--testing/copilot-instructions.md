## testing

> |


# When to use this skill

Use this skill when you need to:

- Write new tests (Unit, Integration/Feature, or E2E) for any feature or bug fix
- Review existing tests for quality, placement, or value
- Decide whether a test belongs at Unit, Integration, or E2E level
- Refactor tests that are fragile, duplicated, or misplaced
- Design a testing strategy for a new project or module

# When NOT to use this skill

- For non-test code changes that don't need test coverage decisions

---

# The Testing Pyramid (Martin Fowler)

```
         /‾‾‾‾\          E2E: very few, real browser/environment
        /  few  \         Test user JOURNEYS that require real UI/runtime
       /‾‾‾‾‾‾‾‾\
      / moderate  \       Integration/Feature: moderate count, real deps
     / integration \      Test API flows, DB interactions, framework features
    /‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
   /     many fast    \   Unit: many, fast, isolated
  / pure logic & rules \  Test functions, services, helpers — NO external deps
 /________________________\
```

**The pyramid is a COST model.** Each layer up costs more to run, more to maintain, and more to debug when broken. Push tests DOWN the pyramid as far as possible.

---

# Decision Flowchart

When writing a new test, ask these questions in order:

1. **Does it test pure logic with no framework/DB/external dependency?** → Unit test
2. **Does it test an API endpoint, DB interaction, or framework feature?** → Integration/Feature test
3. **Is it already covered by a lower-level test?** → Don't write it again at any level
4. **Does it REQUIRE a real UI/runtime (JS must execute, multi-page state, visual a11y)?** → E2E test
5. **Could you test it with a direct API/function call?** → Integration/Feature test, not E2E

**Default to the lowest layer.** The burden of proof is on E2E: you must not be able to achieve the same confidence at a lower level.

---

# Before Writing Any Test — Adapt to the Project

Testing tools and conventions vary by project. Before writing tests:

1. **Identify the test framework and runner** — e.g., Jest/Vitest (JS), pytest (Python), Pest/PHPUnit (PHP), JUnit (Java), Go `testing`, RSpec (Ruby), etc.
2. **Find existing test files** — look in common locations: `tests/`, `test/`, `spec/`, `__tests__/`, `*_test.*`, `*.test.*`, `*_spec.*`
3. **Mirror existing patterns** — naming, structure, setup/teardown, fixture usage, assertion style
4. **Check project config** — `package.json` scripts, `phpunit.xml`, `pytest.ini`, `Makefile`, CI config for how tests run
5. **If the project has no tests yet** — set up a sensible default structure following the pyramid and conventions of the language/framework

---

# Layer 1 — Unit Tests

**Purpose:** Test pure business logic in isolation. Fast. No framework boot. No database. No external services. No file system.

**MUST test:**

- Pure functions (calculations, transformations, parsing, formatting)
- Business rules and decision logic (eligibility checks, status transitions, validations)
- Data structures (value objects, enums, custom types)
- Utility/helper functions
- Model/domain logic that doesn't require persistence (computed fields, derived values)

**MUST NOT test:**

- Database CRUD operations (that's Integration level)
- HTTP request/response cycles (that's Integration level)
- Framework internals (the framework already tests itself)
- Configuration values (config files ARE the assertion)
- Language/runtime features (e.g., "class exists", "type casting works")

**How to verify placement:** If your unit test needs a database connection, makes HTTP calls, reads files, or boots the full framework, it is NOT a unit test. Move it to Integration.

**Anti-patterns to REJECT:**

```
BAD: Testing framework internals
- Asserting a model's "fillable fields" or "column types"
- Asserting that type casting works
- Asserting that a class exists
- Asserting that a method is defined

BAD: Testing through external dependencies
- Creating DB records to test a pure calculation
- Calling a real API to test response parsing
```

**Correct unit test patterns:**

```
GOOD: Test the business logic directly
- Given inputs X and Y, function returns Z
- Given edge case input, function handles it correctly
- Given invalid input, function throws expected error

GOOD: Test query/command building without executing
- Verify the query builder produces correct SQL
- Verify the command object has correct parameters

GOOD: Test state transitions
- Given status A + event B, new status is C
- Given precondition not met, transition is rejected
```

---

# Layer 2 — Integration/Feature Tests

**Purpose:** Test how components work together. HTTP request/response cycles, database interactions, middleware, authentication/authorization, service integration, API contracts.

**MUST test:**

- API endpoints / controller actions (correct status codes, response shapes, side effects)
- Authorization (who can access what, who is forbidden, who is redirected)
- Input validation (bad input is rejected, good input is accepted)
- Database interactions (creates, updates, deletes produce correct DB state)
- Service integration (calling a service produces correct side effects — emails sent, events dispatched, caches invalidated)
- Multi-tenancy or scoping (tenant A cannot see tenant B's data)
- Error handling (graceful failure modes, correct error responses)

**MUST NOT test:**

- Implementation details of views/templates (exact HTML strings, CSS classes)
- Framework internals (that the mailer/queue/cache system works)
- That third-party libraries work (test YOUR integration code, not their internals)

**Anti-patterns to REJECT:**

```
BAD: Asserting exact HTML/template output
- Couples tests to template formatting/whitespace
- Breaks on cosmetic changes

BAD: Testing that infrastructure works
- "Email provider can send emails" → that's the provider's test
- "Database can store records" → that's the DB's test

BAD: Testing class existence or structure
- "Notification class exists" → autoloading/import resolution handles this
```

**Correct integration test patterns:**

```
GOOD: Test the API/HTTP contract
- Given valid input + authenticated user → returns 201 with expected resource
- Given missing auth → returns 401/403
- Given invalid input → returns 422 with error details

GOOD: Test authorization boundaries
- Admin can access admin endpoints
- Regular user gets 403
- Unauthenticated user gets redirected to login

GOOD: Test side effects through fakes/mocks
- Service call triggers notification (via fake notification sender)
- Service call enqueues job (via fake job queue)
- Service call writes to DB (verify DB state directly)
```

---

# Layer 2.5 — Component Tests (Special Case)

**Component tests are an exception to the "don't assert HTML structure" rule.**

When a component has **variants**, **props**, or **visual contracts**, you MUST test that they render correctly by asserting the CSS classes and HTML structure. The component's contract IS the HTML it produces.

**When to write component tests:**

- Component has variants (e.g., `x-btn variant="primary|outline|ghost"`)
- Component renders conditionally based on props (e.g., `x-card :sold-out="true"` shows overlay)
- Component has accessibility features that depend on HTML structure (e.g., `role="radiogroup"`, `aria-disabled`, `aria-current`)
- Component has complex slots or conditional slots (e.g., `x-card` with named slots)

**What to assert in component tests:**

```
✅ GOOD — Assert the component's contract
- x-btn variant="primary" renders with primary class and correct shadow
- x-btn :loading="true" renders spinner and sets aria-disabled
- x-card :sold-out="true" renders overlay element
- x-variant-selector unavailable option has strikethrough AND reduced opacity
- x-nav renders skip-to-content link as first focusable element
- x-breadcrumb last item has aria-current="page"
```

**What NOT to assert in component tests:**

```
❌ BAD — Implementation details unrelated to the contract
- The exact pixel size of padding (that's a design token, not a test)
- The exact shadow offset (test via class, not raw px values)
- The internal element count or nesting structure
- The specific font family or weight used
```

**Component test structure (Laravel Blade):**

Use the `$this->blade()` test helper with `InteractsWithViews` trait:

```php
<?php

namespace Tests\Components;

use Tests\TestCase;

class ButtonComponentTest extends TestCase
{
    /** @test */
    public function primary_variant_renders_with_correct_class()
    {
        $view = $this->blade(
            '<x-btn variant="primary">Submit</x-btn>'
        );
        
        $view->assertSee('class="btn btn--primary"', false);
    }
    
    /** @test */
    public function loading_state_renders_spinner_and_sets_aria_disabled()
    {
        $view = $this->blade(
            '<x-btn :loading="true">Submit</x-btn>'
        );
        
        $view->assertSee('aria-disabled="true"', false);
        $view->assertSee('spinner'); // or however the spinner is identified
    }
    
    /** @test */
    public function href_prop_renders_as_anchor_not_button()
    {
        $view = $this->blade(
            '<x-btn href="/shop">Browse</x-btn>'
        );
        
        $view->assertSee('<a', false);
        $view->assertDontSee('<button', false);
    }
}
```

**The distinction from Feature tests:**

- **Component test**: Assert that `x-btn variant="primary"` produces `class="btn btn--primary"`
- **Feature test**: Assert that a page containing that button works as a user expects, WITHOUT asserting the class name directly

In a Feature test for the checkout page, you'd assert: `$response->assertSee('Submit')` — not `$response->assertSee('class="btn--primary"')`.

This keeps Feature tests resilient to cosmetic changes while ensuring components themselves work correctly.

---

# Layer 3 — E2E Tests

## The Core Principle: Test User Journeys, Not Features

**E2E tests answer: "Can a real user accomplish a meaningful goal?"**

They do NOT answer: "Does this endpoint work?" or "Does this component render?" Those are lower-level questions.

A **user journey** is a multi-step sequence from a user's perspective with a meaningful outcome:

> "User browses catalog, adds items to cart, checks out, and sees order confirmation"

A **feature** is a technical capability:

> "The POST /cart/items endpoint creates a CartItem record"

**The feature is already tested in Integration tests.** The E2E test only exists to verify the full journey works end-to-end with real runtime, real UI, and cross-page state.

---

## What Belongs at E2E Level

**Only write an E2E test when ALL of these are true:**

1. The scenario requires a **real runtime/UI** (JavaScript must execute, native UI must render, multi-process coordination)
2. The scenario spans **multiple pages or states** that can't be tested with a single request
3. The behaviour is **not already covered** at Integration level
4. Losing this test would leave a **real, user-facing bug undetectable**

**Write E2E tests for:**

| Journey type                                                           | Why E2E is needed                                                  |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Multi-step flows crossing pages (onboarding, checkout, signup)         | Cross-page state, redirects, session — can't fake with one request |
| Flows requiring client-side interaction (modals, dynamic UI, drag/drop) | Client-side logic must actually run                                |
| Cross-role journeys (user acts → admin reviews → user sees result)     | Multi-actor, multi-page state                                      |
| Keyboard navigation and focus management                                | Real UI/rendering only                                             |
| Accessibility audits (axe-core, screen reader testing)                  | Requires rendered DOM                                              |
| Visual regression (screenshot comparison)                               | Requires real rendering                                            |

---

## What Does NOT Belong at E2E Level

**If you can test it with a direct API call or integration test, do that instead.**

| ❌ E2E anti-pattern                                       | ✅ Write this instead                                       |
| --------------------------------------------------------- | ----------------------------------------------------------- |
| Navigate to form → fill → assert redirect                 | Integration: direct API call with expected response         |
| Navigate to list page → assert item appears               | Integration: API call, assert response contains item        |
| Assert HTTP status code                                   | Integration: assert response status directly                |
| Assert a heading or label renders                         | Integration: assert response body contains expected text    |
| Test that unauthorized user is redirected                 | Integration: auth/middleware test                            |
| Test every form validation error message                  | Integration: input validation test                          |
| Test notification/email is sent when form submitted       | Integration: fake the notification sender                    |
| Create record via UI → assert record in DB                | Integration: test DB state directly                         |
| Test a flow with zero client-side interaction             | Integration: it's just an HTTP flow                         |

**The litmus test:** Remove client-side JavaScript/rendering. If the test would still pass with plain API calls, it's an Integration test. Don't write it at E2E level.

---

## One Journey, One Test

**Do not write multiple E2E tests for permutations of the same journey.** Pick the happy path. All edge cases, validation errors, and failure paths belong in Integration tests.

```
❌ BAD — testing permutations at E2E level:
  test("can create item with 1 option")
  test("can create item with 2 options")
  test("cannot create item without a name")      ← validation = Integration
  test("cannot create item with invalid date")    ← validation = Integration

✅ GOOD — one happy path:
  test("user can create an item with options and land on detail page")
```

---

## How Many E2E Tests is Too Many?

The entire E2E suite should run in under 5 minutes in CI. If it's growing beyond ~25 spec files, scrutinise every file. Ask:

- Is this testing a journey or a feature?
- Is this already covered by an Integration test?
- Would an Integration test give the same confidence with less cost?

Delete or demote anything that fails those questions.

---

## E2E Workflow

1. **Understand the UI/runtime first** — navigate, inspect, screenshot if needed. Note dynamic content, loading states, async behaviour. Ask: does this actually need a real UI to test?
2. **Check if an Integration test already covers it** — search test files before writing anything at E2E level. If an Integration test exists → do not duplicate.
3. **Review existing patterns** — check for fixtures, helpers, page objects, and conventions already in use.
4. **Write the test** — name after the **user journey**, not the feature.
5. **Run and validate** — run the test, ensure it's deterministic and not flaky.

## E2E Best Practices

- **Name tests as user goals**, not technical actions: `"user can complete checkout"`, not `"POST /cart/checkout returns 200"`
- **Prefer accessible selectors** — `getByRole()`, `getByLabel()`, `getByText()` over CSS selectors and IDs
- **Never use arbitrary waits/sleeps** — wait for conditions: URL changes, element visibility, network idle
- **Use auth fixtures** — set up authenticated state once, reuse across tests; don't log in manually every time
- **Seed only what the journey needs** — minimal data, targeted setup
- **Flaky tests must be fixed or deleted.** A flaky E2E test is worse than no test.

---

# What NOT to Test (Delete on Sight)

| Anti-pattern                                  | Why it's harmful                                   |
| --------------------------------------------- | -------------------------------------------------- |
| Framework/ORM metadata assertions             | Tests framework internals, not your code           |
| Type-casting/serialization assertions         | Same — language/framework behaviour                |
| Relationship/association existence via DB     | Test via object construction or trust your schema  |
| Config value assertions                       | Config files ARE the assertion                     |
| `class_exists()` / import checks              | Module resolution works; this tests the runtime    |
| Test fixture/factory shape assertions         | Fixtures are test infra, not production code       |
| Exact HTML/template string matching           | Couples tests to formatting/whitespace             |
| Testing that migrations/schema changes work   | Setup scripts/ORM already proves this              |
| E2E test for a plain API call                 | Direct testing is faster, cheaper, more stable     |
| E2E test for page content                     | Integration test with response assertion is enough |
| E2E test for auth/redirect logic              | Integration middleware test handles this           |
| E2E test for validation errors                | Integration input validation test is right level   |
| Duplicate E2E + Integration test for same flow| Double maintenance cost, zero extra confidence     |

---

# Test File Organisation

Adapt to the project's language/framework conventions, but aim for this structure:

```
tests/
├── Unit/              (or unit/, __tests__/unit/)
├── Integration/       (or Feature/, api/, __tests__/integration/)
└── E2E/               (or e2e/, browser/, playwright/, cypress/)
```

Organise within each layer however makes sense for the project's domain and framework.

---

# Naming Conventions

Follow the project's existing conventions. If starting fresh:

- **Test files:** Name after the thing being tested — `UserServiceTest`, `user-service.test`, `test_user_service`
- **Test cases:** Describe behaviour, not implementation — `"calculates total with discount"`, `"rejects unauthenticated requests"`
- **E2E files:** Name after the user journey — `checkout-flow.spec`, `onboarding-journey.test`
- **E2E test cases:** Describe user goals — `"user can complete checkout"`, `"new user can sign up and see dashboard"`

---

# Maintenance Rules

1. **Every test must justify its existence.** Ask: "If this test disappeared, would a real bug go undetected?" If no → don't write it.
2. **No test should break from cosmetic changes.** Reformatting a template must not break any test.
3. **No test should duplicate another.** If Integration tests cover a flow, don't also cover it in E2E unless the real UI adds genuine value.
4. **Prefer fewer, richer tests over many thin ones.** A test with 5 meaningful assertions beats 5 tests with 1 assertion each.
5. **Tests that need external dependencies belong in Integration.** If it needs a database, file system, or network, it's not a unit test. Period.

---

# References

- Martin Fowler, "TestPyramid" — https://martinfowler.com/bliki/TestPyramid.html
- Martin Fowler, "TestingErlang" (on unit vs integration boundaries)
- Kent Beck, "Test-Driven Development: By Example"
- Google Testing Blog, "Just Say No to More End-to-End Tests"

---
> Source: [nickperkins/testing](https://github.com/nickperkins/testing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
