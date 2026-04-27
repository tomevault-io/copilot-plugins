## playwright-e2e-template

> This is a **Playwright E2E testing framework** with **Cucumber BDD** integration, written in **TypeScript**. The framework provides a robust, maintainable, and scalable solution for end-to-end browser automation testing with comprehensive Allure reporting capabilities.

# GitHub Copilot Instructions for Playwright E2E Template

## Project Overview

This is a **Playwright E2E testing framework** with **Cucumber BDD** integration, written in **TypeScript**. The framework provides a robust, maintainable, and scalable solution for end-to-end browser automation testing with comprehensive Allure reporting capabilities.

## Tech Stack

- **Browser Automation**: Playwright v1.45.0 (Chromium, Firefox, WebKit support)
- **BDD Framework**: Cucumber v10.3.1 (Gherkin syntax)
- **Language**: TypeScript v5.3.3 (strict mode enabled)
- **Test Runner**: cucumber-js
- **Reporting**: Allure Reports v2.34.1 + Cucumber HTML/JSON/XML reports
- **Node Runtime**: ts-node v10.9.2

## Architecture & Design Patterns

### Page Object Model (POM) Pattern ⭐ PRIMARY PATTERN

- **BasePage** (`src/pages/base.page.ts`): Abstract base class with common page operations
- **Page Classes** (`src/pages/*.page.ts`): Concrete page classes extending BasePage
  - Elements declared as `protected` properties
  - Locators initialized in constructor
  - Public methods for business actions
  - Encapsulate all page-specific logic
- **Step Definitions** (`src/step-definitions/*.steps.ts`):
  - ONLY call methods from page objects
  - NO direct interaction with browser or elements
  - NO dependency on PageHelper for page interactions
  - Focus on orchestrating test flow

### Singleton Pattern

- **BrowserManager** (`src/config/browser.config.ts`) uses the Singleton pattern to ensure one browser instance per test scenario
- Access browser via `browserManager.getPage()` ONLY when initializing page objects

### Helper Pattern

- **AssertionHelper**: Custom assertion utilities for validation patterns
- **AllureHelper**: Allure report attachment and metadata management
- **PageHelper**: DEPRECATED - Use Page Object classes instead

### BDD Pattern

- Feature files in Gherkin syntax define test scenarios in human-readable format
- Step definitions implement test orchestration by calling page object methods
- Hooks manage test lifecycle (BeforeAll, Before, After, AfterAll)

## Project Structure

```
src/
├── config/                    # Configuration files
│   ├── browser.config.ts      # Browser manager (singleton)
│   └── test.config.ts         # Test environment config
├── pages/                     # Page Object Model classes ⭐ NEW
│   ├── base.page.ts           # Abstract base page class
│   ├── *.page.ts              # Concrete page classes
│   └── index.ts               # Page exports
├── features/                  # Gherkin feature files
│   └── *.feature              # BDD test scenarios
├── step-definitions/          # Step implementations
│   └── *.steps.ts             # Step definition files (call page methods)
├── hooks/                     # Test lifecycle hooks
│   └── hooks.ts               # BeforeAll, Before, After, AfterAll
└── support/                   # Helper utilities
    ├── assertion-helper.ts    # Custom assertions
    └── allure-helper.ts       # Allure reporting helpers
```

## Coding Guidelines

### When Writing Feature Files (.feature)

1. **Use Gherkin Syntax**: Feature, Scenario, Given, When, Then, And, But
2. **Add Tags**: Use tags for organization (@smoke, @regression, @critical, etc.)
3. **Be Descriptive**: Write clear, business-readable scenarios
4. **Use Background**: For common setup steps shared across scenarios
5. **Parameterize**: Use placeholders like `{string}` for dynamic values

**Example:**

