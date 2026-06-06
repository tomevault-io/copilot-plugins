## server-api

> Generate a server full API


# Server API Rules

## Overview
A complete API resource consists of:
1. A controller to handle HTTP requests/responses (using [server-api-controller.mdc](mdc:.cursor/rules/server-api-controller.mdc))
2. A repository for database operations (using [server-api-repository.mdc](mdc:.cursor/rules/server-api-repository.mdc))
3. Domain entity type definitions if not already existing (using [server-domain-entity.mdc](mdc:.cursor/rules/server-domain-entity.mdc) )
4. DTO types for requests and responses (defined in [server-api-controller.mdc](mdc:.cursor/rules/server-api-controller.mdc))

## Implementation Process
To create a new API resource, follow these steps:

1. **Create Resource Directory**:
   ```
   src/server/api/{resource-name}/
   ```

2. **Generate Domain Entity** (if needed):
   Create a domain entity following the [server-entity.mdc](mdc:.cursor/rules/server-entity.mdc) rules:
   ```
   src/server/domain/{resource-name}.type.ts
   ```

3. **Create Repository**:
   Generate a repository using [server-repository.mdc](mdc:.cursor/rules/server-repository.mdc) rule :
   ```
   src/server/api/{resource-name}/{resource-name}.repository.ts
   ```

4. **Create Controller**:
   Generate a controller following [server-controller.mdc](mdc:.cursor/rules/server-controller.mdc)  with appropriate endpoints:
   ```
   src/server/api/{resource-name}/{resource-name}.controller.ts
   ```

5. **Create DTOs**:
   Add request/response DTOs as described in [server-controller.mdc](mdc:.cursor/rules/server-controller.mdc) 
   ```
   src/server/api/{resource-name}/{resource-name}-response.type.ts
   src/server/api/{resource-name}/{resource-name}-{action}-request.type.ts
   ```

6. **Register in API Controller**:
   Update `/src/server/api/api.controller.ts` to include the new resource.

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
