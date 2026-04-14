## home-ai-assistant

> This file provides custom instructions for the agent when working on this project.

# Project Agent Instructions

This file provides custom instructions for the agent when working on this project.

## Command Execution Safety

**CRITICAL:** Never run commands that will hang or require waiting for completion unless explicitly instructed to do so.

1. **Always use `bg_bash` for background processes** - Never use `&` at the end of commands
2. **Check command success immediately** - After running `bg_bash`, immediately verify it started successfully
3. **Validate before proceeding** - Only move forward after confirming the background process is running
4. **Handle stuck commands** - If a background command appears stuck, kill it and restart with timeouts if needed
5. **Do not wait for long-running processes** - Background processes should start and exit quickly (within 5-10 seconds)

## Frontend Testing

When running end-to-end frontend tests with Playwright:

1. **Check if servers are running first:**
   - Backend: Check with `lsof -ti:PORT` (use the backend port from `docker-compose.yml`)
   - Frontend: Check with `lsof -ti:3000` (or the port configured in `playwright.config.ts`)

2. **If servers are running:**
   - Run tests directly without starting servers
   - Example: `cd frontend && npm run test:e2e`

3. **If servers are NOT running:**
   - Start backend server first using `bg_bash "source .venv/bin/activate && uvicorn app.main:app --host 0.0.0.0 --port 8000"`
   - For frontend, if it's a Next.js app, you can either:
     - Let Playwright start it automatically (recommended - configured in `playwright.config.ts` with `reuseExistingServer: false`)
     - Run it in background using `bg_bash` tool instead of `&` (e.g., `bg_bash "npm run dev"` or `bg_bash "npm run dev > /tmp/frontend-dev.log 2>&1"`)

4. **Timeout warning:**
   - E2E tests can take 1-5 minutes to complete
   - DO NOT run `npm run test:e2e` if you're going to wait for completion
   - If stuck or hanging, you may need to:
     - Kill the test process (Ctrl+C)
     - Check logs in `/tmp/frontend-dev.log`
     - Verify server health before retrying

5. **After tests:**
   - Check test results in `frontend/test-results/`
   - View HTML report: `npx playwright show-report`

## Backend Testing

When testing backend services:

1. Always use the virtual environment: `source .venv/bin/activate`
2. Run tests with: `pytest` or the appropriate test command
3. If testing with uvicorn, ensure you terminate after 5 seconds
4. **Starting backend server for testing:**
   - Use `bg_bash "source .venv/bin/activate && uvicorn app.main:app --host 0.0.0.0 --port 8000"` to start in background
   - **Immediately check success** with `lsof -ti:8000` to verify the process started
   - Check logs with `cat /tmp/<process-name>.log` if needed
5. **Terminating backend server:**
   - Use `pkill -f "uvicorn app.main:app"` to stop the server
6. **Important:** Always check if the background process started successfully before proceeding

## bg_bash Command Best Practices

1. **Always verify success after bg_bash:**
   ```bash
   # Run the command
   bg_bash "command"

   # Immediately verify it started
   lsof -ti:PORT || echo "Process not running"
   ```

2. **If bg_bash appears stuck:**
   - Kill the process: `pkill -f "command_name"`
   - Restart with a timeout if needed
   - Do NOT wait for completion

3. **Never assume background commands succeed** - Always validate with `lsof` or process checks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maty0609) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