```gherkin
@smoke @regression
Feature: User Authentication
  As a user
  I want to log in to the application
  So that I can access protected features

  Background:
    Given I navigate to the login page

  @critical
  Scenario: Successful login with valid credentials
    When I enter username "user@example.com"
    And I enter password "SecurePass123"
    And I click the login button
    Then I should see the dashboard
    And I should see welcome message "Welcome back!"
```

### When Writing Step Definitions (\*.steps.ts)

1. **Import Required Decorators**: Use `Given`, `When`, `Then` from `@cucumber/cucumber`
2. **Import Page Objects**: Import page classes from `../pages`
3. **Initialize Page Objects**: Create page instance with `browserManager.getPage()`
4. **Call Page Methods**: Use page object methods instead of direct browser interaction
5. **Type Parameters**: Use TypeScript types for step parameters (string, number, etc.)
6. **Add Logging**: Include console.log for debugging and visibility
7. **Async/Await**: All step functions must be async
8. **Attach to Allure**: Use AllureHelper for screenshots, HTML snapshots, and metadata

**Example:**

```typescript
import { Given, When, Then } from "@cucumber/cucumber";
import { browserManager } from "../config/browser.config";
import { PlaywrightHomePage } from "../pages";
import { AssertionHelper } from "../support/assertion-helper";
import { AllureHelper } from "../support/allure-helper";

let playwrightHomePage: PlaywrightHomePage;

Given("I navigate to {string}", async function (url: string) {
  const page = browserManager.getPage();
  playwrightHomePage = new PlaywrightHomePage(page);
  await playwrightHomePage.openUrl(url);
  await AllureHelper.attachScreenshot(this, page, "Page Loaded");
  console.log(`✅ Navigated to: ${url}`);
});

When("I click on the search button", async function () {
  await playwrightHomePage.clickSearch();
  console.log("✅ Clicked on search button");
});

Then("the search input should be visible", async function () {
  const isVisible = await playwrightHomePage.isSearchInputVisible();
  await AssertionHelper.assertElementVisible(isVisible, "Search input");
  console.log("✅ Search input is visible");
});
```

### When Working with Page Objects

1. **Extend BasePage**: All page classes must extend the abstract BasePage
2. **Declare Elements**: Declare locators as `protected` properties
3. **Initialize in Constructor**: Initialize all locators in the constructor
4. **Use XPath ONLY**: All locators must use XPath syntax with `xpath=` prefix
5. **Public Methods Only**: Only expose public methods for actions/queries
6. **Use BasePage Methods**: Leverage inherited methods from BasePage
7. **Return Values**: Return data when needed (title, text, visibility status)
8. **Handle Errors**: Use try-catch for complex interactions
9. **No Direct Browser Access**: Don't access browserManager in page classes

**Example:**

```typescript
import { Page, Locator } from "@playwright/test";
import { BasePage } from "./base.page";

export class LoginPage extends BasePage {
  // Declare elements as protected
  protected readonly usernameInput: Locator;
  protected readonly passwordInput: Locator;
  protected readonly loginButton: Locator;

  constructor(page: Page) {
    super(page);

    // Initialize locators in constructor using XPath
    this.usernameInput = this.page.locator('xpath=//input[@name="username"]');
    this.passwordInput = this.page.locator('xpath=//input[@name="password"]');
    this.loginButton = this.page.locator('xpath=//button[@type="submit"]');
  }

  // Public methods for actions
  async login(username: string, password: string): Promise<void> {
    await this.fillInput(this.usernameInput, username);
    await this.fillInput(this.passwordInput, password);
    await this.clickElement(this.loginButton);
  }

  async getPageTitle(): Promise<string> {
    return await super.getPageTitle();
  }
}
```

### When Working with Browser Manager

1. **Never Instantiate**: Always use the singleton instance `browserManager`
2. **Get Page**: Use `browserManager.getPage()` to access the current page
3. **Only in Step Definitions**: Access browser ONLY when creating page objects
4. **Don't Launch/Close**: Browser lifecycle is managed by hooks
5. **Configuration**: Browser options are controlled via environment variables

