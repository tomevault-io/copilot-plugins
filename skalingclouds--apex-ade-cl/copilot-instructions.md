## apex-ade-cl

> Critical features and requirements for Apex ADE

# Critical Features & Requirements

## Core Functionality
1. **Large PDF Support**: Up to 1GB files with chunking
2. **Dynamic Field Selection**: AI-powered field discovery
3. **Human-in-the-Loop Review**: Dual-pane interface
4. **Batch Processing**: Handle thousands of documents
5. **Schema Library**: Save/reuse extraction schemas

## Extraction Requirements
- **Primary Field**: APEX ID (format: "25USOA...")
- Multi-field extraction must work reliably
- Always show data in clean table format
- Never display raw JSON or markdown

## UI/UX Requirements
- Side-by-side PDF and extraction view
- Approve/Reject/Escalate workflow
- Next/Back navigation for batch review
- Dark mode by default
- Responsive and fast

## Chat Features
- GPT-5 powered Q&A about documents
- Highlight answers in PDF with bounding boxes
- Context-aware responses
- Chat history persistence

## Technical Constraints
- Use GPT-5 exclusively (no other OpenAI models)
- Frontend port 3000, Backend port 8000
- SQLite database (migrations via Alembic)
- Local file storage in `./uploads`

## Error Handling
- Graceful fallbacks for extraction failures
- Retry mechanisms with exponential backoff
- User-friendly error messages
- Escalation options for failed extractions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skalingclouds)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/skalingclouds)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
