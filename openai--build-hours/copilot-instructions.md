## build-hours

> - Top-down Phaser sandbox where the player chats with nearby NPCs acting as LLM-backed mentors inside OpenAI HQ.

# Agents Handbook

## What We're Building

- Top-down Phaser sandbox where the player chats with nearby NPCs acting as LLM-backed mentors inside OpenAI HQ.
- Each NPC has its own dialogue agent with custom instructions and rolling memory so conversations resume where they left off.
- The UI keeps the last two thread messages visible, color-codes player (green) vs NPC (white), and supports live streaming responses.

## Core Pieces

- `src/scenes/MainScene.ts`: Manages player movement, proximity detection, interaction prompts, and the threaded chat UI. Handles switching between NPCs, cancelling with ESC or by walking away, and consuming streamed responses.
- `src/gameobjects/Character.ts`: Wrapper around a Phaser sprite + optional `DialogueAgent` configuration and physics helpers.
- `src/dialogue/DialogueAgent.ts`: Streams responses from OpenAI (or falls back to canned text), stores per-NPC message history, and exposes helpers for display name, prefixes, and initial greetings.

## Implementation Notes

- Conversations stream token-by-token. `MainScene` updates the latest NPC entry as chunks arrive and only resumes input once streaming completes.
- `DialogueAgent` records every user/assistant turn; conversations should never be manually truncated unless intentionally resetting state.
- NPC configs currently live inline in `MainScene` (`npcConfigs`). Expand these or load from data files if you need richer casts.
- The default `DialogueAgent` constructor bakes in a temporary API key; replace it with environment-driven configuration before shipping anything real.

## Open TODOs & Watchouts

- Populate `npcConfigs` with real assets/positions/instructions; current entries are placeholders.
- Swap the hardcoded OpenAI key and model for injected configuration or per-NPC overrides.
- Interaction radius (`interactionRadius = 72`) may feel aggressive when NPCs are close together; tweak or add prioritization if players see prompt flicker.
- Double-check UI teardown: leaving range or cancelling should hide both dialogue boxes and reset `threadMessages`.

## Testing & Manual QA

- No automated tests yet. Manually verify:
    - Switching between multiple NPCs without dropping ongoing history.
    - Walking away mid-stream cancels gracefully and lets streaming finish silently.
    - ESC always exits the active conversation and clears the UI.
- Consider adding integration tests (Playwright + Phaser headless) if time allows.

## Future Extensions

- Wire real LLM calls via dependency injection (pass an `OpenAI` client per NPC or through a factory).
- Persist conversation history to storage so it survives reloads.
- Replace hardcoded art/assets with data-driven definitions or a content pipeline.
- Add message throttling/length guards before sending to the LLM.

Keep the UX snappy: conversations should feel like texting with a coach. Preserve the last-two-message window and streaming feel whenever you touch dialogue logic.

---
> Source: [openai/build-hours](https://github.com/openai/build-hours) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
