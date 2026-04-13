## oidc-auth

> Goal: Create a standalone, reusable Python library (fastapi-oidc-middleware) that provides OIDC authentication and session management for FastAPI and Starlette applications.

# FastAPI OIDC Middleware Library

## 1\. Project Overview

Goal: Create a standalone, reusable Python library (fastapi-oidc-middleware) that provides OIDC authentication and session management for FastAPI and Starlette applications.  
Target Audience: Developers building multiple microservices who need a "drop-in" authentication solution without duplicating code.

## 2\. Core Architecture

The library is composed of three decoupled components to ensure flexibility:

1. **OIDCController:** Handles the business logic (Login redirect, Token exchange, User info fetch). It must remain framework-agnostic where possible.  
2. **OIDCAuthMiddleware:** A Starlette BaseHTTPMiddleware implementation that intercepts requests. It delegates logic to the Controller.  
3. **SessionStorage:** An abstract interface for session persistence. The default implementation is in-memory, but the design must support Redis/Memcached plugins.

## 3\. AI Coding Directives (The "Rules")

When generating or refactoring code for this project, adhere to the following directives:

### A. Library-First Mindset

* **No Hardcoding:** Never hardcode secrets, URLs, or specific app instances. All configuration must be passed via a Config object or Environment Variables.  
* **Dependency Injection:** Do not instantiate global objects (like app \= FastAPI()) inside the library modules. The library provides classes; the consuming application instantiates them.  
* **Framework Compatibility:** While targeted at FastAPI, ensure the core middleware inherits from starlette.middleware.base so it works with raw Starlette apps if needed.

### **B. Session Management**

* **Abstract Storage:** Do not couple the logic to a specific database. Use a dictionary-like interface for sessions so users can swap Dict\[str, dict\] for a Redis wrapper easily.  
* **Cookie Security:** Always enforce httponly=True and samesite='lax' (or 'strict') for session cookies. Allow secure flag to be toggled via config.

### C. Type Safety & Documentation

* **Type Hints:** All function signatures must be fully type-hinted.  
* **Docstrings:** Every public class and method (OIDCController, dispatch, etc.) must have a docstring explaining its purpose and arguments.

### **D. Packaging Structure**

When asked to restructure for distribution, organize files as:

/fastapi\_oidc/  
    ├── \_\_init\_\_.py  
    ├── middleware.py  \# Contains OIDCAuthMiddleware  
    ├── controller.py  \# Contains OIDCController  
    ├── main.py        \# FastAPI instance for test purposes
    ├── types.py       \# Configuration TypedDicts/Pydantic models  
    └── session.py     \# Session protocols

## **4\. Common Tasks & Commands**

* **Refactor for Pydantic:** If asked to improve configuration, suggest using pydantic.BaseSettings for the OIDC\_CONFIG dictionary.  
* **Add Redis Support:** If asked for persistence, create a class that adheres to the MutableMapping protocol (or a custom SessionStore protocol) using redis-py.  
* **Unit Testing:** When generating tests, use pytest and httpx to mock the OIDC provider endpoints. Do not make real network calls to Google/Auth0 during tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cetres)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cetres)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
