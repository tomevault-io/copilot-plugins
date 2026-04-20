## cluubz

> A parent activity app for connecting children and organizing activities. Built with React frontend and Node.js/PostgreSQL backend.

# Cluubz Parent Activity App - Claude Context

## Project Overview
A parent activity app for connecting children and organizing activities. Built with React frontend and Node.js/PostgreSQL backend.

## ✅ CORRECT PROJECT STRUCTURE:
- **Working Directory**: `/home/jonathan/Claud-Clubs2/cluubz-web/` (ONLY THIS ONE)
- **Dead Projects**: `parent-activity-web` (DO NOT USE - being decommissioned)

## Key URLs
- **Staging Frontend**: https://cluubz-staging.web.app
- **Staging Backend**: https://cluubz-backend-staging-294249cc77d1.herokuapp.com
- **Production Frontend**: https://cluubz-production.web.app
- **Production Backend**: https://cluubz-backend-production-ddf3bc671afb.herokuapp.com

## Test Users
- **new700@example.com** - Current test user
- **testxy@example.com** - Used for skeleton connection testing

## 🔍 BROWSER TESTING & QA

### Playwright MCP for Automated Testing

Claude Code has access to the **Playwright Model Context Protocol (MCP)** for automated browser testing and quality assurance.

**Available Application Environments:**
- **Staging App**: https://cluubz-staging.web.app (primary testing environment)
- **Production App**: https://cluubz-production.web.app (use with caution)

**Browser Testing Capabilities:**
- ✅ **Navigate**: Open Chrome and navigate to the staging/production app
- ✅ **Interact**: Click links, buttons, input text into forms
- ✅ **Validate**: Take screenshots and verify UI rendering
- ✅ **Debug**: Check JavaScript console for runtime errors
- ✅ **Login**: Test authentication flows with test users
- ✅ **E2E Testing**: Perform end-to-end user workflows

**Example Testing Workflows:**
```
1. Login Test:
   - Navigate to https://cluubz-staging.web.app
   - Enter test user credentials (new700@example.com)
   - Verify successful login and dashboard display

2. Feature Validation:
   - Test specific features after deployment
   - Take screenshots to verify UI changes
   - Check console for JavaScript errors

3. Regression Testing:
   - Verify existing functionality still works
   - Test common user workflows
   - Validate critical paths (login, create activity, connections)
```

**When to Use Browser Testing:**
- ✅ After deploying changes to staging
- ✅ Before deploying to production
- ✅ When implementing new features
- ✅ When fixing UI bugs
- ✅ When user reports issues

**Test User Credentials for Login:**
- Email: **new700@example.com**
- Password: (user will provide if needed for testing)

## 🚨 CRITICAL DATABASE SAFETY RULES

### ⚠️ MANDATORY ENVIRONMENT PROTECTION FOR CLAUDE:

**BEFORE ANY DATABASE OPERATION, CLAUDE MUST:**
1. ✅ **VERIFY DATABASE URL**: Always confirm which database URL is being used
2. ✅ **MATCH REQUEST TO ENVIRONMENT**: Ensure the operation targets the correct environment
3. ✅ **VALIDATE ENVIRONMENT VARIABLES**: Never assume DATABASE_URL points to the right environment
4. ✅ **EXPLICIT CONFIRMATION**: When user says "production", use ONLY production URLs
5. ✅ **NO ASSUMPTIONS**: Never run scripts without explicitly setting the correct DATABASE_URL

### 🔒 ENVIRONMENT DATABASE URLS - CLAUDE MANDATORY REFERENCE:

```bash
# STAGING ONLY - cluubz-backend-staging
STAGING_DATABASE_URL="postgres://uf589tiejgjbfq:p56aef075a3fc09891e7550eaac1c8afaf4604985023985df9e0dd004340d5d8a@c7itisjfjj8ril.cluster-czrs8kj4isg7.us-east-1.rds.amazonaws.com:5432/daha3hutf78prh"

# PRODUCTION ONLY - cluubz-backend-production
PRODUCTION_DATABASE_URL="postgres://uagekj91u4hrmc:p904b26b59665bc319897558fbcceda73908268f3ffa579f13acb6ea29ab2b74a@c7itisjfjj8ril.cluster-czrs8kj4isg7.us-east-1.rds.amazonaws.com:5432/denpo4iq2ebe9i"
```

