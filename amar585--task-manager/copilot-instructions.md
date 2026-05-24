## task-manager

> - This is a Django-based web application with multiple apps

# Nexus Project Rules

## Project Structure

- This is a Django-based web application with multiple apps
- Core functionality is in the 'tasks' app
- Authentication is handled by the 'auth_app' app with Supabase integration
- Templates are in both app-specific templates/ dirs and project-level templates/
- The main project is in 'mysite/'
- AI integration appears to be in 'chatbot_integration/'

## Coding Patterns

- Models use UUID primary keys throughout
- Model relationships are extensive (e.g., ForeignKey, ManyToMany)
- Django signals used for event handling
- Django forms used for data validation and processing
- Most views follow a pattern of form validation → business logic → response
- Template inheritance is used with base.html as the foundation
- Form-based filtering with GET parameters for search and filtering

## Naming Conventions

- Snake case for Python variables and functions (e.g., `task_detail`)
- PascalCase for model and class names (e.g., `TaskComment`)
- Models use singular names (e.g., `Task` not `Tasks`)
- URL patterns generally follow RESTful conventions
- Template names match their corresponding view function (e.g., `task_detail.html`)

## Database Patterns

- UUID primary keys used for models
- Created/updated timestamps on most models
- Status fields often use choices
- Soft deletion with archived flags rather than hard delete
- Many-to-many relationships for task assignments, project membership

## UI Patterns

- Bootstrap-based responsive design
- Card-based UI components
- Fade-in animations for visual elements
- Consistent color scheme for status and priority
- Icon usage with Bootstrap Icons
- Collapsible sections for advanced functionality
- Basic vs. Advanced interface separation for progressive disclosure

## Authentication

- Hybrid authentication with Supabase and Django
- Webhook-based user synchronization
- JWT tokens for API authentication
- Session-based authentication for web interface

## Important Implementation Details

- Task status transitions have specific methods (`mark_completed`, etc.)
- Project progress is calculated based on completed tasks
- Tasks can have complex relationships (dependencies, parent-child)
- Custom fields allow for flexible task attributes
- Task activity is logged for audit and history
- Advanced search uses form.cleaned_data for robust validation
- Search filters persist across pagination via query_params

## Search and Filtering

- TaskSearchForm defines all available filtering criteria
- Filter by text, status, priority, dates, assignees, etc.
- Search within specific content fields (title, description, comments)
- Range-based filtering for numeric fields (estimated hours)
- Client-side filter state management using JavaScript
- Filter persistence across pagination
- Support for multiple sorting options

## Known Quirks

- Some CSS issues exist in the dashboard progress bar
- UUID-based models may have performance implications at scale
- Complex relationship traversal requires careful query optimization
- Authentication state must be synchronized between Django and Supabase
- Django template syntax causes JavaScript linter errors
- Complex search queries may impact performance at scale

## User Preferences

- Clean, minimalist UI preferred
- Animations should be subtle and non-disruptive
- Performance is a priority, especially for data-heavy views
- Mobile responsiveness is essential
- Progressive disclosure for complex features 

---
> Source: [Amar585/Task_Manager](https://github.com/Amar585/Task_Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
