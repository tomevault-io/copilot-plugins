## sharable-lists

> **This document (CLAUDE.md) must be kept up-to-date with all codebase changes.** When making any modifications to the codebase:

# Lists Sharing App - Implementation Notes

## IMPORTANT: Documentation Maintenance
**This document (CLAUDE.md) must be kept up-to-date with all codebase changes.** When making any modifications to the codebase:
- Update relevant sections to reflect new implementations
- Document new design decisions and architectural changes
- Update API endpoint documentation when adding/modifying endpoints
- Mark completed items in the Next Steps section
- Add new tasks as they are identified
- Keep the implementation status current

This ensures future agents have accurate information about the codebase state.

## Overview
This document captures all design decisions and implementation details for the lists sharing application.

## Architecture Decisions

### 1. Database Design
- **PostgreSQL** as the database backend
- **Soft deletes** implemented via `deleted_at` timestamps
- **UUID primary keys** for all entities
- **Many-to-many relationships**:
  - Users ↔ Groups (via GroupMembers)
  - Lists ↔ Entries (via ListEntries)
  - Permissions handled via separate permission tables

### 2. Authentication System
- **OAuth providers**: Google and Apple ID
- Users must create a username on first login
- Multiple OAuth providers can be linked to one account
- **API Keys**: Generated for programmatic access
  - Hashed using bcrypt for security
  - Last used timestamp tracked
- **JWT tokens** for session management

### 3. Permission System
The permission system has distinct and separate behaviors for lists and entries:

#### List Permissions
- **Owner**: Full control (can modify list metadata, manage permissions, add/remove entries)
- **Collaborator**: Can add/remove entries to/from the list (but NOT modify entry contents)
- **Viewer**: Can only view the list and see which entries are in it

#### Entry Permissions
- **Creator**: Full control over the entry contents
- **Explicit permissions**: Can be granted to users/groups
  - **View**: Can see entry contents
  - **Collaborate**: Can modify entry contents (description, notes, dates)
- **Implicit permissions**: 
  - View access ONLY if entry is in a list user can view
  - NO implicit collaborate permissions (must be explicitly granted)

**Important**: Being a list collaborator does NOT grant permission to modify entry contents. List permissions are about managing the list itself (which entries belong to it), while entry permissions are about the entry's actual content.

#### Group Permissions
- **Owner**: Can transfer ownership, delete group
- **Members**: Can add/remove other members (except owner)
- Flat hierarchy - all members have equal rights except owner

### 4. API Design
- **RESTful API** using Flask
- **Pydantic** for request/response validation
- **Type annotations** throughout the codebase
- Decorators for authentication and permission checks

### 5. Key Features Implemented

#### Entry Behavior
- Entries can belong to multiple lists
- Changes to an entry reflect in all lists
- "Deep clone" creates a completely separate copy
- Entries are automatically deleted when removed from last list

#### Soft Deletes
- All entities have `deleted_at` timestamp
- Deleted items are hidden but not removed from database
- Allows for potential recovery features

## Code Organization

### CRITICAL IMPORT RULE
**ALL imports MUST be at the top level of Python files.** This is a hard requirement for this repository. Never place imports inside functions, methods, or conditional blocks (except for TYPE_CHECKING blocks which are allowed for type hints). This ensures:
- Consistent code structure
- Easier dependency tracking
- Better performance (imports happen once at module load)
- Cleaner code organization

### Models (`app/models/`)
- `base.py`: Base model with common fields
- `user.py`: User, OAuth providers, API keys
- `group.py`: Groups and memberships
- `list.py`: Lists, tags, and list permissions
- `entry.py`: Entries, list-entry associations, entry permissions

### Schemas (`app/schemas/`)
- Pydantic models for API validation
- Separate schemas for create, update, and response
- Model validators for business logic

### Services (`app/services/`)
- `auth.py`: Authentication logic
- `oauth.py`: OAuth provider verification
- `jwt.py`: JWT token management
- `security.py`: Password and API key hashing
- `permissions.py`: Permission checking and decorators

### API Endpoints (`app/api/`)
- `auth.py`: OAuth login, API key management
- `users.py`: User profile management
- `groups.py`: Group CRUD and member management
- `lists.py`: List CRUD, permissions, tags ✅
- `entries.py`: Entry CRUD, permissions ✅

