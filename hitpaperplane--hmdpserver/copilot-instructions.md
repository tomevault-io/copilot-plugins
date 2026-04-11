## hmdpserver

> This document provides a comprehensive overview of the HmdpServer project, its architecture, and instructions for development, building, and running the application.

# GEMINI Project Analysis: HmdpServer

This document provides a comprehensive overview of the HmdpServer project, its architecture, and instructions for development, building, and running the application.

## Project Overview

HmdpServer is a full-stack web application that emulates a "Yelp-like" platform for customer reviews and local business discovery. The name "Hmdp" likely stands for "Hao Men Dian Ping" (好店点评 - Good Shop Reviews).

The application features include:
*   User authentication via email/SMS verification.
*   Browsing and searching for shops.
*   Voucher "seckill" (flash sales) functionality.
*   A social feed for user posts and interactions.
*   User profiles and follow/follower relationships.

## Architecture

The project follows a microservices-oriented architecture with a decoupled frontend.

*   **Frontend**: A modern single-page application (SPA) built with Vue.js.
*   **Backend**: A set of Java-based microservices built with the Spring Boot framework.
*   **Web Server/Proxy**: Nginx is used to serve the frontend application and act as a reverse proxy for the backend services.
*   **Data Storage**: MySQL is used as the primary relational database, and Redis is used extensively for caching, session management, and implementing features like distributed locks and rate limiting.
*   **Message Queue**: RabbitMQ is used for asynchronous task processing, particularly for handling high-concurrency scenarios like the "seckill" feature.

## Backend

The backend is a multi-module Maven project. The main services are:

*   `hmdp-service`: The core backend service containing most of the business logic.
*   `feed-service`: A service dedicated to handling the user feed.
*   `order-service`: A service for managing orders.
*   `relay-service`: A service whose purpose is not immediately clear from the file structure, but may be related to data synchronization or message relaying.

### Technologies

*   **Framework**: Spring Boot 2.3.12.RELEASE
*   **Database**: MySQL
*   **ORM**: MyBatis-Plus 3.5.3.1
*   **In-Memory Data Store**: Redis (requires version >= 6.2 for GEO commands)
*   **Messaging**: RabbitMQ 3.9
*   **Java Version**: JDK 1.8
*   **Build Tool**: Apache Maven

### Building and Running

1.  **Configuration**:
    *   Navigate to `hmdp-service/src/main/resources/application.yaml`.
    *   Update the connection details for `spring.datasource` (MySQL), `spring.redis`, and `spring.rabbitmq` to match your local environment.

2.  **Build**:
    *   From the root directory of a service (e.g., `hmdp-service/`), run the Maven build command:
        ```bash
        mvn clean package
        ```

3.  **Run**:
    *   Execute the generated JAR file to start the service:
        ```bash
        java -jar target/hm-dianping-0.0.1-SNAPSHOT.jar
        ```
    *   The main service (`hmdp-service`) will run on port `8088` by default.

## Frontend

The frontend is a Vue.js 3 application.

### Technologies

*   **Framework**: Vue.js 3
*   **Build Tool**: Vite
*   **UI Library**: Vant 4
*   **State Management**: Pinia 2
*   **Routing**: Vue Router 4
*   **Text Editor**: Quill

### Building and Running

1.  **Installation**:
    *   Navigate to the `hmdp-frontend` directory.
    *   Install the dependencies:
        ```bash
        npm install
        ```

2.  **Development Server**:
    *   Start the local development server:
        ```bash
        npm run dev
        ```

3.  **Production Build**:
    *   To create a production-ready build, run:
        ```bash
        npm run build
        ```
    *   The output will be in the `hmdp-frontend/dist` directory.

## Nginx Configuration

The `nginx/hmdp.conf` file contains the Nginx configuration. It is set up to:
1.  Serve the static files for the frontend application (from `hmdp-frontend/dist`).
2.  Proxy API requests starting with `/api` to the backend `hmdp-service` running on `localhost:8088`.

## Development Conventions

*   **Backend**: The backend code follows standard Spring Boot conventions. Lombok is used to reduce boilerplate code for entities and DTOs.
*   **Frontend**: The frontend uses Vue's Single File Components (`.vue`) structure. Code is organized by feature into directories like `views`, `components`, `stores`, and `api`.
*   **Asynchronous Operations**: The application makes heavy use of asynchronous processing with RabbitMQ for scalability and Redis for caching and distributed operations to improve performance and user experience.
*   **Database Schema**: The initial database schema can be found in `hmdp-service/src/main/resources/db/hmdp.sql`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HITPaperPlane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HITPaperPlane)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