**Example:**

```typescript
// In step definition - create page object
const page = browserManager.getPage();
const loginPage = new LoginPage(page);

// Then use page object methods
await loginPage.login("user@example.com", "password");
```

### When Writing Helper Classes

1. **Assertions**: Add custom assertions to `AssertionHelper`
2. **Allure Attachments**: Use `AllureHelper` static methods for report attachments
3. **Type Safety**: Use Playwright's Page type for type checking
4. **Error Handling**: Include try-catch blocks and meaningful error messages
5. **Page Interactions**: Add new methods to BasePage for common operations instead of PageHelper

**Example - AssertionHelper:**

```typescript
export class AssertionHelper {
  static async assertElementVisible(
    isVisible: boolean,
    elementName: string
  ): Promise<void> {
    if (!isVisible) {
      throw new Error(`Expected ${elementName} to be visible, but it was not.`);
    }
  }

  static async assertTextContains(
    actualText: string,
    expectedText: string
  ): Promise<void> {
    if (!actualText.includes(expectedText)) {
      throw new Error(
        `Expected text to contain "${expectedText}", but got "${actualText}"`
      );
    }
  }
}
```

### When Adding Allure Reporting

1. **Tags**: Use meaningful tags in feature files (maps to Allure categories)
2. **Screenshots**: Attach screenshots for important steps using `AllureHelper.attachScreenshot()`
3. **HTML Snapshots**: Capture page HTML with `AllureHelper.attachHTML()`
4. **Page Info**: Attach page metadata with `AllureHelper.attachPageInfo()`
5. **Custom Data**: Attach JSON data with `AllureHelper.attachJSON()`
6. **Metadata**: Use AllureHelper methods for epic, feature, story, severity, etc.

**Example:**

```typescript
// In step definition
await AllureHelper.attachScreenshot(this, page, "After Login");
await AllureHelper.attachPageInfo(this, page);
await AllureHelper.attachJSON(
  this,
  { userId: "123", timestamp: Date.now() },
  "User Data"
);

// Metadata (logs only - Allure integration captures from tags)
AllureHelper.epic("User Management");
AllureHelper.feature("Authentication");
AllureHelper.story("User Login");
AllureHelper.severity("critical");
```

### When Configuring Tests

1. **Environment Variables**: Use `.env` file for configuration (see `.env.example`)
   - `HEADLESS`: Set to 'false' to see browser (default: true)
   - `RECORD_VIDEO`: Set to 'true' to record videos (default: false)
   - `DEFAULT_TIMEOUT`: Test timeout in milliseconds (default: 30000)
   - `BASE_URL`: Base URL for tests
2. **Browser Config**: Modify `src/config/browser.config.ts` for browser options
3. **Test Config**: Update `src/config/test.config.ts` for test settings
4. **Cucumber Config**: Edit `cucumber.js` for test runner configuration

## Common Commands

```bash
# Run all tests
npm test

# Run tests in parallel (2 threads)
npm run test:parallel

# Run tests with browser visible
npm run test:headed

# Generate HTML report
npm run test:report

# Run with Allure reporter
npm run test:allure

# Generate and open Allure report
npm run test:allure:report

# Serve Allure report
npm run allure:serve

# Clean Allure reports
npm run allure:clean

# Install Playwright browsers
npm run playwright:install
```

## Best Practices

### DO ✅

- **Use async/await** for all asynchronous operations
- **Import types** from `@playwright/test` for type safety
- **Use descriptive names** for scenarios and step definitions
- **Add console.log** statements for debugging visibility
- **Leverage helper classes** instead of duplicating code
- **Tag scenarios** appropriately for filtering (@smoke, @regression, @critical)
- **Attach screenshots** to Allure reports for important steps
- **Wait for elements** before interacting with them
- **Use Page Object pattern** through PageHelper for complex pages
- **Write atomic scenarios** that test one thing at a time
- **Clean up** test data in After hooks if needed
- **Use meaningful variable names** and follow TypeScript conventions
- **Add JSDoc comments** for complex helper methods
- **Handle errors gracefully** with try-catch blocks
- **Use Playwright's built-in locators** (getByRole, getByText, etc.) when possible

