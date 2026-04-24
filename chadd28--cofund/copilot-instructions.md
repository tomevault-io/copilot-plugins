## cofund

> – Always prefer simple solutions

# Coding pattern preferences

– Always prefer simple solutions  
-Always use best SWE practices
– Avoid duplication of code whenever possible, which means checking for other areas of the codebase that might already have similar code and functionality  
– Write code that takes into account the different environments: dev, test, and prod  
– You are careful to only make changes that are requested or you are confident are well understood and related to the change being requested  
– When fixing an issue or bug, do not introduce a new pattern or technology without first exhausting all options for the existing implementation. And if you finally do this, make sure to remove the old implementation afterwards so we don’t have duplicate logic.  
– Keep the codebase very clean and organized  
– Avoid writing scripts in files if possible, especially if the script is likely only to be run once  
– Avoid having files over 200–300 lines of code. Refactor at that point.  
– Mocking data is only needed for tests, never mock data for dev or prod  
– Never add stubbing or fake data patterns to code that affects the dev or prod environments  
– Never overwrite my .env file without first asking and confirming
– Make sure that everything you do in the frontend is able to be seamlessly integrated with how the backend is configured.
–After major components, write a brief summary in /docs/[component].md (e.g., login.md)
- Modify only the code I specify; leave everything else untouched.
- Break large tasks into stages; pause after each for my approval.
- Before big changes, tell me the plan and await my confirmation.
- After each component, summarize what’s done.
- Classify changes as Small, Medium, or Large.
- If my request is unclear, ask me before proceeding.
- Follow SOLID principles (e.g., single responsibility, dependency inversion) where applicable.
- Begin every response with a random emoji (e.g., 🐙) to confirm context retention.
- Optimize outputs to minimize token usage without sacrificing clarity.
- Never alter the stack without my explicit approval
-Do not make any changes, until you have 95% confidence that you know what to build
ask me follow up questions until you have that confidence

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chadd28) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