### 🚨 CLAUDE DATABASE OPERATION PROTOCOL:

**NEVER RUN DATABASE SCRIPTS WITHOUT:**
1. Setting explicit DATABASE_URL for the target environment
2. Double-checking which environment the user requested
3. Confirming the database URL matches the request (staging vs production)

**CORRECT EXAMPLES:**
```bash
# When user says "clean production clubs":
DATABASE_URL="postgres://uagekj91u4hrmc:..." node clean-production-clubs.js

# When user says "clean staging clubs":
DATABASE_URL="postgres://uf589tiejgjbfq:..." node clean-staging-clubs.js
```

**FORBIDDEN - NEVER DO THIS:**
```bash
# WRONG - relies on environment variable that could be anything
node clean-production-clubs.js
```

## 🚀 DEPLOYMENT PIPELINE

### 🚨 CRITICAL DEPLOYMENT SAFETY FOR CLAUDE:

**BEFORE ANY DEPLOYMENT, CLAUDE MUST:**
1. ✅ **VERIFY TARGET ENVIRONMENT**: Confirm staging vs production request
2. ✅ **VERIFY DEPLOYMENT COMMANDS**: Use correct Heroku app and Firebase project
3. ✅ **VERIFY BRANCH**: Ensure pushing to correct git remote
4. ✅ **DOUBLE-CHECK URLS**: Confirm deployment targets match user request

### 🔒 DEPLOYMENT COMMAND MAPPING - CLAUDE MANDATORY REFERENCE:

**⚠️ CRITICAL: Always deploy from the ROOT directory `/home/jonathan/Claud-Clubs2/`**

#### 🚨 MANDATORY: ONLY Use Deployment Scripts - NO MANUAL COMMANDS ALLOWED

**CLAUDE PRE-DEPLOYMENT CHECKLIST (CHECK EVERY TIME):**
- [ ] Am I about to run `npm run build` directly? **STOP - USE SCRIPT INSTEAD**
- [ ] Am I about to run `firebase deploy` directly? **STOP - USE SCRIPT INSTEAD**
- [ ] Am I about to run `cd cluubz-web && npm run build`? **STOP - USE SCRIPT INSTEAD**
- [ ] Did I verify I'm running the deployment SCRIPT, not manual commands?
- [ ] Is the script in the ROOT directory (not cluubz-web subdirectory)?

**✅ ONLY ALLOWED DEPLOYMENT COMMANDS:**
```bash
# STAGING - Automated deployment with safety checks
./deploy-staging.sh

# PRODUCTION - Automated deployment with confirmation prompt
./deploy-production-simple.sh
```

**❌ ABSOLUTELY FORBIDDEN - THESE WILL CAUSE ENVIRONMENT MIXUPS:**
```bash
# FORBIDDEN - bypasses safety checks, will deploy wrong backend URL
cd cluubz-web && npm run build && firebase deploy

# FORBIDDEN - bypasses env file swapping
npm run build
firebase deploy --only hosting

# FORBIDDEN - manual commands skip verification
cd cluubz-web && npm run build && cd .. && firebase use cluubz-staging && firebase deploy
```

**WHY SCRIPTS ARE MANDATORY:**
1. Scripts swap `.env.staging` ↔ `.env.production` during build
2. Scripts verify built files contain correct backend URLs
3. Scripts prevent staging from connecting to production backend
4. Manual commands will use `.env.production` (which points to PRODUCTION backend!)
5. This causes staging to hit production database - CRITICAL SAFETY ISSUE

### 🔒 CRITICAL: .env.production File Configuration

**⚠️ THE .env.production FILE CONTROLS WHICH BACKEND THE FRONTEND CONNECTS TO**

The `cluubz-web/.env.production` file is used by React's build process and **OVERRIDES** the default API_BASE_URL in the source code. This file **MUST** be set correctly for each environment:

**STAGING CONFIGURATION (Current):**
```bash
# File: cluubz-web/.env.production
NODE_ENV=production
REACT_APP_API_URL=https://cluubz-backend-staging-294249cc77d1.herokuapp.com
```

