## automatic-poll-generator

> This is useful when you need to get an overview what's this project, for example when a new function development requested, a function change implementation requested, or when an overall information needed regarding this project.

This is useful when you need to get an overview what's this project, for example when a new function development requested, a function change implementation requested, or when an overall information needed regarding this project.

The project purpose is to provide an interface to select Jira tickets for product backlog refinement meetings and build a global PBR Queue, building a confluence page based on the queue, and provide a trustworthy scrum poker interface to iterate the PBR queue and estimate the tickets by a team. There is a fourth page that support building a preparation page for a sprint demo that is grouped by topics.

The project has 4 pages: Jira Tickets, PBR Planning (queue), Planning Poker and Sprint Demo planner.

This project built on a stack separated to 3 parts:
    - a backend in the /backend folder, it separates to two different side:
    -- a Python 3 project that responsible for user management
    -- a Node.js project that provides API for the frontend
    - a frontend in the /frontend folder, that is a React application

If you need info on running the project chekc the README.MD file.

This project uses WebSocket to poll user joins to the planning poker sessions, it is fundamentally using the global PBR queue.

Always use Vite and avoid to use Create React App! 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dil-bolahlautner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
