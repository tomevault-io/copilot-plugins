## voyage-backend

> This rule, presentation_layer_sop.mdc, should be used when you are working on any aspect of the Presentation Layer of the application. This layer is responsible for all interactions with the "outside world." Refer to this rule if your task involves: Handling Incoming Requests: Creating or modifying how the application receives requests, whether they are from HTTP (e.g., REST API controllers, route definitions), WebSockets, Command-Line Interfaces (CLIs), or other external triggers. Input Validation: Implementing or updating the logic that validates data coming from external sources before it's processed further. Data Transformation: Mapping incoming data (e.g., request bodies, query parameters) into Data Transfer Objects (DTOs) for the Application Layer. Formatting data from the Application Layer (e.g., Response DTOs) into a structure suitable for the client (e.g., JSON responses, HTML views). Interacting with the Application Layer: Writing code that calls use cases or application services and handles their responses or errors. Response Generation & Error Handling: Constructing and sending responses (including appropriate status codes and headers for HTTP) and translating application errors into client-friendly messages. User Interface (UI) Elements: If the Presentation Layer directly involves UI components or views (e.g., server-side rendering). Directory Structure & Organization: Understanding or deciding where to place files related to controllers, routes, validation schemas, presentation-specific types/interfaces, or mappers within the presentation/ directory. Adhering to Clean Architecture Principles: Ensuring the Presentation Layer only depends on the Application Layer and not directly on the Domain or Infrastructure layers. Keeping controllers "thin" and focused on orchestration. Maintaining clear separation of concerns. Avoiding Common Pitfalls: Such as placing business logic in controllers, bypassing the Application Layer, or inadequate input validation. In essence, if you are building or modifying the parts of the system that users or other external systems directly interact with, this SOP is your guide.

# Standard Operating Procedure (SOP): Presentation Layer Guidelines

## 1. Introduction & Purpose

The Presentation Layer is the outermost layer in a Clean Architecture system. Its primary role is to handle all interactions with the outside world. This includes, but is not limited to:

*   User interfaces (e.g., web pages, mobile app views)
*   External API endpoints (e.g., REST, GraphQL)
*   Command-Line Interfaces (CLI)
*   Message queue consumers or event listeners that initiate actions.

The Presentation Layer is responsible for receiving input from these external sources and presenting data back to them in an appropriate format. It acts as a gateway, translating external requests into calls to the Application Layer and formatting Application Layer responses for external consumption.

## 2. Core Responsibilities

The Presentation Layer has several key responsibilities:

*   **Request Handling**: Managing incoming requests from various sources (e.g., HTTP methods like GET, POST, PUT, DELETE; WebSocket messages; CLI commands).
*   **Input Validation**: Rigorously ensuring that all incoming data is valid, well-formed, and meets expected criteria (e.g., data types, formats, required fields) before any further processing.
*   **Data Transformation (Request)**: Mapping incoming request data (e.g., HTTP request bodies, query parameters, CLI arguments) to Data Transfer Objects (DTOs) or simple data structures expected by the Application Layer's use cases.
*   **Application Layer Interaction**: Invoking the appropriate use cases or application services within the Application Layer, passing the validated and transformed input data.
*   **Data Transformation (Response)**: Mapping data received from the Application Layer (e.g., Response DTOs, domain entities) into formats suitable for the client (e.g., JSON, XML, HTML, plain text).
*   **Response Generation**: Constructing and sending responses back to the client. For HTTP, this includes setting appropriate status codes (e.g., 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 500 Internal Server Error) and headers.
*   **Error Handling**: Catching errors (both business exceptions and unexpected errors) propagated from the Application Layer and translating them into user-friendly error responses suitable for the client, without leaking sensitive internal details.
*   **(Optional) User Interface Rendering**: If the layer is responsible for rendering UI components or views (e.g., in server-side rendered web applications or desktop applications).

## 3. Recommended Directory Structure (within `presentation/`)

A well-organized Presentation Layer might have the following subdirectories:

*   **`controllers/`** (or `handlers/` for generic request handlers, `resolvers/` for GraphQL):
    *   **Purpose**: Contains the primary request-handling logic. Controllers orchestrate the flow of incoming requests to the appropriate Application Layer use cases and manage the response.
    *   **Details**: Each controller method typically corresponds to a specific endpoint, action, or user interaction. They should remain "thin" by delegating business logic.

*   **`routes/`**:
    *   **Purpose**: Defines the mapping between external entry points (e.g., API endpoints, CLI commands, event topics) and the corresponding controller methods or handlers.
    *   **Details**: This is where you configure how an incoming request like `GET /products/:id` is routed to `ProductController.getProductById()`.

