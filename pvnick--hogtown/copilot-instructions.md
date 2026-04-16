## hogtown

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Django web application project called "hogtown_project" with a core app.

## Development Commands

### Environment Setup
- Activate virtual environment: `source venv/bin/activate`
- Install dependencies: `pip install -r requirements.txt`

### Django Commands
- Run development server: `python manage.py runserver`
- Create migrations: `python manage.py makemigrations`
- Apply migrations: `python manage.py migrate`
- Create superuser: `python manage.py createsuperuser`
- Run tests: `python manage.py test`
- Collect static files: `python manage.py collectstatic`

### Project Structure
- `hogtown_project/`: Main Django project directory containing settings and configuration
- `core/`: Primary Django app for core functionality
- `venv/`: Python virtual environment (excluded from git)
- `requirements.txt`: Python package dependencies

## Email Configuration

### Development
- Default: Console backend (emails printed to console)
- To test with AWS SES locally, update `.env` file:
  ```
  EMAIL_BACKEND=anymail.backends.amazon_ses.EmailBackend
  AWS_ACCESS_KEY_ID=your-key-here
  AWS_SECRET_ACCESS_KEY=your-secret-here
  AWS_REGION=us-east-1
  ```

### Production
- Uses AWS SES via django-anymail
- Credentials automatically generated and managed by Terraform
- IAM user with minimal SES permissions created automatically
- Email templates: `core/templates/core/emails/`

### Email Types
- Admin notifications (new user registrations)
- User approval notifications
- User rejection notifications
- All emails support retry mechanism through Django admin

## Development Notes

- Always activate the virtual environment before running Django commands
- The project uses Django 5.2.2
- Main app is called "core" - add new features here or create additional apps as needed
- Email service uses AWS SES (migrated from Brevo/Sendinblue)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pvnick) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
