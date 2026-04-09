## dental-dashboard

> Effective logging is essential for monitoring application health, debugging issues, and understanding system behavior. This project uses Winston for logging. As an expert Software Engineer, you are expected to implement comprehensive and structured logging.

# Winston Logging Guidelines

Effective logging is essential for monitoring application health, debugging issues, and understanding system behavior. This project uses Winston for logging. As an expert Software Engineer, you are expected to implement comprehensive and structured logging.

## General Principles

-   **AI Role Reminder**: Remember, you are an expert SWE tasked to write efficient, well-commented code while following specific development guidelines, including these logging practices.
-   **Log Every Logical Connection and Workflow**: Ensure that all significant operations, workflow steps, external API calls, and critical decision points are logged. This creates a traceable history of application execution.
-   **Structured Logging**: Use structured logging by passing an object as the second argument to Winston log methods. This allows for easier parsing, searching, and filtering of logs.
    ```typescript
    // ✅ DO:
    logger.info('User login attempt', { userId: user.id, ipAddress: req.ip });

    // ❌ DON'T:
    logger.info(`User login attempt for ${user.id} from ${req.ip}`);
    ```
-   **Context is Key**: Include relevant contextual information in your logs, such as user IDs, request IDs, transaction IDs, relevant data snippets (be mindful of sensitive information), and error details.
-   **Avoid Sensitive Information**: Never log sensitive data like passwords, API keys, personal identifiable information (PII) directly in logs unless absolutely necessary and properly secured/masked. If logging request/response bodies, ensure sensitive fields are filtered.

## Log Levels

Use appropriate log levels to categorize the importance and nature of log messages. The standard levels are:

-   `error`: For critical errors that prevent normal operation or indicate a significant failure. These require immediate attention.
    ```typescript
    logger.error('Failed to connect to database', { error: err.message, stack: err.stack });
    ```
-   `warn`: For potential issues or unexpected situations that do not immediately halt the application but might lead to problems if not addressed. Also for deprecated API usage or recoverable errors.
    ```typescript
    logger.warn('Deprecated API endpoint hit', { endpoint: req.path, userId });
    ```
-   `info`: For general operational messages that highlight the progress of the application. Useful for tracking workflows and major events.
    ```typescript
    logger.info('Order processed successfully', { orderId: order.id, customerId: customer.id });
    ```
-   `http`: For logging HTTP requests and responses (method, URL, status code, duration). Often handled by middleware.
    ```typescript
    logger.http('HTTP Request', { method: req.method, url: req.url, statusCode: res.statusCode, durationMs: responseTime });
    ```
-   `verbose`: For detailed information that is not typically needed for general monitoring but can be useful for more in-depth debugging of specific functionalities.
    ```typescript
    logger.verbose('Cache lookup details', { cacheKey, hit: true, itemSize });
    ```
-   `debug`: For fine-grained information useful only during development and debugging. These logs are usually disabled in production.
    ```typescript
    logger.debug('Variable value check', { variableName: 'myVar', value: myVar });
    ```
-   `silly`: For the most detailed (and often excessive) information, typically only used for diagnosing very specific, complex issues during development.

**Choosing the Right Level:** Consider the environment and the audience for the logs. Production logs should be less verbose than development logs. `info` and above are common for production environments.

## What to Log

-   **Application Lifecycle Events**: Startup, shutdown, initialization of services.
-   **Requests and Responses**: Incoming requests, outgoing requests to external services, and their responses (headers, status codes, timings). Consider using dedicated logging middleware for HTTP traffic.
-   **Function Calls**: Entry and exit of critical functions, especially those involving I/O, complex business logic, or state changes. Log key parameters and return values (or summaries if large).
-   **Business Logic Decisions**: Points where the application makes significant choices or follows different paths based on data or conditions.
-   **Errors and Exceptions**: All caught exceptions and error conditions, along with stack traces and relevant context.
-   **Resource Usage**: Potentially, if relevant (e.g., database connection pool status, memory usage at critical points), though this often overlaps with dedicated monitoring tools.
-   **Security Events**: Authentication successes and failures, authorization checks, potential security anomalies.

## Winston Setup (Conceptual)

While the specific Winston setup might exist elsewhere in the project (e.g., in a dedicated logging module `src/lib/logger.ts`), ensure your logging calls are consistent with how Winston is configured (e.g., expected metadata fields, log formatting).

Example of a logger import:
```typescript
import logger from '@/lib/logger'; // Assuming logger is configured and exported here
```

By adhering to these Winston logging guidelines, you will contribute to a more observable, debuggable, and maintainable system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AojdevStudio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AojdevStudio)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
