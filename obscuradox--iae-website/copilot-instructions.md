## iae-website

> - **Rising Capital Edition: May 1–3, 2026**

# Infinity Ængines — Project Instructions

## Critical Rules

### Event Dates — ALWAYS USE THESE
- **Rising Capital Edition: May 1–3, 2026**
- **The Innovation Show happens within May 1–3, 2026 (Day 1 & Day 2)**
- Never use April 29–30 or any other dates. The event is May 1–3, 2026. Period.

### Design System
- Color palette is BLACK and WHITE only. No gold, no orange, no colored accents.
- Buttons: black/white only (white text on black bg, or black text on white bg)
- Text: white, white with opacity variants, gray shades only
- Never introduce gold (#FFD700, #D4AF37, #FFA500) into UI elements, buttons, or text gradients
- The existing Playfair Display + Inter font pairing is correct

### Brand Voice
- Infinity Ængines is a UNIQUE, ORIGINAL concept — never compare to or reference other shows (no "Shark Tank meets X", no "Oscars of Y")
- Position as the definitive, first-of-its-kind — not a derivative of anything
- Use sophisticated, status-driven language that signals exclusivity and authority
- Target demographics: founders/builders, investors/judges, global audience, sponsors — tailor messaging per segment

### Image Generation — CRITICAL
- **NEVER use Pillow** to generate pitch deck slides or marketing images
- **ALWAYS use kie.ai nano-banana-pro** API for all image generation (same model as the rest of the deck)
- Be conservative with kie.ai credits — plan generations carefully, batch when possible
- API: POST `https://api.kie.ai/api/v1/jobs/createTask`, Poll: GET `https://api.kie.ai/api/v1/jobs/recordInfo?taskId=TASK_ID`
- Use `image_input` array for reference photos (must be public URLs)
- Gallery photos available as webp at `https://www.infinityaengines.com/images/gallery/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Obscuradox)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Obscuradox)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
