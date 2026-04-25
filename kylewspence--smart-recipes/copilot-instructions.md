## smart-recipes

> This is a full stack AI-powered recipe app called Smart Recipes, built for a coding bootcamp final project.


This is a full stack AI-powered recipe app called Smart Recipes, built for a coding bootcamp final project.

Tech stack:
	•	Frontend: Next.js (App Router) with TypeScript and Tailwind CSS
	•	Backend: Express.js with TypeScript (separate server, running on port 3001)
	•	Database: PostgreSQL
	•	AI Integration: OpenAI API

Folder conventions:
	•	Frontend lives in app/ or src/app/
	•	Backend code lives in server/
	•	Shared types go in server/types/index.ts
	•	Utilities go in utils/ or lib/

Coding standards:
	•	Use async/await, not .then()
	•	Always add error handling for database and API operations
	•	Modularize large functions or route handlers
	•	Prioritize readable, clean code over brevity

Validation:
	•	All user input must be validated before processing—prefer Zod schemas
	•	Do not trust req.body or OpenAI responses without validation
	•	Sensitive info (like passwords or tokens) must never be logged

OpenAI usage:
	•	AI recipes must honor user preferences, allergies, dislikes, and stretch preferences
	•	The OpenAI prompt should be generated dynamically from user data
	•	Save the generatedPrompt with each recipe for traceability

App purpose reminder:
This app helps picky eaters find meals they’ll actually enjoy, based on their preferences and available ingredients. The goal is a clean UX and smart, personalized AI suggestions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kylewspence) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
