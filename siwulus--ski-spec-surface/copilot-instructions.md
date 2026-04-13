## ski-spec-surface

> - Initialize configuration only with Chromium/Desktop Chrome browser

## TESTING

### Guidelines for E2E

#### PLAYWRIGHT

- Initialize configuration only with Chromium/Desktop Chrome browser
- Use browser contexts for isolating test environments
- Implement the Page Object Model for maintainable tests in ./e2e/page-objects
- Use `data-testid` attributes when introducing resilient test-oriented selectors
- When following `data-testid` convention, locate elements by `await page.getByTestId('selectorName')`
- Leverage API testing for backend validation
- Implement visual comparison with expect(page).toHaveScreenshot()
- Use the codegen tool for test recording
- Leverage trace viewer for debugging test failures
- Implement test hooks for setup and teardown
- Use expect assertions with specific matchers
- Leverage parallel execution for faster test runs
- Follow 'Arrange', 'Act', 'Assert' approach to test structure for simplicity and readability.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siwulus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/siwulus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
