## ai-agentic-mar28

> AI personal assistant that learns from Gmail, shops Amazon, orders DoorDash autonomously


# Agentic Second Brain

An AI-powered personal assistant with 3 agents, 15 autonomous tools, and 2-pass vision that:

- Connects to Gmail and extracts food/shopping order history into a structured database
- Suggests food based on most-ordered restaurants (weighted random — highest ordered = highest priority)
- Opens a real browser to add products to your Amazon cart
- Opens DoorDash to order food from your favourite restaurants
- Identifies products from photos using 2-pass Gemini Vision and finds them on Amazon
- Finds past purchases in Gmail and helps you reorder
- Sends emails on your behalf
- Supports voice input and voice output
- Runs scheduled agents daily to sync new orders and validate data accuracy

## Setup

1. Clone: `git clone https://github.com/sandeepvijayarao09/Agentic-Second-Brain`
2. Install: `pip install -r requirements.txt`
3. Copy `.env.example` to `.env` and add your GEMINI_API_KEY and Google OAuth credentials
4. Run: `uvicorn api.main:app --port 8080`
5. Connect Gmail: visit `http://localhost:8080/auth/gmail/authorize`
6. Sync orders: click "Sync Gmail" in the UI or `POST /sync/gmail`
7. Chat: open `http://localhost:8080` and start talking

## Agents

| Agent | Type | Tools | Description |
|---|---|---|---|
| SecondBrain | Chat | 15 | Main conversational agent — food, shopping, Gmail, vision, voice |
| EmailSyncAgent | Scheduled | 1 | Checks Gmail daily for new order confirmations |
| FavouritesValidator | Scheduled | 0 | Recomputes favourites accuracy from order data |

## Tech Stack

Railtracks Agentic Framework, Google Gemini 2.5 Flash Lite, FastAPI, SQLAlchemy, Playwright, Gmail API, DigitalOcean App Platform, assistant-ui, Augment Code

## Live Demo

https://octopus-app-2ceva.ondigitalocean.app

## GitHub

https://github.com/sandeepvijayarao09/Agentic-Second-Brain

---
> Source: [sandeepvijayarao09/AI-Agentic-Mar28](https://github.com/sandeepvijayarao09/AI-Agentic-Mar28) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
