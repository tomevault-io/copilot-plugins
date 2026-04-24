## react-package-template

> Rules for useful code comments and docstrings.


## Rules for Good Code Comments
- Avoid Temporal Context: Do not reference recent refactors, changes, or historical context. Comments should describe code as it currently is, not how it evolved.
- Explain Why, Not What: Clarify intent or reasoning behind code choices. Avoid describing exactly what the code does; the code itself should be clear enough.
- Avoid Redundant or Noisy Comments: Don’t restate obvious logic. Only comment where additional context significantly aids understanding.
- Preserve Useful Comments: Only remove comments that are actively false, misleading, or outdated. Even seemingly redundant comments may be useful to others.
- Keep Comments Up-to-Date: Update comments immediately whenever code changes. Outdated comments are worse than none.
- Document Surprises Clearly: Clearly comment unusual implementations, hacks, or workarounds. Include references or issue links when available.
- Be Concise and Clear: Short, direct comments are best. Avoid verbosity or ambiguous language.
- Include References: Link to external resources, algorithms, or documentation when implementing complex or standard patterns.
- TODO Comments: Always clearly specify the next steps and resolution conditions. Include ticket numbers or issue IDs if available.
- Standardize Comment Style: Consistency matters. Adopt and enforce a uniform commenting format project-wide.

## Documentation Comments (Docstrings)
- Public API Documentation:
    - Document all exported functions.
    - Document all exported custom types.
- Clarity and Accuracy: Keep descriptions concise, accurate, and relevant. Regularly verify that annotations match the current implementation.
- JSDoc Usage:
    - Prefer JSDoc syntax in TypeScript projects.
    - If a file already consistently uses another docstring format, continue using it.
    - Consistency within a file is more important than strictly following external standards.
- Provide Useful Context: Focus comments on clarifying intent, usage examples, and context beyond what TypeScript types convey.
- Examples:
    - Basic Function:
        ```typescript
        /**
         * Calculates total price including tax.
        *
        * @param price - Initial price.
        * @param taxRate - Tax rate as decimal (e.g., 0.2 for 20%).
        * @returns Price including tax.
        *
        * @example
        * calculateTotal(100, 0.2); // 120
        */
        const calculateTotal = (price: number, taxRate: number): number => {
            return price * (1 + taxRate);
        };
        ````
    - Optional & Default Parameters:
        ```typescript
        /**
         * Logs a message with optional verbosity.
        *
        * @param message - Message to log.
        * @param verbose - Log verbosely (default: false).
        */
        const logMessage = (message: string, verbose: boolean = false): void => {
            console[verbose ? 'info' : 'log'](message);
        };
        ```
    - Custom Type Definition:
        ```typescript
        /**
        * @typedef {Object} User
        * @property {string} id - User's unique identifier
        * @property {string} name - User's full name
        * @property {number} age - User's age
        */
        type User = {
            id: string;
            name: string;
            age: number;
        };

        /**
         * Fetches user details by ID.
        *
        * @param id - The user's unique identifier.
        * @returns User details.
        */
        const getUser = (id: string): User => {
            return { id, name: "Alice", age: 30 };
        };
        ```
- Common JSDoc Tags:
    | Tag           | Usage                             | Description                                     |
    | ------------- | --------------------------------- | ----------------------------------------------- |
    | `@param`      | `@param name Description`         | Describes function parameters                   |
    | `@returns`    | `@returns Description`            | Describes the return value                      |
    | `@typedef`    | `@typedef {Type} Name`            | Defines a custom type                           |
    | `@type`       | `@type {Type}`                    | Defines type for variables                      |
    | `@property`   | `@property name Description`      | Describes object properties                     |
    | `@example`    | `@example Code Example`           | Provides usage examples                         |
    | `@throws`     | `@throws {ErrorType} Description` | Describes potential exceptions                  |
    | `@deprecated` | `@deprecated Reason/Version`      | Marks code as deprecated                        |
    | `@see`        | `@see Reference/Link`             | References related documentation                |
    | `@private`    | `@private`                        | Indicates code should not appear in public docs |
    | `@async`      | `@async`                          | Marks function as asynchronous                  |

## Sanity Check
- If asked when you can have "no comment" you should answer "only when speaking to the paparazzi"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaronbassett) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
