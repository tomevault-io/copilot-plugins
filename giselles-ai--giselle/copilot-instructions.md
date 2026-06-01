## design-mode

> User request `design mode`, use this rule to enable design mode for all files.

# Design Mode

Design Mode is intended for users who excel in design rather than software development. It enables direct application design by editing Next.js code instead of using tools like Figma.

- **Avoid destructive changes.**
- Limit modifications to appearance or behavior only.
- If functional changes or additions are necessary, create a consultation text for engineers and encourage the user to seek their input.

---

## How to Start Design Mode

The step-by-step procedure below **MUST** be followed in order. Each step **MUST** be completed before proceeding to the next one. No steps should be skipped.

### 1. Check Requirements

All required tools must be verified in the following order:

1. **Node.js Version Check**
   ```bash
   node -v
   ```
   ✅ REQUIRED: Version must be 22.14.0 or later
   ❌ If version requirement not met: See `./nodejs.mdc` for installation instructions

2. **pnpm Version Check**
   ```bash
   pnpm -v
   ```
   ✅ REQUIRED: Version must be 10.2.1 or later
   ❌ If version requirement not met: See `./nodejs.mdc` for installation instructions

3. **Vercel CLI Version Check**
   ```bash
   vercel --version
   ```
   ✅ REQUIRED: Version must be 41.6.0 or later
   ❌ If Vercel CLI is missing: Install with the command below
   ```bash
   pnpm add -g vercel
   ```

4. **Vercel CLI Login Status**
   ```bash
   vercel whoami
   ```
   ✅ REQUIRED: Must show a username
   ❌ If not logged in: See `./vercel-cli.mdc` for login instructions

5. **Vercel Project Link Check**
   ```bash
   [ -f .vercel/project.json ] && echo "Project is linked" || echo "We need to link vercel project"
   ```
   ✅ REQUIRED: Must show "Project is linked"
   ❌ If "We need to link vercel project" is displayed:
      a. List available teams:
         ```bash
         vercel team list
         ```
      b. Switch to team and list projects:
         ```bash
         vercel team switch [team-name]
         vercel project list
         ```
      c. Link the project:
         ```bash
         vercel link --yes --project [project-name]
         ```

### 2. Clean Your Workspace

IMPORTANT: Complete each step in order:

1. **Preview Files to be Removed**
   ```bash
   git clean -fx -n
   ```
   ✅ REQUIRED: Review the output and get user approval before proceeding
   ❌ If the user doesn't want these files removed: Skip the cleaning step but note potential issues

2. **Current Branch Decision**
   Ask the user if they want to:
   - Continue in the current branch
   - Switch to the main branch with:
     ```bash
     git checkout main && git pull
     ```

3. **Install Dependencies**
   ```bash
   pnpm i
   ```
   ✅ REQUIRED: Wait for installation to complete successfully

4. **Build SDK Dependencies**
   ```bash
   pnpm build-sdk
   ```
   ✅ REQUIRED: Wait for build to complete successfully

5. **Check Port 3000 Availability - CRITICAL STEP**
   ```bash
   lsof -i :3000 || echo "Port 3000 is available"
   ```
   ✅ REQUIRED: Must either:
   - Show "Port 3000 is available" OR
   - If a process is using port 3000, terminate it:
     ```bash
     kill -9 $(lsof -t -i:3000)
     ```
   - Verify port is free after termination:
     ```bash
     lsof -i :3000 || echo "Port 3000 is now available"
     ```

### 3. Start Design Mode

After completing ALL previous steps, start the development server:

```bash
vercel dev
```

The server should start successfully and display:
```
> Ready! Available at http://localhost:3000
```

---

## Design Mode Checklist Summary

Use this checklist to ensure all steps are completed:

- [ ] Node.js version verified (22.14.0+)
- [ ] pnpm version verified (10.2.1+)
- [ ] Vercel CLI version verified (41.6.0+)
- [ ] Vercel CLI login confirmed
- [ ] Vercel project link confirmed
- [ ] Workspace files previewed for removal
- [ ] Branch decision made (current or main)
- [ ] Dependencies installed
- [ ] SDK dependencies built
- [ ] Port 3000 verified as available
- [ ] Development server started with vercel dev

---
> Source: [giselles-ai/giselle](https://github.com/giselles-ai/giselle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
