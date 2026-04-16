## chale-ham

> A lightweight expense-sharing web application built for a specific group of friends.

# Chale-Ham Expense Sharing Application

## Project Overview
A lightweight expense-sharing web application built for a specific group of friends.

## Tech Stack
- **Frontend**: React.js
- **Backend**: FastAPI (Python)
- **Authentication**: Google OAuth
- **Database**: GitHub Repository (JSON Document Store) & MongoDB Atlas

## Current Progress
- [x] Initialized Backend directory structure.
- [x] Implemented Pydantic models for `Person`, `Expense`, `Trip`, and `User`.
- [x] Created `GitHubService` for interacting with the GitHub API (CRUD on JSON files).
- [x] Implemented `AuthService` for Google OAuth token verification and Admin check.
- [x] Developed `main.py` with FastAPI endpoints.
- [x] Dockerized Backend (Dockerfile, .dockerignore).
- [x] Dockerized Frontend (Dockerfile, .dockerignore, package.json).
- [x] Created `docker-compose.yml` for orchestration with Hot Reloading.
- [x] Set up Frontend (React.js) with routing.
- [x] Implement Google OAuth in Frontend.
- [x] Create Dashboard UI.
- [x] Create Trip Detail and Expense Adding UI.
- [x] Implement User Tracking in GitHub (users/ folder by email).
- [x] Integrate MongoDB Atlas for user data management.
- [x] Implement Admin Dashboard for user status management (Active/Inactive).
- [x] Fixed Docker Hot Reloading issues on Windows.
- [x] Refactored Frontend into component-based folder structure.
- [x] Implemented Multi-theme CSS (Light/Dark/Mobile).
- [x] Created separate Header, Footer, and Profile components.
- [x] Added 404 NotFound page.

## Pending Tasks
- [ ] End-to-end testing.
- [ ] Implement Expense deletion (Admin only).
- [ ] Enhance UI/UX with better styling.

## Environment Variables Required
- `GITHUB_PAT`: Personal Access Token for GitHub API.
- `GITHUB_USERNAME`: GitHub Username.
- `GITHUB_REPO_NAME`: Name of the repository used as a database.
- `GOOGLE_CLIENT_ID`: Google OAuth Client ID.
- `ADMIN_EMAIL`: `ka25eq0346@gmail.com` (hardcoded).
- `MONGO_URI`: MongoDB Atlas connection string.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lohitpt252003) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