*   **`validation/`**:
    *   **Purpose**: Houses schemas, rules, custom validation functions, or classes responsible for validating incoming request data.
    *   **Details**: Validation logic should be applied early in the controller before passing data to the Application Layer. Libraries like Zod, Joi, or class-validator are commonly used here.

*   **`interfaces/`** (or `types/`):
    *   **Purpose**: Defines TypeScript interfaces or types specifically for the Presentation Layer's request and response structures, especially if they differ from Application Layer DTOs or require client-specific shaping.
    *   **Details**: This helps ensure type safety and clear contracts for data exchanged with clients.

*   **(Optional) `mappers/`** (or `viewModels/`):
    *   **Purpose**: Used for transforming Application Layer DTOs or domain objects into presentation-specific formats, often called View Models. This is useful when the data structure required by the client (e.g., a specific UI component) differs significantly from what the Application Layer provides.
    *   **Details**: Mappers ensure that the Presentation Layer can adapt data to various client needs without altering the Application Layer.

*   **(Optional) `ui/`** or **`views/`**:
    *   **Purpose**: If the application serves HTML directly (Server-Side Rendering) or contains UI components specific to a framework (e.g., React components, Vue components, Angular components that are part of the presentation layer itself).
    *   **Details**: Contents would depend heavily on the UI technology chosen.

## 4. Key Principles & Best Practices

To maintain a clean and effective Presentation Layer:

*   **Adherence to Dependency Rule**:
    *   The Presentation Layer **depends on** the Application Layer (specifically its use case interfaces and DTOs).
    *   It **must NOT** depend directly on the Domain Layer (entities, domain services) or the Infrastructure Layer (repositories, external service clients). All business logic execution and data persistence should be delegated to the Application Layer.

*   **Thin Controllers/Handlers**:
    *   Controllers or handlers should be lean and act primarily as orchestrators. Their main tasks are:
        1.  Parse/extract data from the incoming request.
        2.  Validate the input data (often using dedicated validation components).
        3.  Map validated input to an Application Layer Request DTO.
        4.  Call the appropriate Application Layer use case.
        5.  Receive the Response DTO or error from the use case.
        6.  Map the Response DTO to a presentation-specific format (if necessary).
        7.  Construct and send the final response (e.g., HTTP response with status code and body).
    *   Avoid embedding any business logic within controllers.

*   **Robust Input Validation**:
    *   Validate *all* external input rigorously before any further processing. Assume all input from the outside world is potentially malicious or malformed.
    *   Use dedicated validation libraries or schemas (e.g., Zod, Joi, class-validator) to define and enforce validation rules.
    *   Return clear, informative error messages to the client when validation fails (e.g., HTTP 400 Bad Request with details about the invalid fields).

*   **DTOs for Communication**:
    *   Strictly use Data Transfer Objects (DTOs) for passing data to and receiving data from the Application Layer. These DTOs define the contract between the Presentation and Application layers.

*   **Consistent and User-Friendly Error Handling**:
    *   Implement a standardized mechanism for handling errors that propagate from the Application Layer or occur within the Presentation Layer itself.
    *   Map application-specific exceptions or error codes to appropriate HTTP status codes (or other client-relevant error representations).
    *   Provide error responses that are helpful to the client but do not leak sensitive internal system details or stack traces in a production environment.

*   **Separation of Concerns**:
    *   The Presentation Layer should *only* deal with presentation concerns: receiving requests, validating input, delegating to the application layer, formatting output, and sending responses.
    *   It should not contain any business rules, data access logic, or complex orchestration that belongs in the Application or Domain layers.

*   **Testability**:
    *   Design controllers, handlers, and other presentation components to be easily unit-testable.
    *   Dependencies on the Application Layer (use cases) should be injected (e.g., via constructor or method parameters) and easily mockable during tests.
    *   Focus tests on the Presentation Layer's responsibilities: request parsing, validation, interaction with mocks of application services, and response formatting.

## 5. Typical Workflow Example (Handling an HTTP GET Request)

1.  **Request Arrival**: An HTTP `GET` request arrives at a defined API endpoint (e.g., `/api/products/{productId}?includeDetails=true`).
2.  **Routing**: The web framework's routing mechanism (configured in `routes/`) directs the request to the `getProductById` method in a `ProductController` (located in `controllers/`).
3.  **Parameter Extraction**: The `ProductController.getProductById` method extracts path parameters (e.g., `productId`), query parameters (e.g., `includeDetails`), and headers from the incoming HTTP request.
4.  **Input Validation**: The extracted parameters (e.g., `productId` should be a valid UUID, `includeDetails` should be a boolean) are validated using schemas or rules defined in `validation/`.
    *   If validation fails, the controller immediately constructs and returns an error response (e.g., an HTTP 400 Bad Request with a JSON body like `{"error": "Invalid productId format"}`).
