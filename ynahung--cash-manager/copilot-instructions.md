## cash-manager

> Create a clear directory structure for both Django and React components.


Rules for LLM Agent
General Guidelines

    Project Structure:
        Create a clear directory structure for both Django and React components.
        Keep backend and frontend code in separate folders (e.g., backend for Django and frontend for React).

    Environment Setup:
        Use Poetry for managing Python dependencies.
        Initialize the Poetry project with poetry init and manage dependencies in pyproject.toml.
        Use poetry install to install the required packages.

Backend (Django) Rules

    Django Setup:
        Initialize a new Django project using django-admin startproject.
        Create apps within the project using python manage.py startapp <app_name>.

    Database Configuration:
        Use SQLite for development; configure PostgreSQL for production.
        Ensure models are defined in models.py with appropriate relationships (ForeignKey, ManyToMany).

    API Creation:
        Use Django REST Framework to create RESTful APIs.
        Define serializers for models in serializers.py.

    CORS Handling:
        Install and configure django-cors-headers to allow requests from the React frontend.

    Authentication:
        Implement token-based authentication using Django REST Framework’s authentication classes.

    Testing:
        Write unit tests for models and views.
        Ensure all tests can be run with python manage.py test.

Frontend (React) Rules

    React Setup:
        Create a new React app using create-react-app.
        Organize components in a structured way (e.g., components, pages, hooks).

    State Management:
        Use React Context or Redux for global state management as needed.

    API Integration:
        Use axios or fetch to make API calls to the Django backend.
        Handle asynchronous operations with async/await.

    Routing:
        Use react-router-dom for client-side routing.
        Define routes for all pages and components.

    Styling:
        Use CSS Modules or styled-components for scoped styling.
        Ensure responsive design practices are followed.

Deployment Rules

    Deployment Preparation:
        Use gunicorn for serving Django in production.
        Build the React app (npm run build) for production deployment.

    Environment Variables:
        Store sensitive information (like API keys) in environment variables.
        Use .env files for local development.

    Final Testing:
        Conduct end-to-end testing to ensure integration between frontend and backend works smoothly.

Documentation

    Code Documentation:
        Comment code and maintain a clear README file.
        Document API endpoints and usage with tools like Swagger or Postman.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ynahung) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