**PRODUCTION CONFIGURATION (When deploying to production):**
```bash
# File: cluubz-web/.env.production
NODE_ENV=production
REACT_APP_API_URL=https://cluubz-backend-production-ddf3bc671afb.herokuapp.com
```

**🚨 MANDATORY RULE FOR CLAUDE:**
- **BEFORE deploying to staging**: Verify `.env.production` points to **staging** backend
- **BEFORE deploying to production**: Verify `.env.production` points to **production** backend
- **DEPLOYMENT SCRIPTS**: Automatically check this and will **FAIL** if misconfigured

**Why This Matters:**
- React's `npm run build` uses `.env.production` by default
- If `.env.production` has wrong URL, staging frontend will connect to production backend (or vice versa)
- This causes 401 errors, data corruption, and environment confusion
- Deployment scripts now include safety checks to prevent this

**Safety Checks Added:**
- ✅ Pre-build verification of `.env.production` file
- ✅ Post-build verification of compiled JavaScript files
- ✅ Deployment blocked if wrong backend URL detected

#### ❌ MANUAL DEPLOYMENTS ARE FORBIDDEN

**DO NOT USE MANUAL DEPLOYMENT COMMANDS - THEY BYPASS SAFETY CHECKS**

The deployment scripts exist for a reason:
- ✅ Backend-only deployments: Use `./deploy-staging.sh` (backend is deployed first)
- ✅ Full stack deployments: Use `./deploy-staging.sh` (deploys both backend + frontend)
- ❌ Manual `git push cluubz-staging main` alone: FORBIDDEN - use script
- ❌ Manual `npm run build && firebase deploy`: FORBIDDEN - bypasses env swapping
- ❌ Manual `firebase deploy`: FORBIDDEN - will use wrong backend URL

**If deployment script is broken or unavailable:**
1. **STOP** - Do not attempt manual deployment
2. Fix the deployment script first
3. Ask user for guidance if script cannot be fixed
4. Manual deployment will cause environment mixups - NOT ALLOWED

### 🚨 DEPLOYMENT VERIFICATION PROTOCOL:

**MANDATORY CHECKS BEFORE EVERY DEPLOYMENT:**
1. User requested which environment? (staging/production)
2. Git remote matches the request?
3. Firebase project matches the request?
4. Build command appropriate for environment?
5. Heroku app name correct for environment?

**FORBIDDEN - NEVER DO THESE:**
```bash
# WRONG - could deploy to wrong environment
git push heroku main
firebase deploy

# WRONG - using wrong project
firebase use <wrong-project>
```

### ⚠️ CRITICAL DEPLOYMENT RULES FOR CLAUDE:

1. **ALL CHANGES GO TO TEST FIRST**: When user requests changes, ALWAYS deploy to test environment automatically
2. **PRODUCTION REQUIRES EXPLICIT REQUEST**: Only deploy to production when user specifically says "deploy to production"
3. **AUTO-TEST DEPLOYMENT**: After making any code changes, automatically run test deployment
4. **MANDATORY BACKEND URL CHECK**: ALWAYS verify staging URL is hardcoded in api.ts before deploying frontend

### 🧪 Staging Environment (Auto-Deploy)
**ALWAYS deploy here when making changes**

**✅ CORRECT DEPLOYMENT COMMAND:**
```bash
./deploy-staging.sh
```

**❌ DO NOT USE MANUAL COMMANDS - THEY ARE FORBIDDEN:**
```bash
# FORBIDDEN - bypasses safety checks
cd cluubz-web && npm run build && firebase deploy

# FORBIDDEN - uses wrong backend URL
git push cluubz-staging main && firebase deploy
```
- **Backend**: https://cluubz-backend-staging-8c6c18b86b0f.herokuapp.com
- **Frontend**: https://cluubz-staging.web.app
- **Users**: roberts10@example.com, roberts11@example.com, charlie@example.com

### 🚀 Production Environment (Manual Only)
**ONLY deploy when user explicitly requests "deploy to production"**

**✅ CORRECT DEPLOYMENT COMMAND:**
```bash
./deploy-production-simple.sh
```

