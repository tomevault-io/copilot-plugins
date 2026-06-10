## posthog-demo-3000

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HogFlix Demo 3000 is a Flask-based demo application instrumented with PostHog analytics. It simulates a movie streaming service and is used to showcase PostHog features including event tracking, feature flags, and LLM analytics.

## Commands

```bash
# Install dependencies (uses uv)
make install

# Run the app (checks env, inits DB, seeds PostHog data, creates artifacts)
make run

# Initialize local SQLite database only
make db

# Seed historical events to PostHog (default: 30 days, 100 iterations)
make seed DAYS=30 ITER=100

# Create PostHog demo artifacts (actions, cohorts, insights, feature flags)
make artifacts

# Run tests
make test
```

## Environment Setup

Copy `.env.example` to `.env` and configure:
- `PH_HOST`: PostHog host (e.g., `https://us.i.posthog.com`)
- `PH_PROJECT_KEY`: PostHog project API key
- `PH_PERSONAL_API_KEY`: Personal API key for creating artifacts
- `PH_PROJECT_ID`: Project ID
- `OPENAI_API_KEY`: (Optional) Enables the chat feature with LLM analytics

## Architecture

**Core Application (`app.py`)**: Flask app with routes for user auth, movie browsing, plans/subscriptions, blog, and an LLM chat feature. PostHog events are captured throughout for analytics demos.

**Database Models (`models.py`)**: SQLAlchemy models for User, Movie, MovieStats, and BlogPost. Uses SQLite by default.

**Scripts (`scripts/`)**:
- `seed_demo_data.py`: Generates pseudo-random historical events in PostHog
- `create_posthog_artifacts.py`: Creates actions, cohorts, insights, and feature flags via PostHog API
- `generate_fake_names_and_emails.py`: Regenerates the CSV of fake users

**PostHog Integration**: The app uses the `posthog` Python SDK. When `OPENAI_API_KEY` is set, LangChain is used with PostHog's callback handler for LLM analytics tracing.

---
> Source: [PostHog/posthog-demo-3000](https://github.com/PostHog/posthog-demo-3000) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
