## classai

> Apply these rules when making changes to the project


Update this rule if user requested changes to the project requirement, etc.
flowchart TD
    Start[Project Start]
    Auth[User Authentication and Role Management]
    Security[Security & Compliance\nHTTPS, AES, COPPA, GDPR, FERPA]
    RoleSelect{Select Role}
    Student[Student Interface\nGuided Sessions, Interactive Responses,\nReal-time Feedback, Visual Progress]
    Parent[Parent/Teacher Interface\nCurriculum Settings, Dashboards, Analytics,\nNotifications, Customization]
    AI[AI Modules Cluster]
    LessonGen[Lesson Generator]
    Socratic[ Socratic Engine]
    Narration[ Narration Assistant]
    Writing[ Writing Coach]
    Latin[ Latin Tutor]
    Content[Content Sourcing & Management\nAutomated Scraping, Indexing, Categorization,\nManual Review]
    Analytics[Analytics & Dashboard\nKPIs and Weekly Digests]
    TechStack[Tech Stack\nReact, Node.js, Firebase, OpenAI API, Hosting]

    Start --> Auth
    Auth --> Security
    Auth --> RoleSelect
    RoleSelect -- Student --> Student
    RoleSelect -- Parent --> Parent
    Student --> AI
    Student --> Analytics
    Parent --> Analytics
    AI --> LessonGen
    AI --> Socratic
    AI --> Narration
    AI --> Writing
    AI --> Latin
    Content --> LessonGen
    Content --> Socratic
    Content --> Narration
    Content --> Writing
    Content --> Latin
    TechStack --> Auth
    TechStack --> AI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cloudzombie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
