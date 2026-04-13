## recursion

> 🚨 **MANDATORY AI CODING ASSISTANT RULES - NO EXCEPTIONS** 🚨

🚨 **MANDATORY AI CODING ASSISTANT RULES - NO EXCEPTIONS** 🚨

⚠️ **CRITICAL**: These rules are FREQUENTLY IGNORED - PAY ATTENTION! ⚠️

- **🔧 TOOLS - STRICT REQUIREMENTS**

  - 🛑 **MANDATORY**: Use Bun for package management (NOT npm, NOT yarn)
  - 🛑 **MANDATORY**: Fix TypeScript errors after ALL changes

- **📝 CODE CHANGES - ZERO TOLERANCE POLICY**

  - ✅ **ONLY modify relevant code parts** - Do NOT touch unrelated code
  - ✅ **PRESERVE ALL**: formatting, names, and documentation unless EXPLICITLY requested
  - ✅ **FOLLOW EXISTING PATTERNS**: Refer to existing similar code structure when generating new code (components, API routes, utilities, types, assets)

- **📋 PROJECT MANAGEMENT - ABSOLUTELY REQUIRED**

  - 🔴 **MANDATORY**: Use TODO.md for tasks, progress, and issues. Update regularly - NO EXCEPTIONS
  - 🔴 **SESSION START CHECKLIST**: review TODO.md, run `git status`, check recent commits - DO NOT SKIP

- **⚡ DEVELOPMENT PROCESS - ENFORCE STRICTLY**

  - 🛑 **REQUIRED**: Plan and discuss approaches before coding - NO RUSHING
  - 🛑 **REQUIRED**: Make small, testable changes - NO BIG CHANGES
  - 🛑 **REQUIRED**: Eliminate duplicates proactively
  - 🛑 **REQUIRED**: Log recurring issues in TODO.md - ALWAYS DOCUMENT

- **🔒 CODE QUALITY - NON-NEGOTIABLE STANDARDS**

  - ✅ **MANDATORY**: Handle errors and validate inputs - NO EXCEPTIONS
  - ✅ **MANDATORY**: Follow conventions and secure secrets - NEVER EXPOSE SECRETS
  - ✅ **MANDATORY**: Write clear, type-safe code - NO SHORTCUTS
  - ✅ **PRODUCTION RULE**: Remove ALL debug logs before production - CLEAN CODE ONLY

- **📐 DEVELOPMENT STANDARDS - ABSOLUTE REQUIREMENTS**
  - 🎯 **PRIORITY #1**: Simplicity and readability over clever solutions
  - 🎯 **APPROACH**: Start with minimal working functionality - BUILD INCREMENTALLY
  - 🎯 **CONSISTENCY**: Maintain consistent style throughout - NO STYLE MIXING

🔥 **FINAL WARNING**: If you violate these rules, you are COMPLETELY IGNORING the project standards!

## API Architecture
- **Proxy configuration**:
  - Development: Vite proxy routes requests to microservices (see vite.config.ts)
  - Production: Nginx reverse proxy handles routing (see infrastructure/nginx.conf)
- **Service ports**:
  - Auth service: 3000
  - Business logic service: 3001
  - File storage service: 3002
- **DO NOT use absolute URLs with ports** - Let the proxy handle routing

## When to Rebuild/Restart Services

**You MUST restart services when:**
- ❗ **TypeScript compilation errors** - The service won't start with errors
- ❗ **Adding/removing middleware or routes** - Route registration happens at startup
- ❗ **Changing environment variables** - They're loaded at startup
- ❗ **Modifying Prisma schema** - Run `bun prisma generate` then restart
- ❗ **Installing new dependencies** - Run `bun install` then restart
- ❗ **Changing service configuration** - Port numbers, database connections, etc.

**You DON'T need to restart for:**
- ✅ **Route handler logic changes** - Hot reload handles these
- ✅ **Service/utility function changes** - Hot reload handles these
- ✅ **Frontend changes** - Vite has its own hot reload
- ✅ **CSS/styling changes** - Handled by Vite HMR
- ✅ **React component changes** - Handled by Vite HMR

**Quick Commands:**
```bash
# Check if services are running
lsof -i :3000  # Auth service
lsof -i :3001  # Business logic
lsof -i :3002  # File storage

# Restart all services
./stop-local-services.sh
./start-local-services.sh

# Check logs if service fails to start
cat logs/auth-service.log
cat logs/business-logic-service.log
cat logs/file-storage-service.log
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdu211171)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jdu211171)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