### DON'T ❌

- **Don't create multiple browser instances** - use the singleton
- **Don't launch/close browser manually** - hooks handle this
- **Don't use hard-coded waits** - use Playwright's smart waiting
- **Don't skip error handling** in helper methods
- **Don't write long scenarios** - break them into smaller ones
- **Don't duplicate step definitions** - make them reusable
- **Don't ignore test failures** - investigate and fix them
- **Don't commit** node_modules, reports, or .env files
- **Don't use CSS selectors** when semantic selectors are available
- **Don't forget to add types** to function parameters
- **Don't use 'any' type** unless absolutely necessary
- **Don't create side effects** between test scenarios
- **Don't hardcode test data** - parameterize in feature files
- **Don't use deprecated Playwright APIs**

### DON'T ❌

- **Don't create multiple browser instances** - use the singleton
- **Don't launch/close browser manually** - hooks handle this
- **Don't use hard-coded waits** - use Playwright's smart waiting
- **Don't skip error handling** in helper methods
- **Don't write long scenarios** - break them into smaller ones
- **Don't duplicate step definitions** - make them reusable
- **Don't ignore test failures** - investigate and fix them
- **Don't commit** node_modules, reports, or .env files
- **Don't use CSS selectors** when semantic selectors are available
- **Don't forget to add types** to function parameters
- **Don't use 'any' type** unless absolutely necessary
- **Don't create side effects** between test scenarios
- **Don't hardcode test data** - parameterize in feature files
- **Don't use deprecated Playwright APIs**

## Page Object Model (POM) Implementation Details

### Architecture Overview

Framework sử dụng **Page Object Model** pattern với 3 layers:

1. **BasePage** (Abstract Class) - `src/pages/base.page.ts`

   - Protected `page` property
   - Common navigation methods: `navigateTo()`, `reloadPage()`, `goBack()`
   - Element interactions: `clickElement()`, `fillInput()`, `isVisible()`, `waitForElement()`
   - Information getters: `getPageTitle()`, `getPageUrl()`, `getTextContent()`
   - Screenshot: `takeScreenshot()`

2. **Concrete Page Classes** - `src/pages/*.page.ts`

   - Extend BasePage
   - Declare elements as `protected` properties
   - Initialize locators in constructor using XPath
   - Public methods for business actions
   - Example structure:

   ```typescript
   export class MyPage extends BasePage {
     protected readonly submitButton: Locator;

     constructor(page: Page) {
       super(page);
       this.submitButton = this.page.locator('xpath=//button[@type="submit"]');
     }

     async submit(): Promise<void> {
       await this.clickElement(this.submitButton);
     }
   }
   ```

3. **Step Definitions** - `src/step-definitions/*.steps.ts`
   - Only call page object methods
   - No direct browser or element interaction
   - No PageHelper dependency
   - Create page instance with `browserManager.getPage()`

### Key Principles

**✅ Separation of Concerns:**

- Page Objects: Element locators + page interactions
- Step Definitions: Test logic + orchestration
- Helpers: Utilities + assertions

**✅ Encapsulation:**

- Elements declared `protected`
- Locators initialized in constructor
- Only public methods exposed

**✅ Inheritance:**

- All pages extend BasePage
- Reuse common methods
- Override when needed

**✅ Single Responsibility:**

- Each page class handles one page
- Each method does one thing
- Clear layer boundaries

### Method Naming Conventions

- **Actions**: `clickButton()`, `fillInput()`, `selectOption()`
- **Queries**: `getTitle()`, `getText()`, `getUrl()`
- **Checks**: `isVisible()`, `isEnabled()`, `isChecked()`
- **Waits**: `waitForLoad()`, `waitForElement()`

