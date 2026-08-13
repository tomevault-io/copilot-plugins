## cors-bridge

> This is a simple RSS proxy server built with Node.js, Express, and TypeScript. It allows users to fetch RSS feeds from other domains by providing a URL, bypassing CORS restrictions. The server fetches the feed, caches it in memory for 5 minutes, and returns it to the client with the appropriate `Access-Control-Allow-Origin` header.

# Gemini Code Assistant Context

## Project Overview

This is a simple RSS proxy server built with Node.js, Express, and TypeScript. It allows users to fetch RSS feeds from other domains by providing a URL, bypassing CORS restrictions. The server fetches the feed, caches it in memory for 5 minutes, and returns it to the client with the appropriate `Access-Control-Allow-Origin` header.

The project follows a clean architecture pattern, with a clear separation of concerns between the presentation, application, and domain layers.

**Key Technologies:**

- **Node.js:** The runtime environment.
- **Express:** The web framework for handling HTTP requests.
- **TypeScript:** The programming language.
- **pino:** For logging.
- **node-cache:** For in-memory caching.
- **Docker:** For containerization.

## Building and Running

### Prerequisites

- Node.js (>=18.19.0)
- npm

### Development

To run the server in development mode with hot reloading:

```sh
npm run dev
```

### Production

To build and run the server in production mode:

1.  **Build the project:**

    ```sh
    npm run build
    ```

2.  **Start the server:**

    ```sh
    npm start
    ```

### Docker

To build and run the project with Docker:

1.  **Build the Docker image:**

    ```sh
    docker build -t rss-proxy .
    ```

2.  **Run the Docker container:**

    ```sh
    docker run -p 8080:8080 rss-proxy
    ```

## Development Conventions

- **Linting:** The project uses ESLint for code quality and consistency. To run the linter:

  ```sh
  npm run lint
  ```

- **Formatting:** The project uses Prettier for code formatting. To format the code:

  ```sh
  npm run format
  ```

  The project includes a `.prettierrc.json` file for configuration.

- **Architecture:** The project follows a clean architecture pattern with three layers:
  - **`presentation`:** Handles HTTP requests and responses.
  - **`application`:** Contains the business logic.
  - **`domain`:** Defines the data structures.
  - **`infrastructure`:** Contains the implementation of the repositories.
- **Dependency Injection:** The project uses dependency injection to decouple the different layers. For example, the `RssService` depends on the `FeedRepository` interface, not on a concrete implementation.

---
> Source: [nico0695/cors-bridge](https://github.com/nico0695/cors-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
