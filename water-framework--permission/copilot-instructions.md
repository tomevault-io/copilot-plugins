## permission

> Manages the **Permission** layer of the Water Framework's RBAC system. Permissions are the atomic access grants that bind a **role** (or user) to a set of **actions** on a specific **entity type** (optionally scoped to a single instance). The `DefaultPermissionManager` evaluates permissions at runtime when `@AllowPermissions` interceptors fire.

# Permission Module — Fine-Grained Access Control

## Purpose
Manages the **Permission** layer of the Water Framework's RBAC system. Permissions are the atomic access grants that bind a **role** (or user) to a set of **actions** on a specific **entity type** (optionally scoped to a single instance). The `DefaultPermissionManager` evaluates permissions at runtime when `@AllowPermissions` interceptors fire.

## Sub-modules

| Sub-module | Runtime | Key Classes |
|---|---|---|
| `Permission-api` | All | `PermissionApi`, `PermissionSystemApi`, `PermissionRestApi`, `PermissionRepository` |
| `Permission-model` | All | `WaterPermission`, `PermissionActions` |
| `Permission-service` | Water/OSGi | Service impl, repository, REST controller, `DefaultPermissionManager` |
| `Permission-service-spring` | Spring Boot | Spring MVC REST controllers |

## WaterPermission Entity

```java
@Entity
@Table(name = "water_permission",
       uniqueConstraints = @UniqueConstraint(
           columnNames = {"roleId", "userId", "entityResourceName", "resourceId"}))
public class WaterPermission extends AbstractJpaEntity implements ProtectedEntity {

    @NotNull @NoMalitiusCode
    private String name;

    @NotNull
    private long actionIds;               // bitmask of allowed actions

    @NotNull @NoMalitiusCode
    private String entityResourceName;    // fully-qualified entity class name

    private long resourceId;              // 0 = all instances; >0 = specific instance

    private long roleId;                  // exclusive with userId
    private long userId;                  // exclusive with roleId
}
```

## Bitmask Action Encoding

Actions from `@AccessControl.availableActions` are encoded as powers of 2:

| Action | Value |
|---|---|
| `SAVE` | 1 |
| `UPDATE` | 2 |
| `FIND` | 4 |
| `FIND_ALL` | 8 |
| `REMOVE` | 16 |
| Custom (e.g., ASSIGN) | 32, 64, 128... |

A permission granting SAVE + FIND has `actionIds = 1 | 4 = 5`.

## DefaultPermissionManager

Evaluates whether the current `SecurityContext` user has the required permission:

```java
// Called by @AllowPermissions interceptor
boolean checkPermission(String entityClassName, long entityId, String action);

// Returns per-action permission map for the logged-in user
Map<String, Boolean> entityPermissionMap(String entityClassName, long entityId);
```

Resolution algorithm:
1. Get user's roles from `RoleManager`
2. For each role, find `WaterPermission` where `entityResourceName = className` AND (`resourceId = 0` OR `resourceId = entityId`) AND `roleId = role.id`
3. Check if `actionIds & actionBitmask != 0`
4. Also check direct user permissions (`userId = currentUserId`)
5. Admin users (`WaterUser.admin = true`) always pass

## Permission Lifecycle

```
@AccessControl on entity → defines available actions + default role-action mapping
         ↓
Framework bootstrap → creates WaterPermission records for @DefaultRoleAccess entries
         ↓
Runtime: @AllowPermissions interceptor → DefaultPermissionManager.checkPermission()
         ↓
        PASS: proceed with method
        FAIL: throw UnauthorizedException
```

## Key Interfaces

### PermissionApi / PermissionSystemApi
```java
WaterPermission save(WaterPermission permission);
WaterPermission update(WaterPermission permission);
WaterPermission find(long id);
PaginatedResult<WaterPermission> findAll(int delta, int page, Query filter);
void remove(long id);

// Convenience: grant/revoke specific action to role
void addPermission(long roleId, String entityClass, long entityId, String action);
void removePermission(long roleId, String entityClass, long entityId, String action);
```

## EntityPermissionMap Feature
Computes a `Map<String, Boolean>` of all possible actions for the current user on a given entity instance. Used by REST clients to render UI controls (e.g., hide "Delete" button if `REMOVE=false`):

```java
// REST endpoint
GET /water/permissions/entityPermission?entityClass=it.water.user.model.WaterUser&entityId=42

// Response
{
  "SAVE": false,
  "UPDATE": true,
  "FIND": true,
  "FIND_ALL": true,
  "REMOVE": false
}
```

## REST Endpoints

| Method | Path | Permission |
|---|---|---|
| `POST` | `/water/permissions` | permissionManager |
| `PUT` | `/water/permissions` | permissionManager |
| `GET` | `/water/permissions/{id}` | permissionViewer |
| `GET` | `/water/permissions` | permissionViewer |
| `DELETE` | `/water/permissions/{id}` | permissionManager |
| `GET` | `/water/permissions/entityPermission` | authenticated |

## Security Annotations Quick Reference

```java
// Check entity-level action permission (on Api service methods)
@AllowPermissions(actions = {"SAVE"}, checkById = false)

// Check permission on the entity returned by the method
@AllowPermissionsOnReturn(actions = {"READ"})

// Check a non-entity named permission
@AllowGenericPermissions(permissions = {"reports.generate"})

// Restrict to specific roles only
@AllowRoles(roles = {"admin", "reportManager"})
```

## Default Roles

| Role | Allowed Actions |
|---|---|
| `permissionManager` | SAVE, UPDATE, FIND, FIND_ALL, REMOVE |
| `permissionViewer` | FIND, FIND_ALL |
| `permissionEditor` | UPDATE, FIND, FIND_ALL |

## Dependencies
- `it.water.repository.jpa:JpaRepository-api` — `AbstractJpaEntity`
- `it.water.core:Core-permission` — `DefaultPermissionManager`, `PermissionUtil`, `CrudActions`
- `it.water.role:Role-api` — `RoleManager` for role-based permission resolution
- `it.water.rest:Rest-persistence` — `BaseEntityRestApi`

## Testing
- Unit tests: `WaterTestExtension` — test permission CRUD + `DefaultPermissionManager` evaluation
- Permission scenario pattern:
  ```java
  // Create permission for role
  WaterPermission perm = new WaterPermission("test", CrudActions.SAVE_VALUE | CrudActions.FIND_VALUE,
                                              MyEntity.class.getName(), 0, roleId, 0);
  permissionApi.save(perm);

  // Test as user with that role
  TestRuntimeInitializer.getInstance().impersonate(targetUser, runtime);
  Assertions.assertDoesNotThrow(() -> myApi.save(entity));

  // Restore admin
  TestRuntimeUtils.impersonateAdmin(componentRegistry);
  ```
- REST tests: **Karate only** — never JUnit direct calls to `PermissionRestController`

## Code Generation Rules
- Permission records for default roles are created via `@DefaultRoleAccess` at framework bootstrap — never create them in `@OnActivate` manually
- Always use bitmask ORing for multi-action permissions (`SAVE | FIND = 1 | 4 = 5`)
- `resourceId = 0` means the permission applies to ALL instances of the entity type
- `PermissionRestController` tested **exclusively via Karate**
- Never call `DefaultPermissionManager` directly in business code — use the `@AllowPermissions` annotation on service method

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Water-Framework)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Water-Framework)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
