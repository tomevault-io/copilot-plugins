## onlineclinicprompt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains system prompt materials for an AI chatbot assistant for **onlineclinic.ge**, a Georgian telemedicine platform. The chatbot helps users book medical consultations, select subscription plans, and get information about healthcare services.

## Repository Structure

- `system_prompt.md` - Main system prompt for the AI assistant (English)
- `frequently_asked_questions.txt` - FAQ content in Georgian language
- `*.jpeg` - Reference images (website screenshots, UI mockups, design references)

## Key Context

### Bilingual Requirements
The chatbot operates in both **Georgian and English**. When working with content:
- Georgian text uses the Georgian alphabet (ქართული)
- Always preserve the language structure when editing
- The assistant must detect and respond in the user's language

### Medical Context Restrictions
The system prompt enforces strict non-diagnostic boundaries:
- NEVER provide medical diagnoses, prescriptions, or treatment advice
- Always direct users to professional consultations
- Emergency cases → direct to 112 (Georgia's emergency number)
- Urgent cases → offer 30-minute consultation booking

### Core Service Information

**Subscription Plans (5 tiers):**
- Standard: 40 GEL/month (3 free consultations)
- Standard Plus: 50 GEL/month (4 free consultations)
- Premium: 180 GEL/month (unlimited)
- Children: 30 GEL/month (unlimited, under 18)
- Family: 75 GEL/month (unlimited for up to 3 members)

**Contact Information:**
- Mobile: +995 544 112 888
- Landline: +995 32 2 112 888
- Email: info@onlineclinic.ge

## Editing Guidelines

When modifying the system prompt:

1. **Maintain bilingual consistency** - If you update English content, check if corresponding Georgian content exists in `frequently_asked_questions.txt`

2. **Preserve pricing structure** - The subscription plan table is critical for user guidance. Any pricing changes must be accurate and synchronized.

3. **Keep medical boundaries clear** - The "Critical Restrictions" and "Conversation Guidelines" sections define liability boundaries. Changes here may have legal implications.

4. **Preserve conversation flow logic** - The plan selection algorithm (based on frequency of consultations) is intentionally designed to optimize user value.

## Common Tasks

**Update pricing:**
Edit the subscription plans table in `system_prompt.md:20-28`

**Add new FAQ:**
Add to both `system_prompt.md` (English) and `frequently_asked_questions.txt` (Georgian) to maintain parity

**Modify response guidelines:**
Update the "Conversation Guidelines" section in `system_prompt.md:77-101`

**Update contact information:**
Change all instances in `system_prompt.md:35-39` and verify consistency throughout the prompt

---
> Source: [Vkvara/onlineClinicPrompt](https://github.com/Vkvara/onlineClinicPrompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
