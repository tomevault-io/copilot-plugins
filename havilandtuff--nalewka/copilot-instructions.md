## nalewka

> This document provides a summary of the `nalewka` project, a web application for managing homemade liqueurs.

# Gemini Project Overview: nalewka

This document provides a summary of the `nalewka` project, a web application for managing homemade liqueurs.

## Project Description

The `nalewka` project is a web application designed to help users track the process of making homemade liqueurs (nalewka). It allows users to:

- Create and manage different types of liqueurs.
- Record the ingredients and quantities for each batch.
- Track the bottling of each batch, including the number of bottles and volume.

## Technology Stack

The project is built with the following technologies:

- **Backend:** Python, Flask
- **Database:** SQLAlchemy, Flask-SQLAlchemy, Flask-Migrate (supports PostgreSQL and SQLite)
- **Authentication:** Flask-Login
- **Forms:** Flask-WTF
- **Deployment:** Gunicorn

## Key Features

- **User Management:** Users can register and log in to the application.
- **Liquor Management:** Create, view, and delete different types of liqueurs.
- **Batch Management:** Create and manage batches for each liquor, including detailed ingredient lists.
- **Bottle Tracking:** Track the number of bottles and volume produced for each batch.
- **Service Layer:** Business logic is separated from the presentation layer in a dedicated `services.py` module.

## Project Structure

The project follows a standard Flask application structure:

- **Application Factory:** The application is created using the factory pattern in `app/__init__.py`.
- **Blueprints:** Routes are organized using a Blueprint in `app/routes.py`.
- **Models:** Database models are defined in `app/models.py`.
- **Forms:** Web forms are defined in `app/forms.py`.
- **Templates:** HTML templates are located in the `app/templates` directory.
- **Configuration:** Application configuration is managed in `config.py`.
- **Entry Point:** The main application entry point is `nalewka.py`, which also includes CLI commands for database management.

## Refactoring Plan

A detailed refactoring plan is available in [TODO.md](TODO.md).

## My Approach

My goal is to be a cheerful and supportive assistant in all our interactions. However, when it comes to coding and technical tasks, I will always act with the utmost professionalism. I will think critically about every solution, and I will self-reflect on any advice I provide before presenting it to you. My aim is to ensure that all my contributions are well-considered, robust, and of high quality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HavilandTuff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HavilandTuff)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
