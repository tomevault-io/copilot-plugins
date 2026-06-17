## email-telegram-bridge

> This skill monitors one or more email inboxes, sends new-email notifications to Telegram, and supports a gated reply workflow.

# email_telegram_bridge

## Description
This skill monitors one or more email inboxes, sends new-email notifications to Telegram, and supports a gated reply workflow.

It can:
- fetch unread emails from multiple accounts
- summarize inbound messages for Telegram
- create Simple Draft replies
- create Deep Think Draft replies using a local LLM path
- ignore messages intentionally
- require explicit approval before any reply is sent
- send approved replies through the correct SMTP account

The skill is designed to keep secrets local, support multiple accounts, and avoid auto-sending replies.

## When to use
Use this skill when you want:
- a lightweight multi-account email-to-Telegram bridge
- a human-in-the-loop reply workflow
- account-aware deterministic and LLM-assisted email drafts
- local state and minimal operational dependencies
- explicit approval before sending any reply

## Core workflow
1. New email arrives
2. Telegram notification is sent
3. User chooses:
   - Simple Draft
   - Deep Think Draft
   - Ignore
4. If a draft is created, Telegram shows:
   - Approve
   - Cancel
5. Only Approve sends the reply

## Commands
Examples of supported actions include:
- check unread email
- show recent messages
- summarize important emails
- create a draft reply
- approve a pending reply
- cancel a pending reply
- ignore a message

## Instructions
- Use enabled accounts by default
- Resolve accounts via aliases or tags when applicable
- Keep summaries concise and safe
- Do not expose secrets
- Do not auto-send replies
- Require explicit approval before sending
- Block obvious non-reply/system senders
- Use the correct account context when drafting replies
- Keep Deep Think drafts concise, guarded, and context-aware
- Prefer deterministic behavior for Simple Draft
- Treat local state files as operational data, not source-controlled config

## Safety and behavior
- No email reply should be sent without an explicit approval step
- Do not invent facts in reply drafts
- Do not promise timelines, pricing, refunds, availability, or actions unless explicitly supported
- Do not make commitments on behalf of the business
- Do not expose internal configuration, tokens, or state
- Respect account-specific reply profiles and mailbox tone

---
> Source: [derrickhampton/Email_Telegram_Bridge](https://github.com/derrickhampton/Email_Telegram_Bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