### Creating New Page Objects - Quick Steps

1. **Create page class** extending BasePage:

```typescript
// src/pages/my-page.page.ts
export class MyPage extends BasePage {
  protected readonly element: Locator;

  constructor(page: Page) {
    super(page);
    this.element = this.page.locator('xpath=//div[@id="element"]');
  }

  async doAction(): Promise<void> {
    await this.clickElement(this.element);
  }
}
```

2. **Export from index.ts**:

```typescript
export { MyPage } from "./my-page.page";
```

3. **Use in step definitions**:

```typescript
let myPage: MyPage;

Given("I am on my page", async function () {
  const page = browserManager.getPage();
  myPage = new MyPage(page);
  await myPage.navigateTo("https://example.com");
});

When("I do action", async function () {
  await myPage.doAction();
});
```

### Benefits Achieved

- **Maintainability** ⭐⭐⭐⭐⭐: Change locators in one place
- **Reusability** ⭐⭐⭐⭐⭐: Page methods reused across scenarios
- **Testability** ⭐⭐⭐⭐⭐: Test page classes independently
- **Type Safety** ⭐⭐⭐⭐⭐: Full TypeScript support with autocomplete
- **Scalability** ⭐⭐⭐⭐⭐: Easy to add new pages and extend

## Locator Strategy - XPath Only

This framework uses **XPath ONLY** for all element locators in page objects.

### XPath Syntax (Required)

All locators must use XPath with explicit `xpath=` prefix:

```typescript
// ✅ Good - XPath with explicit prefix
this.submitButton = this.page.locator('xpath=//button[@type="submit"]');
this.emailInput = this.page.locator('xpath=//input[@name="email"]');
this.errorMsg = this.page.locator('xpath=//*[contains(@class, "error")]');

// ❌ Bad - Don't use CSS selectors
this.submitButton = this.page.locator("#submit");
this.emailInput = this.page.locator('input[name="email"]');
```

### Common XPath Patterns

#### 1. Basic Selection

```typescript
// By tag
xpath=//button

// By ID
xpath=//*[@id="myid"]
xpath=//button[@id="submit"]

// By name
xpath=//input[@name="username"]

// By type
xpath=//input[@type="search"]
xpath=//button[@type="submit"]

// By class (contains)
xpath=//*[contains(@class, "error-message")]
```

#### 2. Text Matching

```typescript
// Exact text
xpath=//a[text()="Get started"]
xpath=//button[text()="Submit"]

// Contains text
xpath=//*[contains(text(), "Welcome")]
xpath=//h1[contains(text(), "Login")]

// Normalize space (ignore whitespace)
xpath=//span[normalize-space()="Click here"]
```

#### 3. Multiple Attributes

```typescript
// AND conditions
xpath=//button[@type="submit" and contains(@class, "primary")]
xpath=//input[@name="email" and @type="email"]

// OR conditions
xpath=//input[@type="search" or @type="text"]
xpath=//*[contains(@class, "btn-primary") or contains(@class, "btn-success")]
```

#### 4. Attribute Contains

```typescript
// Contains
xpath=//*[contains(@id, "submit")]
xpath=//*[contains(@class, "button")]
xpath=//a[contains(@href, "login")]

// Starts with
xpath=//*[starts-with(@id, "user-")]
xpath=//*[starts-with(@class, "btn-")]
```

#### 5. Parent/Child/Sibling Navigation

```typescript
// Parent
xpath=//button[@id="submit"]/parent::div

// Ancestor
xpath=//button[@id="submit"]/ancestor::div[@class="form"]

// Direct child
xpath=//div[@class="container"]/button

// Any descendant
xpath=//div[@class="container"]//button

// Following sibling
xpath=//label[@for="email"]/following-sibling::input

// Preceding sibling
xpath=//input[@id="email"]/preceding-sibling::label
```

#### 6. Index and Position

