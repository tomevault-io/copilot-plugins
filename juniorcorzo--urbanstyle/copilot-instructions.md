## urbanstyle

> This is a robust and high-performance e-commerce platform for buying and selling clothes online. The application is built using a hexagonal architecture, allowing for greater flexibility and scalability.


# UrbanStyle Project Analysis

## Project Overview

This is a robust and high-performance e-commerce platform for buying and selling clothes online. The application is built using a hexagonal architecture, allowing for greater flexibility and scalability.

### Frontend

The user interface is developed using Astro as the main framework for building fast and modern websites, along with React for interactive components. Astro allows for the generation of optimized and efficient sites, facilitating agile development and optimal performance in the browser. React, with its focus on reusable components, complements the interactive and easy-to-maintain user experience.

### Backend

The server logic is implemented with Java and the Spring Boot framework. Spring Boot provides a fast and efficient development environment, allowing for the creation of secure and scalable RESTful APIs for communication with the frontend. Java, as a robust and widely used language in the business world, guarantees the stability and performance of the backend.

Additionally, an image processing module inspired by Cloudflare's service was implemented, which allows for the efficient optimization and transformation of images. The images are not stored after being processed but are served dynamically through the API. The only persistent processing consists of transforming the images to the WebP format to improve performance and efficiency in the delivery of multimedia files.

### Database

For data storage, MongoDB is used, a NoSQL document-oriented database. MongoDB offers flexibility in the data schema and high scalability, which makes it ideal for handling the large amount of information associated with an e-commerce site, such as products, users, orders, and more.

For the shopping cart, Redis is used, an in-memory storage system that allows for fast and efficient access to the cart data, which facilitates the user experience when making purchases.

## Building and Running

### Frontend

To run the frontend in a development environment, use the following command:

```bash
pnpm dev
```

To build the frontend for production, use the following command:

```bash
pnpm build
```

### Backend

To run the backend in a development environment, use the following command:

```bash
./gradlew bootRun
```

To build the backend for production, use the following command:

```bash
./gradlew build
```

## Development Conventions

### Frontend

The frontend code is linted using ESLint. To check the code for linting errors, use the following command:

```bash
pnpm lint
```

To automatically fix linting errors, use the following command:

```bash
pnpm lint:fix
```

### Backend

The backend code is built using Gradle. The project follows the standard Java and Spring Boot conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JuniorCorzo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
