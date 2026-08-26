## micky

> Persian-first voice assistant. The bet: AI software got too complicated. Micky is a small companion you talk to, not a workspace. Say the thing, get the work done, go back to life.

# Micky — agent notes

Persian-first voice assistant. The bet: AI software got too complicated. Micky is a small companion you talk to, not a workspace. Say the thing, get the work done, go back to life.

If a feature turns Micky into a workspace, a dashboard, or a text-first chat app, it probably does not belong here. The conversation archive should still feel like part of the small companion.

## Shape

Electron app. Renderer is a 400×712 orb: wake / listen / think / reply. Past conversations live in a compact archive reached from the footer; there is no permanent sidebar or text composer.

| Path                     | Role                                                                                               |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| `src/`                   | Vite React UI. Mic capture, earcons, status text, and the conversation archive. No business logic. |
| `src/lib/`               | Shared types and constants used by both processes.                                                 |
| `electron/`              | Main process. All real work.                                                                       |
| `electron/conversation/` | Turn machine: `idle` → `agent` → `confirm` → `followup` → idle.                                    |
| `electron/system/`       | Path guard, file tools, command policy, Seatbelt sandbox.                                          |
| `electron/wake-word/`    | Local “هی میکی” detector (ONNX worker).                                                            |
| `electron/speech/`       | Local Shenava ASR (`sherpa-onnx` child process).                                                   |
| `electron/agent/`        | OpenRouter tool loop. Short spoken Persian replies.                                                |
| `electron/chats/`        | Local SQLite conversation persistence and full-text search.                                        |
| `electron/soul/`         | Markdown layers: soul, user profile, memory.                                                       |
| `electron/llm/`          | Model + OpenRouter key (OS keychain).                                                              |

IPC lives in `electron/preload.ts` / `src/lib/desktop-api.ts`. Renderer talks only through `window.api`.

## Voice loop

1. Wake word (or tap the orb) → chime → ASR session.
2. Final transcript → `ConversationController` → `AgentService.respond`.
3. On complete: 12s follow-up listen (empty ASR endpoints do **not** kill the window).
4. Silence or `end_conversation` → back to wake word. Agent context is cleared only on end / «گفتگوی تازه».

The first completed user turn creates a local chat. A chat stays active across reloads and follow-up windows, and rolls over after 30 minutes of inactivity. The archive can resume a chat by loading its recent turns back into the agent context.

ASR is messy: no punctuation, broken words, English as Persian phonetics. Design for that. Never surface “corrected” transcripts or ask the user to type.

## Agent + tools

Tools are in `electron/agent/tools.ts`. Status copy for the orb is in `agentStatusLabel` (`src/lib/agent.ts`). Memory: `remember`, `recall`, `update_user_profile`. Past chats: `search_chats`, `read_chat`. Clock: `get_current_datetime`. Session: `end_conversation`. System (kill switch in settings): `read_file`, `list_directory`, `search_files`, `search_in_files`, `open_app`, `run_command`. The orb shows a one-line Persian summary, never raw arguments; tap a confirm prompt to reveal the command. Risky commands need a spoken yes, enforced outside the model.

Replies are **speech**: 1–3 short sentences, no markdown/emoji/lists. Tool guidance and the voice contract live in `electron/soul/prompt.ts` — keep them locked unless the product voice changes.

## How to add something

- Prefer a tool + a one-line Persian status over a new view. The compact conversation archive is the deliberate exception.
- Wire new session behavior through `ConversationController`, not the React tree.
- Keep the home shell: orb, a few words, footer icons.
- `pnpm test` and `pnpm typecheck` after behavior changes. Tests are `*.test.ts` next to the code (`tsx --test`).
- Do not add: a permanent sidebar, a text-chat composer, plugin systems, extra cloud STT, dashboards, or onboarding steps that are not required to hear or answer.

---
> Source: [xmannii/micky](https://github.com/xmannii/micky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
