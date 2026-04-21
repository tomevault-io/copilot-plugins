## geyser-compliance-dashboard

> Essential Docker commands and workflow patterns for the Geyser Compliance Dashboard development environment.


# Docker Development Workflow

## Overview
Essential Docker commands and workflow patterns for the Geyser Compliance Dashboard development environment.

## 1. GraphQL Code Generation

### Generate Types
```bash
docker exec -ti geyser-compliance-dashboard bun run codegen
```

**When to use:**
- After modifying GraphQL queries or mutations
- After backend schema changes
- When GraphQL fragments are updated
- Always before committing GraphQL changes

**What it does:**
- Generates TypeScript types from GraphQL schema
- Updates [`src/types/generated/graphql.ts`](src/types/generated/graphql.ts)
- Creates hooks for queries and mutations
- Ensures type safety across the application

**Troubleshooting:**
- If command fails, check GraphQL syntax in query/mutation files
- Ensure backend is running and schema is accessible
- Verify network connectivity between containers

## 2. Build & Type Checking

### Production Build
```bash
docker exec -ti geyser-compliance-dashboard bun run build
```

**When to use:**
- Before committing major changes
- To verify all imports and dependencies resolve
- To catch TypeScript compilation errors
- To test production bundle creation

**What it validates:**
- All TypeScript files compile without errors
- All imports and dependencies resolve correctly
- Bundle size and optimization warnings
- Production-ready code generation

### Type Checking Only
```bash
docker exec -ti geyser-compliance-dashboard bun run type-check
```

**When to use:**
- Quick type validation without full build
- During development to catch type errors
- CI/CD pipeline validation
- Before running codegen to ensure current state is valid

## 3. Development Commands

### Install Dependencies
```bash
docker exec -ti geyser-compliance-dashboard bun install
```

**When to use:**
- After pulling changes that modify package.json
- When adding new dependencies
- If node_modules gets corrupted

### Development Server
```bash
docker exec -ti geyser-compliance-dashboard bun run dev
```

**When to use:**
- Local development with hot reload
- Testing changes in real-time
- Development mode debugging

## 4. Container Management

### Access Container Shell
```bash
docker exec -ti geyser-compliance-dashboard bash
```

**When to use:**
- Need to run multiple commands inside container
- Debugging container environment issues
- Exploring file system or logs

### Check Container Status
```bash
docker ps | grep geyser-compliance-dashboard
```

**When to use:**
- Verify container is running
- Check container health and uptime
- Troubleshoot container issues

## 5. Development Workflow

### Standard Development Cycle

1. **Pull Latest Changes**
   ```bash
   git pull origin main
   ```

2. **Install Dependencies** (if package.json changed)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun install
   ```

3. **Generate Types** (if GraphQL changed)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

4. **Make Code Changes**
   - Edit components, queries, mutations, etc.

5. **Type Check** (during development)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run type-check
   ```

6. **Full Build** (before committing)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run build
   ```

7. **Commit Changes**
   ```bash
   git add .
   git commit -m "descriptive message"
   git push
   ```

### GraphQL-Specific Workflow

1. **Modify GraphQL Files**
   - Update queries in [`src/graphql/queries/`](src/graphql/queries/)
   - Update mutations in [`src/graphql/mutations/`](src/graphql/mutations/)
   - Update fragments or add new fields

2. **Regenerate Types**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

3. **Update Components**
   - Import new types
   - Use new hooks
   - Update component logic

4. **Verify Build**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run build
   ```

## 6. Troubleshooting

### Container Not Responding
```bash
# Check if container is running
docker ps | grep geyser-compliance-dashboard

# Restart container if needed
docker restart geyser-compliance-dashboard

# Check container logs
docker logs geyser-compliance-dashboard
```

### Permission Issues
```bash
# If getting permission errors, try:
docker exec -ti --user root geyser-compliance-dashboard bash

# Then fix permissions and switch back to normal user
```

### Build Failures
1. **Clean node_modules and reinstall:**
   ```bash
   docker exec -ti geyser-compliance-dashboard rm -rf node_modules
   docker exec -ti geyser-compliance-dashboard bun install
   ```

2. **Clear build cache:**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run clean  # if available
   ```

3. **Check for TypeScript errors:**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run type-check
   ```

### GraphQL Generation Issues
1. **Check GraphQL syntax:**
   - Verify all queries/mutations have proper syntax
   - Ensure fragments are properly defined
   - Check for missing imports

2. **Verify schema access:**
   - Ensure backend is running
   - Check network connectivity
   - Verify GraphQL endpoint configuration

