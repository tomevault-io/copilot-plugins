## ai-coding-template

> Defines error handling standards and best practices for the application


# Error Handling Guidelines

This document outlines consistent error handling practices to ensure robust, user-friendly, and maintainable applications.

## Core Principles

1.  **Clarity and Consistency**:
    *   Error messages shown to users **MUST** be clear, concise, and easy to understand, avoiding technical jargon.
    *   Internal error logs **MUST** contain detailed technical information for debugging.
    *   Error responses from APIs **MUST** follow a consistent format.

2.  **Fail Gracefully**:
    *   Applications **MUST** handle errors gracefully, preventing crashes and data corruption.
    *   Provide fallback mechanisms or degraded functionality where appropriate.

3.  **Security**:
    *   Error messages **MUST NOT** expose sensitive information (e.g., stack traces in production, internal system details, PII).
    *   Validate and sanitize all inputs to prevent errors caused by malicious data.

4.  **Log Effectively**:
    *   All significant errors **MUST** be logged with sufficient context (see `logging_guidelines.md`).
    *   Distinguish between client-side and server-side errors in logs.

## Frontend Error Handling (React)

1.  **React Error Boundaries**:
    *   Implement Error Boundaries at appropriate levels in the component tree to catch JavaScript errors in their child component tree, log those errors, and display a fallback UI.
    *   Use Error Boundaries for sections of the UI that are not critical to the entire application's function.

2.  **Async Operations and Promises**:
    *   Always handle promise rejections. Use `async/await` with `try/catch` blocks for asynchronous operations (e.g., API calls).
    *   Update UI state to reflect loading, success, and error states of async operations.

3.  **User Feedback**:
    *   Provide non-intrusive user feedback for errors (e.g., toast notifications, inline messages).
    *   For form submissions, display validation errors clearly next to the respective fields.

4.  **Client-Side Logging**:
    *   Log unexpected client-side errors to a remote logging service to help diagnose issues encountered by users.
    *   Include context such as browser version, user actions leading to the error, and component state.

## Backend Error Handling (Hono API)

1.  **Consistent HTTP Status Codes**:
    *   Use appropriate HTTP status codes to indicate the nature of the error (e.g., `400` for client errors, `401` for authentication, `403` for authorization, `404` for not found, `500` for server errors).

2.  **Standardized Error Response Format**:
    *   APIs **MUST** return error responses in a consistent JSON format, as defined in `backend_guidelines.md` (e.g., `{ error: { code: "ERROR_CODE", message: "User-friendly message", details?: any } }`).
    *   `code` should be a machine-readable error identifier.
    *   `message` should be a human-readable explanation.
    *   `details` can provide additional context for specific errors (e.g., validation failures).

3.  **Custom Error Classes**:
    *   Create custom error classes (e.g., `NotFoundError`, `ValidationError`, `AuthenticationError`) that extend the base `Error` class.
    *   These classes can encapsulate specific status codes and error codes.

4.  **Hono Error Handling Middleware**:
    *   Utilize Hono's built-in error handling or implement custom error handling middleware.
    *   This middleware should catch errors, log them, and transform them into the standard error response format.
    *   Ensure it's one of the last middleware to be registered.

5.  **Input Validation Errors**:
    *   Validate all incoming data (body, params, query). Libraries like Zod are recommended.
    *   Return a `400 Bad Request` status with detailed validation errors in the response body.

6.  **Server-Side Logging**:
    *   Log all unhandled exceptions and significant operational errors at `ERROR` level.
    *   Include correlation ID, request details (path, method, redacted parameters/body), and stack trace (in non-production environments).

## Specific Error Types

1.  **Validation Errors**: Provide clear messages indicating which fields are invalid and why.
2.  **Authentication/Authorization Errors**: Return `401 Unauthorized` or `403 Forbidden`. Avoid revealing whether a user exists or not on failed login.
3.  **Not Found Errors**: Return `404 Not Found` for resources that don't exist.
4.  **Service Unavailable/Third-Party Errors**: Handle errors from external services gracefully. Consider retries with backoff or circuit breaker patterns. Log these and, if appropriate, inform the user of a temporary issue.

## Testing Error Handling
*   Write unit and integration tests specifically for error conditions.
*   Verify that the correct error messages are displayed/logged and the system behaves as expected.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shamail)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Shamail)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
