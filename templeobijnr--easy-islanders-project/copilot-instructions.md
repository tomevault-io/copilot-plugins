## easy-islanders-project

> This document outlines my understanding of the Easy Islanders project, its architecture, and my plan for contributing.

# Gemini's Project Analysis: Easy Islanders

This document outlines my understanding of the Easy Islanders project, its architecture, and my plan for contributing.

## Project Overview

Easy Islanders is a real estate platform that connects property sellers with potential buyers and renters. The platform features a chat-based interface where an AI-powered agent assists users in finding properties that match their needs. The system is designed to handle both short-term and long-term rentals, with a clear distinction between the two.

The core of the application is a Django-based backend with a PostgreSQL database. The frontend is built with React. The project also includes a sophisticated intent routing system that uses a combination of machine learning and rule-based approaches to understand user queries and route them to the appropriate agent or service.

## Key Technologies

*   **Backend:** Django, Django Rest Framework, Python
*   **Frontend:** React, JavaScript, Axios
*   **Database:** PostgreSQL with PostGIS for geospatial queries
*   **AI/ML:** scikit-learn, OpenAI (optional)
*   **Real-time Communication:** Django Channels, WebSockets
*   **Authentication:** JWT, HttpOnly Cookies
*   **Deployment:** Docker, fly.io

## Directory Structure

*   `assistant/`: The core Django app for the AI assistant, including models, views, and services.
*   `easy_islanders/`: The main Django project, including settings and URL configurations.
*   `frontend/`: The React frontend application.
*   `real_estate/`: A Django app for managing property listings and availability.
*   `scripts/`: Various scripts for tasks like seeding the database and evaluating the router.
*   `docs/`: Project documentation, including API contracts and architectural diagrams.

## Core Components

### 1. AI Assistant & Intent Router

The AI assistant is the primary user interface for the platform. It uses a sophisticated intent router to understand user queries and determine the user's intent. The router is trained on a corpus of user utterances and can distinguish between different types of requests, such as property searches, booking inquiries, and general questions.

### 2. Real Estate App

The `real_estate` app manages property listings, including details about the property, pricing, and availability. The app supports both short-term and long-term rentals, with a clear data model to differentiate between the two.

### 3. Authentication & Authorization

The application uses a secure authentication system based on JWTs and HttpOnly cookies. This approach provides protection against common web vulnerabilities like XSS and CSRF.

### 4. Real-time Communication

WebSockets are used for real-time communication between the frontend and backend, enabling features like live chat and notifications.

## Next Steps

My immediate goal is to familiarize myself with the codebase and the development workflow. I will start by:

1.  **Running the project locally:** I will follow the instructions in the `README.md` file to set up the development environment and run the application on my local machine.
2.  **Exploring the frontend:** I will examine the React components and the overall structure of the frontend application to understand how it interacts with the backend.
3.  **Diving into the assistant app:** I will study the `assistant` app to understand how the AI assistant works, including the intent router and the different agent services.
4.  **Contributing to the project:** Once I have a solid understanding of the project, I will look for opportunities to contribute, such as fixing bugs, adding new features, or improving the documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/templeobijnr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/templeobijnr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
