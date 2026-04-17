## magic-auth-dashboard

> Below is a concise, human-readable reference of every REST endpoint defined in the supplied OpenAPI document.

Below is a concise, human-readable reference of every REST endpoint defined in the supplied OpenAPI document.  
For each route you’ll find the HTTP method, a short description (summary + key points from the longer description) and the request parameters (path, query, and/or body-form fields).

> • “(req)” = required • Location is shown in parentheses → path | query | form-body  
> • When a route accepts both Bearer-header and secure HTTP-only cookies, only the business parameters are listed.

---

# 3-Tier User-Type Multi-Project Auth API – Endpoint Reference

## 1. Authentication
### POST  /auth/login
Authenticate user and start a session (sets HTTP-only cookie).  
Params  
• username (form, req) • password (form, req) • project_hash (form, optional – root may omit)

### POST  /auth/register
Register a new user and auto-assign default group.  
Params  
• username (form, req) • password (form, req) • email (form, optional) • project_hash (form, req)

### GET  /auth/validate
Validate current session token and return user & group context.  
Params – none

### POST  /auth/logout
Invalidate the current session & clear cookie.  
Params – none

### POST  /auth/refresh
Refresh JWT, extending the session.  
Params – none

### POST  /auth/switch-project
Switch active project in an existing session.  
Params  
• project_hash (form, req)

### POST  /auth/check-availability
Check if a username or e-mail is free globally.  
Params  
• username (form, optional) • email (form, optional)

---

## 2. User Management
### GET  /users/profile
Return current user profile, groups & accessible projects.  
Params – none

### PUT  /users/profile
Update own username / email / password.  
Params  
• username (form, optional) • email (form, optional) • password (form, optional)

### GET  /users/access-summary
Short access summary (groups, projects, permissions).  
Params – none

### GET  /users
List users with filters (admin/root only).  
Query params  
• limit (default 50) • offset (default 0) • user_type • search • is_active

### GET  /users/{user_hash}
Get details for a specific user (self or admin/root scope).  
Path param • user_hash

### PATCH  /users/{user_hash}/status
Activate / deactivate a user (admin).  
Path • user_hash Form • is_active (req)

### POST  /users/{user_hash}/reset-password
Admin reset => returns temporary password.  
Path • user_hash

### PATCH  /users/{user_hash}/type
ROOT-only change of user-type (root, admin, consumer).  
Path • user_hash Form • user_type (req)

---

## 3. User-Type Management
### POST  /user-types/root
Create a new root (super-admin).  
Form • username (req) • password (req) • email (optional)

### POST  /user-types/admin
Create an admin for one / many projects.  
Form • username (req) • password (req) • email (req) • assigned_project_id OR assigned_project_ids

### GET  /user-types/{user_hash}/info
Comprehensive type info for a user.  
Path • user_hash

### PUT  /user-types/{user_hash}/type
ROOT-only promotion/demotion + project assignment.  
Path • user_hash Form • user_type (req) • assigned_project_id (optional)

### GET  /user-types/users/{user_type}
List all users of a given type.  
Path • user_type Query • limit • offset

### PUT  /user-types/admin/{user_hash}/projects
Overwrite the list of projects an admin manages.  
Path • user_hash Form • assigned_project_ids (req)

### GET  /user-types/admin/{user_hash}/projects
Fetch all projects an admin currently manages.  
Path • user_hash

### POST  /user-types/admin/{user_hash}/projects/add
Add an extra project to an admin.  
Path • user_hash Form • project_id (req)

### DELETE  /user-types/admin/{user_hash}/projects/{project_id}
Remove admin from a specific project.  
Path • user_hash • project_id

### GET  /user-types/stats
System-wide distribution of user types.  
Params – none

---

## 4. Project Management
### GET  /projects
List projects visible to caller.  
Query • limit (1-100, default 10) • offset • search

### POST  /projects
Create a new project (admin/root).  
Form • project_name (req) • project_description (optional)

### GET  /projects/{project_hash}
Detailed project info with caller’s access context.  
Path • project_hash

### PUT  /projects/{project_hash}
Update project metadata (admin).  
Path • project_hash Form • project_name • project_description

### DELETE  /projects/{project_hash}
Delete / revoke access to a project (admin).  
Path • project_hash

### GET  /projects/{project_hash}/members
List members of a project.  
Path • project_hash Query • limit • offset • user_type

### POST  /projects/{project_hash}/members
Add a member to a project with role.  
Path • project_hash Form • user_hash (req) • role (default consumer)

### DELETE  /projects/{project_hash}/members/{user_hash}
Remove member from project.  
Path • project_hash • user_hash

### GET  /projects/{project_hash}/activity
Project activity feed (phase 2).  
Path • project_hash Query • limit • offset • activity_type • days

### GET  /projects/{project_hash}/stats
Detailed stats (phase 2).  
Path • project_hash

### PATCH  /projects/{project_hash}/owner
Transfer ownership (admin).  
Path • project_hash Form • new_owner_hash (req)

### PATCH  /projects/{project_hash}/archive
Archive / unarchive a project.  
Path • project_hash Form • archived (req, bool)

---

## 5. RBAC Management
### GET  /rbac/projects/{project_hash}/permissions
List permissions in project.  
Path • project_hash Query • category • limit • offset