```typescript
// First element
xpath=(//button)[1]

// Last element
xpath=(//li)[last()]

// Position functions
xpath=//div[@class="list"]/*[position()=1]
xpath=//li[position() >= 2 and position() <= 4]
```

#### 7. Advanced Patterns

```typescript
// NOT condition
xpath=//button[not(@disabled)]
xpath=//div[not(@class)]

// Complex conditions
xpath=//button[@type="submit" and contains(@class, "primary") and not(@disabled)]

// Any element
xpath=//*[@class="highlight"]
xpath=//*[text()="Click me"]
```

### XPath Best Practices

**✅ DO:**

1. Use explicit `xpath=` prefix
2. Use stable attributes (id, name, type, data-testid)
3. Use `.first()` when multiple matches expected
4. Combine conditions for specificity
5. Use `contains()` for partial matching

**❌ DON'T:**

1. Don't use overly complex paths
2. Don't rely on position-based paths
3. Don't use absolute paths from root
4. Don't forget explicit prefix
5. Don't mix with CSS selectors

### XPath Testing in Browser

Test XPath in browser DevTools console:

```javascript
$x('//button[@type="submit"]'); // Get all matches
$x('//button[@type="submit"]').length; // Count
$x('//button[@type="submit"]')[0]; // First match
```

### XPath Priority for Attributes

1. **Stable attributes** (best):

   - `[@id="unique-id"]`
   - `[@name="field-name"]`
   - `[@type="submit"]`
   - `[@data-testid="element-id"]`

2. **Text content** (good):

   - `[text()="Exact Text"]`
   - `[contains(text(), "Partial")]`

3. **Class/complex** (use when needed):
   - `[contains(@class, "class-name")]`
   - Multiple conditions with `and`/`or`

### Real Framework Examples

```typescript
// PlaywrightHomePage
this.searchButton = this.page.locator('xpath=//button[@aria-label="Search"]').first();
this.searchInput = this.page.locator('xpath=//input[@type="search" or contains(@class, "DocSearch-Input")]').first();

// LoginPage
this.usernameInput = this.page.locator('xpath=//input[@name="username"]');
this.errorMessage = this.page.locator('xpath=//*[contains(@class, "error-message")]');

// DocumentationPage
async clickOnLink(linkText: string): Promise<void> {
  const linkLocator = this.page.locator(`xpath=//*[text()="${linkText}"]`);
  await this.clickElement(linkLocator);
}
```

### XPath Quick Reference

| Need          | XPath                                                 |
| ------------- | ----------------------------------------------------- |
| By ID         | `xpath=//*[@id="myid"]`                               |
| By Class      | `xpath=//*[contains(@class, "myclass")]`              |
| By Name       | `xpath=//input[@name="myname"]`                       |
| By Type       | `xpath=//input[@type="text"]`                         |
| By Text       | `xpath=//*[text()="Click"]`                           |
| Contains Text | `xpath=//*[contains(text(), "Click")]`                |
| OR            | `xpath=//input[@type="text" or @type="email"]`        |
| AND           | `xpath=//button[@type="submit" and @class="primary"]` |
| Starts With   | `xpath=//*[starts-with(@id, "prefix-")]`              |
| Parent        | `xpath=//element/parent::div`                         |
| Child         | `xpath=//div[@class="parent"]/child`                  |
| Descendant    | `xpath=//div[@class="parent"]//descendant`            |

## Refactoring Summary

### ✅ Completed Implementation

**Page Object Model:**

- ✅ Created BasePage abstract class with common methods
- ✅ Implemented 3 concrete page classes (PlaywrightHomePage, LoginPage, DocumentationPage)
- ✅ Refactored all step definitions to use page objects exclusively
- ✅ Created common.steps.ts for shared steps with pageContext pattern
- ✅ Removed direct browser access from step definitions
- ✅ All tests passing (2 scenarios, 7 steps, ~4-5s execution)

**XPath Implementation:**

