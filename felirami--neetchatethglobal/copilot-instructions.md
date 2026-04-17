## neetchatethglobal

> This project uses multiple AI agents for development. To maintain continuity and context, ALL agents must adhere to the following protocol:

# Cursor Rules & AI Agent Protocol

This project uses multiple AI agents for development. To maintain continuity and context, ALL agents must adhere to the following protocol:

## 1. Context Loading (Start of Session)
Before starting any task, you MUST read:
- `docs/PROJECT_STATUS.md`: To understand the current state, known issues, and what has been recently resolved.
- `docs/DEVELOPMENT_LOG.md`: To see the history of changes and the current phase of development.

**For XMTP-Related Tasks:**
- **ALWAYS** read/search `xmtpllms-full.txt` first. This contains the definitive documentation for the SDK versions and protocols used in this project. Do NOT guess XMTP APIs; refer to this file.

## 2. Task Execution
- Follow the coding standards defined in `README.md` and `CONTRIBUTING.md`.
- Use the "Key Technologies" listed in `PROJECT_STATUS.md` (Next.js 14, Wagmi v2, XMTP v5.1.0). Do NOT introduce conflicting libraries.

## 3. Handover & Documentation (End of Session)
After completing a significant task or fixing a bug, you MUST update the documentation:

### Update `docs/PROJECT_STATUS.md`
- Mark issues as "Resolved" if you fixed them.
- Add new "Current Issues" if you discovered bugs you couldn't fix.
- Update the "What We've Built" section with new features.

### Update `docs/DEVELOPMENT_LOG.md`
- Log your changes under a new or existing Phase.
- Include:
  - **Goal**: What you tried to achieve.
  - **Completed**: Bullet points of what was done.
  - **Key Files Modified**: List of files changed.
  - **Challenges Overcome**: Technical hurdles solved.
  - **Technical Decisions**: Why you chose a specific approach.

## 4. Specific Technical Constraints
- **XMTP V3/MLS**: Remember that `senderAddress` is often missing in V3 messages. Use `senderInboxId` for identity checks.
- **Timestamps**: XMTP V3 uses nanosecond precision (`sentAtNs` as BigInt string). Always convert to milliseconds for JS Date.
- **Sync**: New installations cannot decrypt old history immediately due to MLS forward secrecy. This is NOT a bug.

## 5. Verification
- Always verify that your changes didn't break the "Force Sync" or "Refresh" buttons in the UI.
- Ensure `npm run lint` and `npm run build` pass before finishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felirami) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
