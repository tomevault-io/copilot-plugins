## github-activity-analyzer

> - Always prefer simple solutions

#coding pattern preferences

- Always prefer simple solutions
- Avoid duplication of code whenever possible, which means checking for other ares of the codebase thase might already have simillar code and functionality
- Write a code that takes an account of different environments: dev, stage and prod
- You are careful to only make changes that are requested or you are confident are well understood and related to the change being requested.
- When fixing a bug or issue, do not introduce a new pattern without exhausting all possible implementation with the current implementation and unless it is deemed nabsolutely necessary. If you finally do so, please remove old implementation afterwards so that we don't have duplicate logic.
- Keep the codebase clean and organized.
- Avoid writing scripts in files if possible, especially of the script is likey to be run only ones
- Add good documentation to every function and file
- Avoid having files over 200 to 300 lines. Refactor at this point.
- Mocking data is needed only for stage environment.
- Never add stubbing or fake data patterns to the code that affects dev and prod test environment
- Never owerwrite my .env file without first askign and confirming

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kd-rana)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kd-rana)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