**❌ DO NOT USE MANUAL COMMANDS - THEY ARE FORBIDDEN:**
```bash
# FORBIDDEN - bypasses safety checks
NODE_ENV=production npm run build && firebase deploy

# FORBIDDEN - uses wrong backend URL
git push cluubz-production main && firebase deploy
```

- **Backend**: https://cluubz-backend-production-ddf3bc671afb.herokuapp.com
- **Frontend**: https://cluubz-production.web.app
- **Admin**: admin@cluubz.com / Admin123!

## Development Commands

### When Making Changes (Claude Protocol)
1. Make the requested changes
2. **ALWAYS** deploy to staging using `./deploy-staging.sh` (NEVER manual commands)
3. Confirm staging deployment successful
4. Only deploy to production if explicitly requested using `./deploy-production-simple.sh`
5. **NEVER** run `npm run build` or `firebase deploy` directly - always use scripts

## Recent Issues Fixed
- ✅ Connection request acceptance (UUID vs ID mismatch)
- ✅ Multiple children selection in connection requests
- ✅ Navigation routing after adding connections
- ✅ Display names for connection requests
- ✅ Deployment safety: Staging was hitting production backend due to manual deployment bypassing env file swapping - fixed by enforcing script-only deployments (Nov 2024)

## Architecture Notes
- Frontend: React with TypeScript, Firebase Hosting
- Backend: Node.js Express, PostgreSQL on Heroku
- Database: Uses UUIDs for security, IDs for internal references
- Authentication: JWT tokens

## 🚨 CRITICAL SECURITY POLICY: UUID-ONLY FRONTEND

**MANDATORY RULE: The frontend MUST NEVER use numeric IDs for any user-facing operations.**

### ⚠️ CLAUDE MANDATORY PRE-CODE CHECKLIST:
**BEFORE WRITING ANY CODE, CLAUDE MUST:**
1. ❌ **STOP** - Am I using ANY numeric IDs? (`id`, `child_id`, `parent_id`, `activity_id`, `invitation_id`, etc.)
2. ❌ **STOP** - Are any variables, parameters, or database queries using numeric IDs?
3. ❌ **STOP** - Is any frontend code accessing `.id` properties instead of `.uuid` properties?
4. ❌ **STOP** - Are any API calls sending numeric IDs as parameters?
5. ✅ **PROCEED ONLY** if ALL database operations use UUID fields exclusively
6. ✅ **PROCEED ONLY** if ALL frontend code uses UUID properties exclusively

**IF ANY ANSWER IS YES TO 1-4: THIS IS A SECURITY VIOLATION. REWRITE USING UUIDs ONLY.**

### UUID-Only Policy
1. **ABSOLUTELY FORBIDDEN**: All numeric `id`, `child_id`, `parent_id`, `activity_id`, `invitation_id` fields in ANY frontend or backend code that Claude writes
2. **MANDATORY**: Always use UUID fields: `uuid`, `child_uuid`, `parent_uuid`, `activity_uuid`, `invitation_uuid`
3. **API CALLS**: All API endpoints must receive UUID parameters, never numeric IDs
4. **FRONTEND CODE**: Frontend must NEVER access or use `.id` properties - only `.uuid` properties
5. **DATABASE QUERIES**: All Claude-written database queries must use `WHERE uuid = $1` not `WHERE id = $1`
6. **FUNCTION PARAMETERS**: All Claude-written functions must accept UUID parameters, never ID parameters

### Conversion Guide
- `child_id` → `child_uuid` (string)
- `parent_id` → `parent_uuid` (string) 
- `activity_id` → `activity_uuid` (string)
- `invitation_id` → `invitation_uuid` (string)
- `connection_id` → `connection_uuid` (string)

### Why UUIDs Only?
- **Security**: Numeric IDs expose database structure and enable enumeration attacks
- **Privacy**: UUIDs prevent unauthorized access to other users' data
- **Scalability**: UUIDs work across distributed systems
- **Compliance**: Required for proper data protection

