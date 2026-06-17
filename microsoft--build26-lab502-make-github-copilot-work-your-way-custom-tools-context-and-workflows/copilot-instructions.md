## build26-lab502-make-github-copilot-work-your-way-custom-tools-co

> This repository is the Microsoft Build 2026 LAB502 content repo for **Make it your own: build, share, and customize with GitHub Copilot**. Treat it as an attendee-facing hands-on lab plus supporting source code for the Space Invaders community experience.

# Project Guidelines

This repository is the Microsoft Build 2026 LAB502 content repo for **Make it your own: build, share, and customize with GitHub Copilot**. Treat it as an attendee-facing hands-on lab plus supporting source code for the Space Invaders community experience.

## Repository Context

- `GUIDANCE.md` still exists, so the Build 2026 setup workflow is not finalized. When a content creator asks for help preparing, refining, or finalizing the repo, read `GUIDANCE.md` and follow its three-phase model.
- This repo is no longer an empty template. Use the existing lab content as the source of truth, especially `docs/00-intro.md`, the numbered modules in `docs/`, `src/README.md`, and `src/community-hub/README.md`.
- The root `README.md` may still contain template placeholders. If asked to update it, infer the session identity from the repo name and lab docs: this is `LAB502`, a hands-on lab, not `BRKXXX`.
- Do not fabricate missing session metadata. Ground titles, descriptions, outcomes, technologies, and next steps in repository content or ask the creator to confirm.

## Content Organization

- `docs/` contains the learner-facing lab modules. Keep the numbered module order, the checkbox-based step style, and the previous/next navigation links intact when editing.
- `docs/assets/` contains lab screenshots and diagrams. Reference existing assets from the relevant module instead of moving them unless the content structure changes.
- `src/game-samples/` contains self-contained Space Invaders sample games and preview images. Keep game samples browser-openable as standalone HTML files unless the user explicitly asks for another structure.
- `src/plugins/` contains Copilot plugin packages used by the lab.
- `src/community-hub/` contains the ASP.NET Core Community Hub service, tests, deployment templates, REST endpoints, and Model Context Protocol endpoint. Follow `.github/community-hub.instructions.md` for detailed rules in that subtree.

## Editing Rules

- Preserve attendee-facing wording and product branding. Write product names in full in docs, for example `Visual Studio Code` instead of `VS Code`.
- When reorganizing source or reference material, copy existing content verbatim unless the user asks for rewriting.
- Never commit secrets, API keys, credentials, connection strings with passwords, storage keys, or SAS tokens. Use environment variables or placeholder values.
- Do not modify `LICENSE`, `LICENSE-DOCS`, `CODE_OF_CONDUCT.md`, or `SECURITY.md`.
- Do not add large binary files such as PowerPoint decks, videos, or recordings. Links are fine.
- Use the Microsoft Learn MCP Server when adding or refreshing Microsoft Learn resource links.

## Build and Test

- For Community Hub changes, work from `src/community-hub/`.
- Restore/build/test with the .NET solution when relevant: `dotnet restore CommunityHub.slnx`, `dotnet test CommunityHub.slnx`, and `dotnet run --project CommunityHub/CommunityHub.csproj`.
- Prefer focused unit tests in `CommunityHub.Tests/` for narrow changes. Treat `CommunityHub.IntegrationTests/` as SQL-backed integration coverage that may require additional environment setup.
- For `src/game-samples/*.html`, opening the file in a browser should be enough to run the sample.

## Issue Support

- If the user asks for help filing an issue or reports a problem that should be tracked, discover available templates in `.github/ISSUE_TEMPLATE/` before choosing a template.
- Discover labels with `gh label list` before applying them. Do not hardcode template names or labels.

---
> Source: [microsoft/Build26-LAB502-make-github-copilot-work-your-way-custom-tools-context-and-workflows](https://github.com/microsoft/Build26-LAB502-make-github-copilot-work-your-way-custom-tools-context-and-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