3. **Clear cache and retry:**
   ```bash
   docker exec -ti geyser-compliance-dashboard rm -rf src/types/generated/
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

## 7. Best Practices

### Command Execution
- Always use `-ti` flags for interactive terminal access
- Use full container name `geyser-compliance-dashboard` for clarity
- Run commands from project root directory

### Development Safety
- Always run `bun run build` before committing
- Regenerate types after any GraphQL changes
- Use type checking during development to catch errors early

### Container Management
- Don't modify files directly in container; use volume mounts
- Keep container running during development for faster command execution
- Restart container if experiencing unusual behavior

## Common Command Reference

| Task | Command |
|------|---------|
| Generate GraphQL Types | `docker exec -ti geyser-compliance-dashboard bun run codegen` |
| Build Project | `docker exec -ti geyser-compliance-dashboard bun run build` |
| Type Check | `docker exec -ti geyser-compliance-dashboard bun run type-check` |
| Install Dependencies | `docker exec -ti geyser-compliance-dashboard bun install` |
| Access Shell | `docker exec -ti geyser-compliance-dashboard bash` |
| Check Container | `docker ps \| grep geyser-compliance-dashboard` |
| View Logs | `docker logs geyser-compliance-dashboard` |
# Docker Development Workflow

## Overview
Essential Docker commands and workflow patterns for the Geyser Compliance Dashboard development environment.

## 1. GraphQL Code Generation

### Generate Types
```bash
docker exec -ti geyser-compliance-dashboard bun run codegen
```

**When to use:**
- After modifying GraphQL queries or mutations
- After backend schema changes
- When GraphQL fragments are updated
- Always before committing GraphQL changes

**What it does:**
- Generates TypeScript types from GraphQL schema
- Updates [`src/types/generated/graphql.ts`](src/types/generated/graphql.ts)
- Creates hooks for queries and mutations
- Ensures type safety across the application

**Troubleshooting:**
- If command fails, check GraphQL syntax in query/mutation files
- Ensure backend is running and schema is accessible
- Verify network connectivity between containers

## 2. Build & Type Checking

### Production Build
```bash
docker exec -ti geyser-compliance-dashboard bun run build
```

**When to use:**
- Before committing major changes
- To verify all imports and dependencies resolve
- To catch TypeScript compilation errors
- To test production bundle creation

**What it validates:**
- All TypeScript files compile without errors
- All imports and dependencies resolve correctly
- Bundle size and optimization warnings
- Production-ready code generation

### Type Checking Only
```bash
docker exec -ti geyser-compliance-dashboard bun run type-check
```

**When to use:**
- Quick type validation without full build
- During development to catch type errors
- CI/CD pipeline validation
- Before running codegen to ensure current state is valid

## 3. Development Commands

### Install Dependencies
```bash
docker exec -ti geyser-compliance-dashboard bun install
```

**When to use:**
- After pulling changes that modify package.json
- When adding new dependencies
- If node_modules gets corrupted

### Development Server
```bash
docker exec -ti geyser-compliance-dashboard bun run dev
```

**When to use:**
- Local development with hot reload
- Testing changes in real-time
- Development mode debugging

## 4. Container Management

### Access Container Shell
```bash
docker exec -ti geyser-compliance-dashboard bash
```

**When to use:**
- Need to run multiple commands inside container
- Debugging container environment issues
- Exploring file system or logs

### Check Container Status
```bash
docker ps | grep geyser-compliance-dashboard
```

**When to use:**
- Verify container is running
- Check container health and uptime
- Troubleshoot container issues

## 5. Development Workflow

### Standard Development Cycle

1. **Pull Latest Changes**
   ```bash
   git pull origin main
   ```

2. **Install Dependencies** (if package.json changed)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun install
   ```

3. **Generate Types** (if GraphQL changed)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

4. **Make Code Changes**
   - Edit components, queries, mutations, etc.

5. **Type Check** (during development)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run type-check
   ```

6. **Full Build** (before committing)
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run build
   ```

7. **Commit Changes**
   ```bash
   git add .
   git commit -m "descriptive message"
   git push
   ```

### GraphQL-Specific Workflow

1. **Modify GraphQL Files**
   - Update queries in [`src/graphql/queries/`](src/graphql/queries/)
   - Update mutations in [`src/graphql/mutations/`](src/graphql/mutations/)
   - Update fragments or add new fields

2. **Regenerate Types**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

3. **Update Components**
   - Import new types
   - Use new hooks
   - Update component logic

4. **Verify Build**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run build
   ```

## 6. Troubleshooting

### Container Not Responding
```bash
# Check if container is running
docker ps | grep geyser-compliance-dashboard

# Restart container if needed
docker restart geyser-compliance-dashboard

# Check container logs
docker logs geyser-compliance-dashboard
```

### Permission Issues
```bash
# If getting permission errors, try:
docker exec -ti --user root geyser-compliance-dashboard bash

# Then fix permissions and switch back to normal user
```

### Build Failures
1. **Clean node_modules and reinstall:**
   ```bash
   docker exec -ti geyser-compliance-dashboard rm -rf node_modules
   docker exec -ti geyser-compliance-dashboard bun install
   ```

2. **Clear build cache:**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run clean  # if available
   ```

3. **Check for TypeScript errors:**
   ```bash
   docker exec -ti geyser-compliance-dashboard bun run type-check
   ```

### GraphQL Generation Issues
1. **Check GraphQL syntax:**
   - Verify all queries/mutations have proper syntax
   - Ensure fragments are properly defined
   - Check for missing imports

2. **Verify schema access:**
   - Ensure backend is running
   - Check network connectivity
   - Verify GraphQL endpoint configuration

3. **Clear cache and retry:**
   ```bash
   docker exec -ti geyser-compliance-dashboard rm -rf src/types/generated/
   docker exec -ti geyser-compliance-dashboard bun run codegen
   ```

## 7. Best Practices

### Command Execution
- Always use `-ti` flags for interactive terminal access
- Use full container name `geyser-compliance-dashboard` for clarity
- Run commands from project root directory

### Development Safety
- Always run `bun run build` before committing
- Regenerate types after any GraphQL changes
- Use type checking during development to catch errors early

### Container Management
- Don't modify files directly in container; use volume mounts
- Keep container running during development for faster command execution
- Restart container if experiencing unusual behavior

## Common Command Reference

| Task | Command |
|------|---------|
| Generate GraphQL Types | `docker exec -ti geyser-compliance-dashboard bun run codegen` |
| Build Project | `docker exec -ti geyser-compliance-dashboard bun run build` |
| Type Check | `docker exec -ti geyser-compliance-dashboard bun run type-check` |
| Install Dependencies | `docker exec -ti geyser-compliance-dashboard bun install` |
| Access Shell | `docker exec -ti geyser-compliance-dashboard bash` |
| Check Container | `docker ps \| grep geyser-compliance-dashboard` |
| View Logs | `docker logs geyser-compliance-dashboard` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/geyserfund) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
