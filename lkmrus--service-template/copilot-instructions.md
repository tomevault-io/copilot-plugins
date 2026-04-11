## service-template

> This project is a billing module built with the NestJS framework. It is designed to be a modular and scalable solution for managing subscriptions and payments. The architecture is event-driven and follows principles of Domain-Driven Design (DDD) and hexagonal architecture, ensuring a clear separation of concerns.

# GEMINI.md

## Project Overview

This project is a billing module built with the NestJS framework. It is designed to be a modular and scalable solution for managing subscriptions and payments. The architecture is event-driven and follows principles of Domain-Driven Design (DDD) and hexagonal architecture, ensuring a clear separation of concerns.

The module is designed to work both as part of a monolithic application and as an independent microservice. It uses Stripe as the payment gateway and interacts with other parts of the system asynchronously through events.

### Key Technologies:

*   **Framework:** NestJS
*   **Language:** TypeScript
*   **Payment Gateway:** Stripe
*   **Architecture:** Domain-Driven Design (DDD), Hexagonal Architecture, Event-Driven
*   **Key Dependencies:**
    *   `@nestjs/core`: Core NestJS framework
    *   `@nestjs/common`: Common NestJS utilities
    *   `@nestjs/event-emitter`: For event-driven communication
    *   `stripe`: Stripe Node.js library for payment processing

## Building and Running

### Installation

```bash
npm install
```

### Running the application

```bash
# development
npm run start

# watch mode
npm run start:dev

# production mode
npm run start:prod
```

### Running tests

```bash
# unit tests
npm run test

# e2e tests
npm run test:e2e

# test coverage
npm run test:cov
```

## Development Conventions

### Architecture

The project follows a hexagonal architecture, with the code organized into the following layers:

*   **`domain`**: Contains the core business logic, entities, and domain events. This layer is independent of any external frameworks or libraries.
*   **`application`**: Contains the application-specific logic, such as use cases and event listeners. It orchestrates the domain layer to perform business operations.
*   **`infrastructure`**: Contains the implementation details for external dependencies, such as database repositories and payment gateway adapters.
*   **`presentation`**: Contains the API controllers and other entry points to the application.

### Event-Driven Communication

The module uses an event-driven approach for communication with other parts of the system.

*   **Incoming Events:** The module listens for events from other domains (e.g., `user.created`) to trigger actions within the billing context.
*   **Outgoing Events:** The module emits events to notify other parts of the system about changes in the billing state (e.g., `subscription.status.changed`).

### Payment Gateway Integration

The integration with the payment gateway (Stripe) is abstracted through a `PaymentGateway` interface. This allows for easier testing and potential replacement of the payment provider in the future. The `StripeAdapter` provides the concrete implementation for the Stripe API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lkmrus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lkmrus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
