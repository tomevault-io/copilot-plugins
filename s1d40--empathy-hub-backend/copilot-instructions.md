## empathy-hub-backend

> This project has undergone a significant migration from a PostgreSQL + SQLAlchemy backend to a **Google Cloud Firestore** backend. As part of this migration, all SQLAlchemy-related code, including the `app/db` and `app/crud` directories, has been removed. The application now uses the `firebase-admin` SDK and a custom `app/services/firestore_services` layer to interact with Firestore.

# Gemini Project Analysis: Empathy Hub Backend

## Project Status (November 2025)

This project has undergone a significant migration from a PostgreSQL + SQLAlchemy backend to a **Google Cloud Firestore** backend. As part of this migration, all SQLAlchemy-related code, including the `app/db` and `app/crud` directories, has been removed. The application now uses the `firebase-admin` SDK and a custom `app/services/firestore_services` layer to interact with Firestore.

This document has been updated to reflect the current Firestore-based architecture.

## Project Overview

The Empathy Hub backend is a Python application built with the **FastAPI** framework. It serves as the API for the Empathy Hub platform, providing endpoints for user authentication, content management (posts and comments), real-time chat, and user interaction features like muting, blocking, and reporting. The backend uses **Google Cloud Firestore** as its database.

## Core Architecture

The application follows a clean, layered architecture that separates concerns into distinct modules:

*   **`api`**: Defines the API endpoints using FastAPI's `APIRouter`. This is the entry point for all incoming HTTP requests.
*   **`core`**: Contains core logic and configuration, including security (JWT handling), application settings, and the real-time chat connection manager.
*   **`services/firestore_services`**: This layer contains the functions that directly interact with Firestore to perform data operations. It acts as an abstraction layer between the API endpoints and the database.
*   **`schemas`**: Defines the data shapes for API requests and responses using **Pydantic** models. This ensures data validation and serialization.
*   **`scripts`**: Contains utility scripts, such as generating the list of default avatar filenames.

## API Endpoints (`app/api/v1/endpoints/`)

The API is versioned under `/api/v1` and is organized into the following resource-based endpoints:

*   **`auth.py`**: Handles user authentication, specifically the `/token` endpoint for obtaining a JWT access token.
*   **`users.py`**: Manages user-related operations, including user creation, fetching user profiles (`/me`, `/{user_id}`), and updating user information. It also includes endpoints for data erasure (`/me/posts`, `/me/comments`, etc.).
*   **`posts.py`**: Provides CRUD operations for posts, including creation, retrieval, updating, and deletion. It also has an endpoint for voting on posts.
*   **`comments.py`**: Handles CRUD operations for comments on posts, including creation, retrieval, updating, deletion, and voting.
*   **`chat.py`**: Manages the real-time chat functionality. It includes endpoints for initiating chats, listing chat rooms, and a WebSocket endpoint (`/ws/{room_anonymous_id}`) for real-time communication.
*   **`user_actions.py`**: Contains endpoints for user interactions like muting, blocking, and listing muted/blocked users.
*   **`reports.py`**: Provides endpoints for users to submit reports against other users, posts, or comments, as well as admin-only endpoints for managing these reports.
*   **`avatars.py`**: A simple endpoint to retrieve the list of default avatar URLs.

## Pydantic Schemas (`app/schemas/`)

Pydantic is used extensively to define the data contracts for the API. For each model, there are typically several schemas:

*   **`*Base`**: Shared properties.
*   **`*Create`**: Properties required for creating a new record.
*   **`*Update`**: Properties that can be updated.
*   **`*Read`**: Properties to be returned in API responses, often including data from related models.

This ensures that all data entering and leaving the API is validated and conforms to the expected structure.

## Core Components

*   **Authentication (`app/core/security.py` & `app/api/v1/deps.py`)**:
    *   Authentication is based on JWT (JSON Web Tokens). The `create_access_token` function generates tokens, and the `get_current_user` dependency in `deps.py` decodes and validates the token from the `Authorization` header to identify the authenticated user for each request.
*   **Chat Management (`app/core/chat_manager.py`)**:
    *   The `ConnectionManager` class is a singleton that manages active WebSocket connections. It keeps track of which users are connected to which chat rooms and provides methods for broadcasting messages to all participants in a room.
*   **Configuration (`app/core/config.py`)**:
    *   The `Settings` class uses `pydantic-settings` to load configuration from environment variables and a `.env` file. This includes the database URL, JWT secret key, and CORS origins.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/s1d40)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/s1d40)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
