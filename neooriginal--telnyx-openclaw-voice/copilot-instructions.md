## telnyx-openclaw-voice

> Instructions for an AI to initiate and manage phone calls via the TelnyxOpenClaw API.


# Telnyx Outgoing Call Skill

This skill allows an AI assistant to initiate phone calls to users and carry out specific tasks via a telephone interface.

## Capabilities
- Initiate a phone call to a specific number.
- Provide a `task` that the call AI will use to introduce itself.
- Automatically handle the transition from the task introduction to a back-and-forth conversation.

## API Documentation

BASE URL: http://localhost:3023 (may differ)

### POST `/call`
Initiates an outgoing call.

**Request Body:**
```json
{
  "task": "The specific reason for the call (e.g., 'Remind the user about their dentist appointment')",
  "to": "+1234567890" (optional, defaults to DEFAULT_TO_NUMBER in .env)
}
```

**Workflow for the AI:**
1. **Identify Need**: When you need to contact a user via phone (e.g., for a reminder, urgent notification, or follow-up).
2. **Formulate Task**: Write a concise task description. The call AI will use this to generate its own greeting.
3. **Execute**: Send a POST request to the `/call` endpoint.

## How it works internally
1. The server receives the task.
2. An LLM generates a friendly, task-specific intro (e.g., "Hello, I'm calling to remind you about...").
3. The call is placed via Telnyx.
4. When answered, the intro is played.
5. The system then enters a recording loop, allowing for a natural conversation.
6. The user can press any key (DTMF) to stop a recording and force the AI to respond immediately.

---
> Source: [neooriginal/Telnyx-OpenClaw-Voice](https://github.com/neooriginal/Telnyx-OpenClaw-Voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