- ✅ Converted all locators to XPath with explicit `xpath=` prefix
- ✅ Used OR conditions: `xpath=//input[@type="search" or contains(@class, "DocSearch-Input")]`
- ✅ Used contains() for class matching: `xpath=//*[contains(@class, "error-message")]`
- ✅ All tests validated and passing with XPath locators

**File Structure:**

```
src/pages/               ⭐ NEW
├── base.page.ts         # Abstract base class
├── playwright-home.page.ts
├── login.page.ts
├── documentation.page.ts
└── index.ts

src/step-definitions/
├── common.steps.ts      ⭐ NEW - Shared steps
├── example.steps.ts     ✏️ REFACTORED
├── screenshot-demo.steps.ts ✏️ REFACTORED
└── fail-demo.steps.ts   ✏️ REFACTORED

src/support/
├── page-helper.ts       ⚠️ DEPRECATED
├── assertion-helper.ts
└── allure-helper.ts
```

### 🔄 Optional Future Enhancements

**High Priority:**

1. **Component Objects** - Extract reusable UI components (SearchBar, Navigation)
2. **Fluent API** - Method chaining for better readability
3. **Data Builders** - Test data management with faker.js

**Medium Priority:** 4. **Page Factory** - Centralized page instance management 5. **Advanced Waiting** - Smart waiting strategies with retry logic 6. **API Integration** - Hybrid API+UI tests for faster setup

**Low Priority:** 7. **Performance Monitoring** - Track page load times and metrics 8. **Mobile Support** - Mobile-specific page objects and gestures 9. **Visual Regression** - Screenshot comparison testing 10. **Accessibility Testing** - axe-core integration with WCAG checks

**Technical Debt:**

- Remove PageHelper class completely
- Create migration guide for legacy tests
- Add video tutorials and troubleshooting guide

### 📊 Benefits Achieved

**Before POM (Old approach):**

```typescript
When("I click search", async function () {
  const page = browserManager.getPage();
  const pageHelper = new PageHelper(page);
  await pageHelper.clickElement(".search-button");
});
```

❌ Direct browser access in steps  
❌ Tight coupling with PageHelper  
❌ Hard-coded locators  
❌ Not reusable

**After POM (Current approach):**

```typescript
// Page class
export class PlaywrightHomePage extends BasePage {
  protected readonly searchButton: Locator;

  constructor(page: Page) {
    super(page);
    this.searchButton = this.page.locator(
      'xpath=//button[@aria-label="Search"]'
    );
  }

  async clickSearch(): Promise<void> {
    await this.clickElement(this.searchButton);
  }
}

// Step definition
When("I click search", async function () {
  await playwrightHomePage.clickSearch();
});
```

✅ Clean step definition  
✅ Encapsulated locators  
✅ Reusable page methods  
✅ Maintainable code  
✅ Type-safe with autocomplete

## Error Handling Pattern

```typescript
Given("I perform complex action", async function () {
  const page = browserManager.getPage();
  try {
    await page.click("#button");
    await page.waitForSelector(".result", { timeout: 5000 });
    console.log("✅ Action completed successfully");
  } catch (error) {
    console.error("❌ Action failed:", error);
    await AllureHelper.attachScreenshot(this, page, "Error State");
    throw error; // Re-throw to fail the test
  }
});
```

## Debugging Tips

1. **Run in headed mode**: `npm run test:headed` to see browser actions
2. **Check screenshots**: Review failure screenshots in `reports/screenshots/`
3. **Inspect HTML snapshots**: Failure HTML is saved alongside screenshots
4. **Use console logs**: Add console.log in step definitions for visibility
5. **Review Allure reports**: Detailed step-by-step execution with attachments
6. **VS Code debugging**: Set breakpoints in step definitions and use VS Code debugger
7. **Slow motion**: Set `SLOW_MO=500` environment variable to slow down execution

## TypeScript Configuration

