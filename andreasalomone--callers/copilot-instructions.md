## callers

> These guidelines apply when writing code that consumes (calls) external or internal APIs.

# API Client Coding Guidelines

These guidelines apply when writing code that consumes (calls) external or internal APIs.

*   **Clear and Intentional Client Functions:**
    *   **Principle:** Wrapper functions or methods for API calls should clearly state their purpose and the specific endpoint they interact with.
    *   **Application:** Name functions descriptively, e.g., `getUserDetails(userId)` or `createProduct(productData)`. Abstract away the underlying HTTP request details (method, headers, URL construction) within these functions.

*   **Robust Error Handling & Retries:**
    *   **Principle:** Anticipate and handle potential API errors gracefully, including network issues, rate limits, and server-side errors.
    *   **Application:** Check HTTP status codes and handle common ones explicitly (e.g., 400, 401, 403, 404, 429, 500, 503). Implement retry mechanisms with exponential backoff for transient errors (like 429 or 5xx), but avoid retrying client errors (4xx) unless appropriate for the specific API.

*   **Data Validation (Request Payloads):**
    *   **Principle:** Validate data before sending it in an API request to catch errors early and ensure the payload conforms to the API's schema.
    *   **Application:** Before making an API call that includes a payload, ensure required fields are present and data types are correct. This can prevent unnecessary API calls and provide better error messages to the user or calling code.

*   **Secure Handling of Credentials & Sensitive Data:**
    *   **Principle:** Never hardcode API keys, tokens, or other sensitive credentials. Avoid logging sensitive request or response data.
    *   **Application:** Use environment variables, configuration files, or secure secret management systems for API credentials. When logging, sanitize or omit sensitive information from payloads and headers.

*   **Efficient Data Fetching & Payload Management:**
    *   **Principle:** Request only the data you need and be mindful of payload sizes.
    *   **Application:** Utilize API features like field selection (e.g., GraphQL, or REST APIs supporting `fields` parameters) if available. For APIs that return large datasets, implement pagination. Avoid sending excessively large payloads in POST/PUT requests.

*   **Idempotency Awareness:**
    *   **Principle:** Understand and leverage API idempotency where available, especially for operations that modify data.
    *   **Application:** For non-idempotent POST requests that create resources, be cautious about retries to avoid creating duplicate resources. If an API supports idempotency keys, use them to safely retry requests.

*   **Configuration and Abstraction:**
    *   **Principle:** Abstract API base URLs and other configurations to allow for easy changes (e.g., switching between staging and production environments).
    *   **Application:** Store base URLs, timeouts, and other API client settings in configuration files or environment variables. An API client class or module can encapsulate these settings. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreasalomone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andreasalomone)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