### Frontend Implementation - CLAUDE RULES
- **MANDATORY**: All API service methods use UUID parameters ONLY
- **MANDATORY**: All React components pass UUIDs between components ONLY  
- **MANDATORY**: All state management uses UUIDs as keys ONLY
- **MANDATORY**: All routing uses UUIDs in URL parameters ONLY
- **ABSOLUTELY FORBIDDEN**: Any frontend code accessing `.id`, `.child_id`, `.parent_id`, `.activity_id`, `.invitation_id` properties
- **ABSOLUTELY FORBIDDEN**: Any frontend code sending numeric IDs in API calls
- **CLAUDE RULE**: If I see existing frontend code using `.id` properties, I must REWRITE it to use `.uuid` properties, never "fix" the existing ID-based code

### Backend Implementation - CLAUDE RULES  
- **CLAUDE RULE**: When writing backend functions, I must use UUID-first database queries (`WHERE uuid = $1`)
- **CLAUDE RULE**: All Claude-written backend functions must accept UUID parameters, never ID parameters
- **CLAUDE RULE**: If I see existing backend code using ID-based queries, I must REWRITE it UUID-first, never "patch" ID-based code
- **BACKEND COMPATIBILITY**: Backend may use numeric IDs internally for existing database efficiency, but all NEW code by Claude must be UUID-first
- **TRANSITION PERIOD**: Backend accepts UUIDs in API endpoints
- **RESPONSE RULE**: Backend always returns both ID and UUID fields in responses, but Claude must ensure frontend only uses UUIDs

### 🔥 CLAUDE VIOLATION PATTERNS TO NEVER REPEAT:
1. **NEVER**: Write `WHERE id = $1` - Always write `WHERE uuid = $1`  
2. **NEVER**: Accept function parameters like `functionName(activityId)` - Always write `functionName(activityUuid)`
3. **NEVER**: Use `pending.activity_id` - Always write `pending.activity_uuid` or lookup UUID first
4. **NEVER**: Write `connectedChild.id` in frontend - Always write `connectedChild.uuid`
5. **NEVER**: "Fix" existing ID-based code - Always REWRITE to be UUID-first

**⚠️ CLAUDE: Any code that uses numeric IDs is a SECURITY VULNERABILITY and must be immediately REWRITTEN (not patched) to use UUIDs ONLY.**

### 📋 CLAUDE UUID-ONLY CODE PATTERNS:

#### ✅ CORRECT Backend Function Pattern:
```javascript
async function processPendingInvitations(client, connectionRequestUuid) {
    // Look up by UUID first
    const connectionRequest = await client.query(
        'SELECT * FROM connection_requests WHERE uuid = $1', 
        [connectionRequestUuid]
    );
    
    // Get activity by UUID  
    const activity = await client.query(
        'SELECT * FROM activities WHERE uuid = $1',
        [activityUuid]
    );
}
```

#### ❌ FORBIDDEN Backend Pattern (What Claude Must Never Write):
```javascript  
async function processPendingInvitations(client, connectionRequestData) {
    // WRONG: Using numeric IDs
    const query = await client.query(
        'SELECT * FROM activities WHERE id = $1',
        [pending.activity_id]  // WRONG: numeric ID
    );
}
```

#### ✅ CORRECT Frontend Pattern:
```typescript
// CORRECT: Using UUID properties only
const handleInvitation = (invitation: ActivityInvitation) => {
    apiService.respondToInvitation(invitation.invitation_uuid, response);
    navigate(`/activity/${invitation.activity_uuid}`);
};
```

#### ❌ FORBIDDEN Frontend Pattern (What Claude Must Never Write):
```typescript
// WRONG: Using numeric ID properties  
const handleInvitation = (invitation: ActivityInvitation) => {
    apiService.respondToInvitation(invitation.id, response); // WRONG
    navigate(`/activity/${invitation.activity_id}`); // WRONG  
};
```

## Database Status Values

### Connection Request Status
Connection requests use the following status values (enforced by database constraint):
- **`pending`**: Request has been sent but not yet responded to
- **`accepted`**: Request was accepted, connection is active
- **`rejected`**: Request was rejected/declined by the target parent

**Important**: Always use `rejected` status for declined requests. Do not use `declined` - this value has been migrated to `rejected` for consistency.
- the adding doesnt show now but you are not persiting the club data in tha activity fields after its selected

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonvic1891) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
