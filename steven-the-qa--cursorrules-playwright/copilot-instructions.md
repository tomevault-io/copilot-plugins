## cursorrules-playwright

> 0. Always use TypeScript instead of vanilla JavaScript whenever possible.

# Playwright Testing - Best Practices
0. Always use TypeScript instead of vanilla JavaScript whenever possible.
1. For locators, always bias towards using Playwright's `getByTestId()` locator method to target elements, unless absolutely unable to do so.
    1. If using `getByTestId()` is not possible, then using a `getByRole()` locator is the next best option.
    2. If using `getByRole()` is not possible, then using a `getByPlaceholder()` or `getByLabel()` or `getByAltText()` locator is the next best option.
    3. If using `getByPlaceholder()` or `getByLabel()` or `getByAltText()` is not possible, then using a `getByText()` locator is the next best option.
        1. If using `getByText()`, always use the `{ exact: true }` option to avoid unwanted partial matches.
    5. If using `getByText()` is not possible, then use whatever locator Playwright allows you to use to target the element.
        1. If using an Xpath or CSS selector is necessary, please try to take the shortest path to target the element and leverage any available attributes to target the element.
2. Save any locator to a variable before using it in a test. Examples:
    DO:
    ```typescript
    const exampleSignInButton = page.getByTestId("ExampleButton_signIn");
    await exampleSignInButton.click();
    ```
    DON'T:
    ```typescript
    await page.getByTestId("ExampleButton_signIn").click();
    ```
3. If logic is being reused in multiple tests, recommend creating a reusable helper/utility function to share this logic across tests. a. If a helper function is created to follow the DRY principle, don't require more arguments in the function than are needed to share the logic. b. Never assert anything in a helper function. Helper functions are reserved for performing a series of user actions, not testing the outcome of those actions.
4. Whenever possible, use Playwright's built-in features, methods, and utilities instead of "reinventing the wheel" with raw JavaScript or TypeScript solutions to solved problems. Examples:
    DO:
    ```typescript
    await expect(async () => {
        await radioButton.click(); // locator defined outside this block
        await expect(radioButton).toBeChecked();
    }).toPass();
    ```
    DON'T:
    ```typescript
    export async function radioButtonGroupClick(
        page: Page,
        urlToWaitFor: string, // The URL where the radio button is
        radioButton: Locator,
        errorTextTrigger: Locator // A button you can click that will throw error text if the radio button wasn't clicked
    ) {
        await page.waitForURL(urlToWaitFor);
        let iterationCounter = 0;
        let errorTextIsVisible = await page.getByTestId("ErrorText").isVisible();
        let buttonWasClicked = await radioButton.isChecked();
        while (errorTextIsVisible || !buttonWasClicked) {
            if (iterationCounter === 5) {
                break;
            } else {
                iterationCounter++;
            }
            await radioButton.click();
            buttonWasClicked = await radioButton.isChecked();
            if (buttonWasClicked) {
                await errorTextTrigger.click();
                errorTextIsVisible = await page.getByTestId("ErrorText").isVisible();
            }
        }
    }
    ```
    The entry point to Playwright's documentation is here if you need to look available features: https://playwright.dev/docs/intro.
5. When doing so follows the other rules in this file, try to write tests similar to other existing tests in the Playwright suite. The Playwright suite consists of everything within the `tests/` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/steven-the-qa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
