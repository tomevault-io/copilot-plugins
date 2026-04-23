## weekly-arcade

> <!-- nx configuration start-->

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

<!-- nx configuration end-->

# Weekly Arcade Project Guidelines

## Project Structure

- `apps/web-astro/` - Frontend (Astro 6, deployed to Firebase Hosting)
- `apps/api/` - Backend (NestJS on Cloud Run)
- `packages/shared/` - Shared types and constants

## API Client Usage

The frontend uses `window.apiClient` (from `/js/api-client.js`) for all backend communication.

### Authentication Required

All write operations require the user to be signed in. Always check auth state before API calls:

```javascript
// Add auth state tracking to your game
let currentUser = null;

// Initialize auth when ready
const authCheckInterval = setInterval(() => {
  if (window.authManager?.isInitialized) {
    clearInterval(authCheckInterval);
    window.authManager.onAuthStateChanged(user => {
      currentUser = user;
    });
  }
}, 100);

// Check before submitting
if (!currentUser || !window.apiClient) {
  console.log('User not signed in');
  return;
}
```

### Score Submission (Leaderboard)

**IMPORTANT:** The API validates score submissions strictly. Only these fields are allowed:

```javascript
await window.apiClient.submitScore('game-id', {
  score: number,           // REQUIRED - the score value
  guessCount?: number,     // Optional - number of attempts
  level?: number,          // Optional - current level
  timeMs?: number,         // Optional - time in milliseconds
  wordHash?: string,       // Optional - for word games validation
  metadata?: {             // Optional - ANY extra data goes here
    wave: number,
    deck: string,
    // ... any custom fields
  }
});
```

**Common Mistakes:**
- DO NOT add custom fields at the top level (e.g., `wave`, `deck`)
- ALL custom/game-specific data must go inside `metadata`
- Score must be a number, not an object

**Correct Example (Fieldstone):**
```javascript
await window.apiClient.submitScore('fieldstone', {
  score: gameState.score,
  metadata: {
    wave: gameState.wave,
    deck: gameState.selectedDeck
  }
});
```

**Correct Example (Wordle):**
```javascript
await window.apiClient.submitScore('wordle', {
  score: scoreData.score,
  level: wordLength - MIN_LENGTH + 1,
  guessCount: scoreData.attempts,
  timeMs: scoreData.timeMs || 0,
  metadata: {
    wordLength: wordLength,
    hardMode: hardMode
  }
});
```

### Other API Endpoints

```javascript
// Get leaderboard
await window.apiClient.getLeaderboard('game-id', 'daily'); // 'daily', 'weekly', 'monthly'

// Save game state
await window.apiClient.saveGameState('game-id', stateObject);

// Load game state
const state = await window.apiClient.getGameState('game-id');

// Unlock achievement
await window.apiClient.unlockAchievement('achievement-id', 'game-id', { metadata });
```

## Adding a New Game (Astro)

1. Create game data: `apps/web-astro/src/data/games/{game-name}.json`
2. Create game page: `apps/web-astro/src/pages/games/{game-name}.astro` (uses `GameLayout.astro`)
3. Create game JS: `apps/web-astro/public/games/{game-name}/game.js`
4. Add to service worker cache in `apps/web-astro/public/sw.js`
5. Register in `packages/shared/src/lib/constants/game-registry.ts`
6. Add score validation config in `apps/api/src/leaderboard/config/game-config.ts`
7. Include auth integration (see above)

## Service Worker / PWA Cache

**IMPORTANT:** Whenever you modify ANY file under `apps/web-astro/` (games, JS, pages), you MUST increment `CACHE_VERSION` in `apps/web-astro/public/sw.js`. This is required for PWA users to receive updates.

```javascript
// In apps/web-astro/public/sw.js — increment this number
const CACHE_VERSION = 24; // ← bump this on every change
```

## Deployment

- Frontend: Auto-deploys to Firebase Hosting on push to main
- Backend: Auto-deploys to Cloud Run on push to main

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HimanshuSingh2308) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