## Database Migrations
- Alembic configured for migrations
- Auto-generates from SQLAlchemy models
- To create initial migration: `uv run alembic revision --autogenerate -m "Initial migration"`
- To apply: `uv run alembic upgrade head`

## Testing Strategy
- Unit tests for all services
- Integration tests for API endpoints
- Test fixtures using Factory Boy
- Mock OAuth providers for testing

## Security Considerations
1. API keys are hashed, never stored in plaintext
2. JWT tokens have expiration
3. All user inputs validated with Pydantic
4. SQL injection prevented via SQLAlchemy ORM
5. CORS configured for specific origins
6. Soft deletes prevent accidental data loss

## Implemented API Endpoints

### Entry Endpoints (`/api/entries`)
- **POST /** - Create a new entry
- **GET /** - List accessible entries (with search, pagination)
- **GET /{entry_id}** - Get entry details
- **PUT /{entry_id}** - Update entry (requires collaborate permission)
- **DELETE /{entry_id}** - Delete entry (creator only)
- **POST /{entry_id}/clone** - Deep clone an entry
- **POST /{entry_id}/permissions** - Grant permissions (creator only)
- **DELETE /{entry_id}/permissions** - Revoke permissions (creator only)

### List Endpoints (`/api/lists`)
#### Core CRUD Operations
- **POST /** - Create a new list
- **GET /** - List accessible lists (with search, tag filtering, pagination)
- **GET /{list_id}** - Get list details
- **PUT /{list_id}** - Update list metadata (owner only)
- **DELETE /{list_id}** - Delete list (owner only)

#### Tag Management
- **POST /{list_id}/tags** - Add tag to list (owner only)
- **DELETE /{list_id}/tags** - Remove tag from list (owner only)

#### List-Entry Relationships
- **GET /{list_id}/entries** - List all entries in a list
- **POST /{list_id}/entries** - Add existing entry to list
- **POST /{list_id}/entries/new** - Create new entry and add to list
- **POST /{list_id}/entries/clone** - Clone an entry and add to list
- **DELETE /{list_id}/entries/{entry_id}** - Remove entry from list

#### Permission Management
- **GET /{list_id}/permissions** - Get list permissions (owner only)
- **POST /{list_id}/permissions** - Grant list permissions (owner only)
- **DELETE /{list_id}/permissions** - Revoke list permissions (owner only)
- **PUT /{list_id}/transfer-ownership** - Transfer list ownership (owner only)

### Group Endpoints (`/api/groups`)
- **POST /** - Create a new group
- **GET /** - List user's groups
- **GET /{group_id}** - Get group details
- **PUT /{group_id}** - Update group
- **DELETE /{group_id}** - Delete group (owner only)
- **POST /{group_id}/members** - Add member
- **DELETE /{group_id}/members/{user_id}** - Remove member
- **PUT /{group_id}/transfer-ownership** - Transfer ownership

### User Endpoints (`/api/users`)
- **GET /me** - Get current user profile
- **PUT /me** - Update current user profile
- **GET /search** - Search users by username
- **POST /link-oauth** - Link additional OAuth provider

### Authentication Endpoints (`/api/auth`)
- **POST /oauth/callback** - OAuth login callback
- **POST /logout** - Logout current session
- **POST /api-keys** - Create API key
- **GET /api-keys** - List API keys
- **DELETE /api-keys/{key_id}** - Revoke API key

## Next Steps
1. ~~Complete list API endpoints~~ ✅
2. ~~Complete entry API endpoints~~ ✅
3. Implement comprehensive test suite
4. Add database indexes for performance
5. Create API documentation (OpenAPI/Swagger)
6. Add logging and monitoring
7. Add request rate limiting
8. Implement activity feed/audit log

## Environment Variables
See `.env.example` for required configuration:
- `DATABASE_URL`: PostgreSQL connection string
- `SECRET_KEY`: Application secret key
- `JWT_SECRET_KEY`: JWT signing key
- OAuth client IDs and secrets
- CORS origins

## Running the Application
```bash
# Install dependencies
uv sync --all-extras

# Run migrations
uv run alembic upgrade head

# Start the development server
uv run flask --app app.main:create_app run --debug
```

---
**Remember**: Keep this CLAUDE.md file updated with any changes you make to the codebase!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BlazingAsher)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BlazingAsher)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