### POST  /rbac/projects/{project_hash}/permissions
Create a permission.  
Path • project_hash Form • permission_name (req) • category (default general) • description

### GET  /rbac/projects/{project_hash}/roles
List roles in project.  
Path • project_hash Query • limit • offset

### POST  /rbac/projects/{project_hash}/roles
Create role (permission group).  
Path • project_hash Form • group_name (req) • priority (default 50) • description • permissions (list)

### POST  /rbac/projects/{project_hash}/initialize
Bootstrap default perms & roles for a project.  
Path • project_hash Query • create_default_permissions (bool) • create_default_roles (bool)

### GET  /rbac/projects/{project_hash}/summary
Full RBAC summary.  
Path • project_hash

### GET  /rbac/projects/{project_hash}/audit
Audit log for RBAC actions.  
Path • project_hash Query • limit • offset • action_type

### POST  /rbac/users/{user_hash}/projects/{project_hash}/roles
Assign user to role.  
Path • user_hash • project_hash Form • role_id (req)

### GET  /rbac/users/{user_hash}/projects/{project_hash}/roles
List user’s roles in project.  
Path • user_hash • project_hash

### DELETE  /rbac/users/{user_hash}/projects/{project_hash}/roles/{role_id}
Remove user from role (phase 2).  
Path • user_hash • project_hash • role_id

### GET  /rbac/users/{user_hash}/projects/{project_hash}/permissions
Effective permissions for a user in project.  
Path • user_hash • project_hash

### GET  /rbac/users/{user_hash}/projects/{project_hash}/check/{permission_name}
Check if user has specific permission.  
Path • user_hash • project_hash • permission_name

---

## 6. Admin – User Groups
### GET  /admin/user-groups
List global user groups.  
Query • limit • offset

### POST  /admin/user-groups
Create a user group.  
Form • group_name (req) • description

### GET  /admin/user-groups/{group_hash}
Group details.  
Path • group_hash

### PUT  /admin/user-groups/{group_hash}
Update group.  
Path • group_hash Form • group_name • description

### DELETE  /admin/user-groups/{group_hash}
Delete group.  
Path • group_hash

### POST  /admin/user-groups/{group_hash}/members
Add user to group.  
Path • group_hash Form • user_hash (req)

### DELETE  /admin/user-groups/{group_hash}/members/{user_hash}
Remove user from group.  
Path • group_hash • user_hash

### POST  /admin/user-groups/{group_hash}/projects
Grant group access to project.  
Path • group_hash Form • project_hash (req)

### DELETE  /admin/user-groups/{group_hash}/projects/{project_hash}
Revoke group’s project access.  
Path • group_hash • project_hash

---

## 7. Admin – Project Groups
### GET  /admin/project-groups
List permission-groups reusable across projects.  
Query • limit • offset

### POST  /admin/project-groups
Create project-group with permissions list.  
Form • group_name (req) • permissions (list) • description

### GET  /admin/project-groups/{group_hash}
Details of a project-group.  
Path • group_hash

### PUT  /admin/project-groups/{group_hash}
Update project-group.  
Path • group_hash Form • group_name • permissions • description

### DELETE  /admin/project-groups/{group_hash}
Delete project-group.  
Path • group_hash

### POST  /admin/project-groups/{group_hash}/projects
Assign a project to the group.  
Path • group_hash Form • project_hash (req)

### DELETE  /admin/project-groups/{group_hash}/projects/{project_hash}
Remove project from group.  
Path • group_hash • project_hash

---

## 8. Admin Dashboard  _(system stats & health)_
• GET /admin/dashboard/stats – Overall counters  
• GET /admin/activity – Activity feed (filters: limit, offset, activity_type, user_id, project_id, days)  
• GET /admin/health – Deep health check  
• GET /admin/activity/types – List of activity categories  
• GET /admin/users/statistics – User growth & type breakdown (days)  
• GET /admin/projects/statistics – Project metrics (days)  
• GET /admin/system/overview – System-wide performance & health overview

---

## 9. Analytics
• GET /analytics/dashboard/stats (period_days)  
• GET /analytics/users (period_days, user_type)  
• GET /analytics/projects (period_days)  
• GET /analytics/activity (period_days, activity_type)  
• GET /analytics/summary

---

## 10. System Info & Cache
• GET /system/info – system summary  
• GET /system/health – comprehensive health  
• GET /system/ping – lightweight ping  
• GET /system/cache/stats – cache metrics  
• POST /system/cache/clear – clear all cache  
• POST /system/cache/invalidate/user/{user_hash}  
• POST /system/cache/invalidate/project/{project_id}

---

## 11. Bulk Operations (Phase 2)
• POST /admin/users/bulk-update – change many users  
• POST /admin/users/bulk-delete – delete many users  
• POST /admin/projects/{project_hash}/bulk-assign-roles  
• POST /admin/user-groups/bulk-assign  
• POST /rbac/projects/{project_hash}/bulk-assign _(RBAC)_

---

## 12. Low-Level Access Check
### HEAD  /access
Verifies JWT presence & permission for arbitrary resources (uses custom headers X-token-user / X-token-collection).  
Params – none

### GET  /ping  (root level)
Simple service-alive check.  

---

**Tip:** All endpoints secured by `HTTPBearerOrCookie` accept either an Authorization Bearer header **or** the secure session cookie automatically issued by the authentication endpoints.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Andres77872) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
