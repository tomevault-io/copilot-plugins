## auth0-nextjs-bootstrap

> >-


# Auth0 Next.js Bootstrap

## Goal

Create a fresh `auth0-demo` Next.js App Router project, install the official
Auth0 React SDK, create/configure an Auth0 Single Page Application in the
dashboard using Chrome, wire the app with the captured tenant domain/client ID,
and leave the user ready to test login.

Use the bundled script `scripts/apply-auth0-next-app.mjs` to write the React
Auth0 provider/page files after the Auth0 domain and client ID are known.

## Required Chrome Skill

For all Auth0 dashboard work, use the Chrome plugin skill
`chrome:control-chrome`.

Do not hardcode a local plugin cache path. The Chrome plugin is installed under
user-, platform-, and version-specific paths such as
`~/.codex/plugins/cache/openai-bundled/chrome/<version>/...`, and that location
will differ across machines.

Resolve Chrome this way:

1. If the user explicitly mentioned or linked `chrome:control-chrome`, use that
   skill from the current context.
2. If it is listed in the available skills, open its `SKILL.md` using the path
   supplied by the runtime for this session.
3. If the skill is not visible but `tool_search` is available, search for
   `chrome control-chrome` or `node_repl js` to expose the Chrome/browser-client
   workflow.
4. If Chrome still is not available, report that the Chrome plugin must be
   installed/enabled and stop. Do not fall back to the in-app Browser for Auth0
   dashboard edits that require the user's signed-in Chrome session.

After resolving the skill, follow its bootstrap exactly, read the full browser
documentation it returns, and use the Chrome extension browser. Do not substitute
generic web browsing, macOS `open`, or a separate browser tool for Auth0
dashboard edits.

Normal no-human-intervention operation assumes Chrome already has a valid Auth0
dashboard session. If Auth0 presents login, MFA, CAPTCHA, or an account chooser
that cannot be completed from the current Chrome session, stop and report the
blocked state.

## Workflow

1. Pick a project directory and exact local origin.
   - Default project command:
     ```bash
     npx create-next-app@latest auth0-demo --ts --app --empty --eslint --use-npm --yes --disable-git
     ```
   - Default origin is `http://localhost:3000`.
   - If port 3000 is occupied by an unrelated process before Auth0 is configured,
     choose the next free port and use that same origin everywhere.

2. Install the official SDK:
   ```bash
   cd auth0-demo
   npm install @auth0/auth0-react@2.x
   ```

3. Configure Auth0 in Chrome.
   - Navigate to `https://manage.auth0.com/dashboard`.
   - Create a new application named `auth0-demo` as a Single Page Application.
   - If an `auth0-demo` application already exists in the selected tenant, reuse
     it only when the user asked for this demo tenant/app or when creating a new
     app with the same name is not possible.
   - In the application Settings tab, ensure:
     - Application Type: `Single Page Application`
     - Token Endpoint Authentication Method: `None`
     - Allowed Callback URLs: chosen origin
     - Allowed Logout URLs: chosen origin
     - Allowed Web Origins: chosen origin
   - Save, reload the settings page, and verify the three origin fields persisted.
   - Capture:
     - Domain, for example `dev-tenant.auth0.com`
     - Client ID

4. Patch the app using the bundled script:
   ```bash
   # Set SKILL_DIR to the directory containing this SKILL.md.
   node "$SKILL_DIR/scripts/apply-auth0-next-app.mjs" \
     --project-dir . \
     --domain "<AUTH0_DOMAIN>" \
     --client-id "<AUTH0_CLIENT_ID>" \
     --origin "<LOCAL_ORIGIN>"
   ```

5. Validate locally:
   ```bash
   npm run lint
   npm run build
   ```

6. Start or report the dev command:
   ```bash
   npm run dev -- --port <PORT>
   ```
   Tell the user it is time to test at the exact configured origin. Do not run a
   full email/password login test unless the user asks.

## Chrome Dashboard Details

Use visible page state as source of truth. Prefer Playwright locators from the
latest DOM snapshot. After every save, reload the settings page and inspect the
field contents again. The key success condition is that Callback, Logout, and
Web Origins all show the exact same origin that the local dev server will use.

If login later returns:

`access_denied` plus `connect ECONNREFUSED 127.0.0.1:3000`

then Auth0 is executing a tenant-side Action/Rule/Hook/custom database script
that calls localhost from Auth0's cloud runtime. Inspect Logs first, then report
the offending tenant-side customization instead of changing unrelated app code.

---
> Source: [grp06/auth0-nextjs-bootstrap](https://github.com/grp06/auth0-nextjs-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