- **Target**: ES2022
- **Module**: CommonJS
- **Strict mode**: Enabled
- **ESM Interop**: Enabled
- **Root**: `./src`
- **Output**: `./dist`

## Dependencies Reference

### Core Dependencies

- `@playwright/test`: ^1.45.0 - Browser automation library
- `@cucumber/cucumber`: ^10.3.1 - BDD testing framework
- `typescript`: ^5.3.3 - TypeScript compiler
- `ts-node`: ^10.9.2 - TypeScript execution engine
- `@types/node`: ^20.11.0 - Node.js type definitions
- `allure-commandline`: ^2.34.1 - Allure CLI tool
- `allure-cucumberjs`: ^3.4.1 - Allure-Cucumber reporter

## File Naming Conventions

- **Feature files**: `*.feature` (kebab-case, e.g., `user-login.feature`)
- **Step definitions**: `*.steps.ts` (kebab-case, e.g., `user-login.steps.ts`)
- **Helper classes**: `*-helper.ts` (kebab-case, e.g., `api-helper.ts`)
- **Config files**: `*.config.ts` (kebab-case, e.g., `browser.config.ts`)
- **Hook files**: `hooks.ts`

## Test Report Locations

- **Cucumber HTML**: `reports/cucumber-report.html`
- **Cucumber JSON**: `reports/cucumber-report.json`
- **JUnit XML**: `reports/cucumber-report.xml`
- **Screenshots**: `reports/screenshots/`
- **Allure Results**: `reports/allure-results/`
- **Allure Report**: `reports/allure-report/`

## Environment Variables Template

```bash
# Browser Configuration
HEADLESS=true
SLOW_MO=0
RECORD_VIDEO=false

# Timeouts (milliseconds)
DEFAULT_TIMEOUT=30000
NAVIGATION_TIMEOUT=30000

# Test Environment
BASE_URL=https://playwright.dev

# Allure Configuration
ALLURE_RESULTS_DIR=reports/allure-results
```

## When Suggesting Code

1. **Always provide TypeScript** - not JavaScript
2. **Include imports** at the top of code snippets
3. **Use async/await** syntax consistently
4. **Follow the existing patterns** in the codebase
5. **Add type annotations** to all parameters and return types
6. **Include console.log** statements for visibility
7. **Add comments** for complex logic
8. **Suggest Allure attachments** for important steps
9. **Use the helper classes** when applicable
10. **Follow the singleton pattern** for BrowserManager access

## Special Considerations

- **Parallel Execution**: Tests can run in parallel (configured in cucumber.js)
- **Browser Isolation**: Each scenario gets a fresh browser context
- **Automatic Cleanup**: Hooks handle browser lifecycle automatically
- **Screenshot on Failure**: Hooks automatically capture screenshots and HTML on test failure
- **Cross-browser**: Framework supports Chromium, Firefox, and WebKit (configured in browser.config.ts)
- **CI/CD Ready**: JSON and XML reports are generated for CI/CD integration
- **Type Safety**: Strict TypeScript mode is enabled for better code quality
- **Allure Integration**: Comprehensive reporting with screenshots, HTML snapshots, and metadata

## Code Review Checklist

When reviewing or suggesting code, ensure:

- [ ] TypeScript types are correctly used
- [ ] Async/await is used for all Playwright operations
- [ ] BrowserManager singleton is accessed correctly
- [ ] Helper classes are used for common operations
- [ ] Console logs are included for debugging
- [ ] Allure attachments are added for important steps
- [ ] Error handling is implemented where needed
- [ ] Step definitions match feature file steps exactly
- [ ] Locators use best practices (user-facing attributes first)
- [ ] No hard-coded waits (use Playwright's smart waiting)
- [ ] Code follows existing patterns and conventions
- [ ] No side effects between test scenarios

---

**Remember**: This framework prioritizes maintainability, reusability, and readability. Always follow BDD principles and leverage the existing helper classes and patterns. Write tests that business stakeholders can understand and that developers can easily maintain.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karamata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