5.  **Request DTO Creation**: The controller creates a Request DTO (e.g., `GetProductRequestDto`) expected by the Application Layer, populating it with the validated parameters: `new GetProductRequestDto(productId, includeDetailsValue)`.
6.  **Application Layer Invocation**: The controller calls the relevant use case in the Application Layer: `this.getProductUseCase.execute(getProductRequestDto)`. The `getProductUseCase` is typically injected into the controller.
7.  **Application Layer Processing**: The `GetProductUseCase` executes the business logic, potentially interacting with the Domain Layer and fetching data via repository interfaces implemented in the Infrastructure Layer. It returns a Response DTO (e.g., `ProductDetailsResponseDto` or `SimpleProductResponseDto`) or throws an application-specific exception (e.g., `ProductNotFoundError`).
8.  **Response Handling**: The controller receives the Response DTO or catches the exception from the use case.
9.  **Error Mapping**: If an exception was caught (e.g., `ProductNotFoundError`), the controller maps it to an appropriate HTTP error response (e.g., 404 Not Found with a message like `{"error": "Product not found"}`).
10. **Success Response Formatting**: If successful, the controller takes the `ProductDetailsResponseDto` and formats it into the desired output format (e.g., a JSON string).
11. **Response Sending**: The controller sends the HTTP response back to the client with the appropriate status code (e.g., 200 OK) and the formatted body.

## 6. Common Pitfalls to Avoid

*   **Business Logic in Controllers/Handlers**: Placing decision-making logic, calculations, or business rule enforcement directly within presentation components. This logic belongs in the Application or Domain layers.
*   **Direct Data Access or Infrastructure Calls**: Controllers, handlers, or other presentation components directly interacting with databases, ORMs, external API clients, or any other infrastructure concern. This bypasses the Application Layer and violates the dependency rule.
*   **Bypassing Application Layer Use Cases**: The Presentation Layer calling Domain services or manipulating entities directly. All interactions with the core business logic should be mediated by Application Layer use cases.
*   **Inadequate or Missing Input Validation**: Trusting external input without proper validation. This can lead to security vulnerabilities (e.g., injection attacks), processing errors, and system instability.
*   **Inconsistent Error Responses**: Different parts of the API or interface returning errors in varying formats or using HTTP status codes inconsistently. This makes client-side error handling difficult.
*   **Leaking Internal System Details**: Exposing internal application structures, detailed error stack traces, or database-specific error messages to the client, especially in production. This can be a security risk and provides a poor user experience.
*   **Fat Controllers/Handlers**: Components in the Presentation Layer becoming overly large and responsible for too many distinct tasks. This indicates that responsibilities are not being properly delegated.
*   **Tight Coupling to Frameworks**: Writing code that is so deeply entangled with a specific web framework that it becomes difficult to test in isolation or to adapt if the framework changes. Aim for framework-agnostic logic where possible, especially outside of the direct request/response handling glue.

## 7. (Optional) Tooling & Framework-Specific Guidance

While the principles of the Presentation Layer are framework-agnostic, the implementation details will vary based on the chosen technology stack.

*   **Web Frameworks (e.g., Express.js, NestJS, Spring Boot, ASP.NET Core)**:
    *   Leverage built-in routing, middleware (for logging, auth, global error handling), and request/response handling capabilities.
    *   For NestJS or Spring Boot, utilize decorators and dependency injection for controllers and services.
*   **Validation Libraries (e.g., Zod, Joi, class-validator, FluentValidation)**:
    *   Use these to define clear, declarative validation schemas for request DTOs or input parameters.
    *   Integrate them into your controllers or as middleware.
*   **API Specification (e.g., OpenAPI/Swagger)**:
    *   Consider defining your API contract using a specification like OpenAPI. This can help in generating client SDKs, documentation, and even for request validation.
*   **GraphQL Libraries (e.g., Apollo Server, TypeGraphQL)**:
    *   Structure your presentation layer around resolvers and GraphQL schemas.
    *   Validation and DTO mapping still apply, often handled within resolver functions.

Always refer to the best practices and documentation of your chosen framework and tools, ensuring they align with the Clean Architecture principles outlined here.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madhavpujara-at) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
