## seleniumautomation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Selenium WebDriver test automation framework (Java 17 + Maven) targeting https://www.saucedemo.com. Tests are written two ways side by side: plain JUnit 5 (`LoginTest`) and Cucumber/Gherkin (`TestRunner` + `login.feature`). Both drive the same Page Object layer.

## Commands

Maven isn't on PATH in this environment — use the copy bundled with IntelliJ IDEA Community Edition:

```bash
MVN="/c/Users/harki/AppData/Local/Programs/IntelliJ IDEA Community Edition/plugins/maven-plugin/lib/maven3/bin/mvn.cmd"
```

- Compile: `"$MVN" clean compile`
- Run everything (JUnit + Cucumber): `"$MVN" clean test`
- Run headless (no visible browser — required in this shell/CI): `"$MVN" test -Dheadless=true`
- Run a single JUnit class: `"$MVN" test -Dtest=LoginTest -Dheadless=true`
- Run just the Cucumber suite: `"$MVN" test -Dtest=TestRunner -Dheadless=true`
- Switch browser without touching code: `"$MVN" test -Dbrowser=firefox -Dheadless=true`
- Run a tagged subset once tags exist: `-Dcucumber.filter.tags="@smoke"`
- Cucumber HTML report: `target/cucumber-report.html` (only produced after a test run; embeds a screenshot on any failed scenario)

Any `-Dkey=value` on the command line overrides `src/test/resources/config.properties` for that run — this is how CI (`.github/workflows/tests.yml`) forces `headless=true` without code changes.

## Architecture

All code lives under `src/test/java/com/demo` — there is no `src/main`, everything here is test code.

- **`framework/config/ConfigReader`** — static holder that loads `src/test/resources/config.properties` once. `System.getProperty` always wins over the file, which is the mechanism behind every `-D` override above.
- **`framework/driver/DriverFactory` + `DriverManager`** — `DriverFactory` builds a configured `WebDriver` from config (`browser`, `headless`); `DriverManager` stores it in a `ThreadLocal` so parallel test threads never share a browser. `DriverManager.quitDriver()` must be paired with every `setDriver()` (see hooks below) or the thread leaks a driver reference.
- **`pages/`** — Page Object Model. `BasePage` is the single choke point all element interaction passes through (`click`, `type`, `getText`, `isVisible`) — every helper uses an explicit `WebDriverWait`; there are no implicit waits and no `Thread.sleep()` anywhere in the framework. Page methods that lead somewhere new return the next page (`LoginPage.loginAs()` → `ProductsPage`), so a test reads as a user journey and the compiler prevents calling products-page methods before logging in. `LoginPage.submitLogin()` (no return value) exists specifically for the failed-login path, which has no "next page."
- **Two parallel test entry points, both exercising the same pages:**
  - `LoginTest` — plain JUnit 5, `@BeforeEach`/`@AfterEach` open/close the browser via `DriverManager`.
  - `runner/TestRunner` — an empty class annotated with `@Suite`/`@IncludeEngines("cucumber")` that points Cucumber at `src/test/resources/features` and glues it to the `com.demo` package (covers both `stepdefs` and `hooks`). **Must be named `Test*`/`*Test`/`*Tests`** or Maven Surefire won't discover it and `mvn test` will silently run zero Cucumber scenarios.
  - `hooks/Hooks` — Cucumber equivalent of `@BeforeEach`/`@AfterEach`. Runs on every scenario. `@After` hooks fire **highest `order` first**: the screenshot hook (`order = 2`) runs before the browser-close hook (`order = 1`), so the screenshot happens before the driver is gone. Getting this backwards means trying to photograph a closed browser.
  - `stepdefs/LoginSteps` — translates Gherkin sentences to page-object calls. Assertions belong here, never inside a Page Object — pages report facts, steps judge them.

## Conventions worth preserving

- Explicit waits only, always through `BasePage`. If new code needs `Thread.sleep()`, that's a sign a wait helper is missing, not that sleep is the answer.
- New locators are `private final By` fields inside the relevant page class — never inline in a step definition or test.
- New browsers are added as a `case` in `DriverFactory.createChrome/createFirefox`-style switch — no test or page file should ever change to support a new browser.
- Cucumber step methods used by more than one scenario (e.g. a shared `When`) must not assume a specific page is reached afterward, since failure-path scenarios land somewhere different than happy-path ones.

---
> Source: [panesarharkiratsingh/SeleniumAutomation](https://github.com/panesarharkiratsingh/SeleniumAutomation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
