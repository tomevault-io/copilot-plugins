## mario-ai-controlcenter

> Build Mario Wangen's professional website as an interactive, evidence-based portfolio dashboard for AI Infrastructure, MLOps, GenAI systems, and senior software engineering.

# AGENTS.md — Mario AI Control Center

## Project mission
Build Mario Wangen's professional website as an interactive, evidence-based portfolio dashboard for AI Infrastructure, MLOps, GenAI systems, and senior software engineering.

This is not a generic portfolio. It should feel like an **AI Engineering Control Center**: curated, searchable, filterable, calm, senior, and production-minded.

Primary audience:
- Recruiters hiring for AI Infrastructure / MLOps / GenAI engineering roles
- Hiring managers and engineering leaders
- Technical interviewers who want evidence, not claims

Primary positioning:
> Senior Software Engineer building production-ready AI systems across AI Infrastructure, MLOps, GenAI, and platform engineering.

Secondary positioning:
> 20+ years software engineering experience, now focused on production AI systems, ML infrastructure, model deployment, RAG systems, agents, and engineering reliability.

## Product principle
Every claim should be backed by evidence.

Avoid fake skill bars like "Python 95%". Instead, show:
- projects using the skill
- insights/articles connected to the skill
- current/recent usage
- architecture decisions
- GitHub/LinkedIn proof links

## Design direction
Visual style:
- Stripe / Linear / Vercel inspired
- clean, modern, minimal
- dashboard-like, not playful
- light theme should feel premium and spacious
- dark theme should feel technical but not hacker-ish
- subtle animation only; no gimmicks

Desired reaction:
> "I immediately want to click around."

The homepage should feel interactive even before the user clicks anything.

## Information architecture
V1 should contain:
- Homepage dashboard
- Search and skill filters
- Featured projects
- Prominent recent insights
- Capability matrix
- Evidence-backed skills
- Builder index / stats
- Timeline
- Project detail views/pages
- Insight detail views/pages

External links are secondary proof buttons. Do not send recruiters away too early.

Correct flow:
Homepage -> internal project/insight detail -> external proof link to GitHub or LinkedIn.

Incorrect flow:
Homepage -> direct GitHub/LinkedIn jump with no explanation.

## Content sources
Use structured content files as the source of truth. Prefer editing data objects over hardcoding content into components.

Suggested content types:
- projects
- insights
- skills
- capabilities
- timeline events
- stats
- links

Every project should support:
- title
- slug
- year
- status
- category
- summary
- problem
- what was built
- architecture notes
- tech stack
- evidence tags
- related insights
- links: GitHub, LinkedIn, demo, article, CV where available

Every insight should support:
- title
- slug
- date/year
- format: LinkedIn post, article, deep dive, carousel, case study
- summary
- core idea
- why it matters
- related skills
- related projects
- original link

## Current known projects
Include and refine these projects:
- Production MLOps Pipeline / AI Infrastructure Platform
- PULSE ContentAgent
- RAG Knowledge System
- StartupCoach
- InterviewCoach
- EasyCover AI
- AI LinkedIn Content Writer
- Django + React Mini LMS with AI assistant
- LiveCV
- Atlas / MLOps Showcase

## Current known skills and tags
Prioritize these tags:
- MLOps
- AI Infrastructure
- Production AI
- MLflow
- Kubernetes
- Terraform
- AWS
- PyTorch
- Docker
- CI/CD
- Python
- FastAPI
- RAG
- LangGraph
- LangChain
- CrewAI
- OpenAI APIs
- Vector Databases
- React
- TypeScript
- .NET / C#
- Azure
- DevOps
- Platform Engineering

## Current known insights / article ideas
Use these as initial insight cards:
- Four Gates Before a Model Touches Production
- Golden Snapshots: Catching the Model That Passes Metrics But Changes Answers
- CI Datasets: Testing Your Pipeline Without Testing Your Real Data
- Alias-Based Model Promotion
- We Automated Our ML Model Deployment and Removed the Bottleneck Person
- Debugging NaN Loss in FasterRCNN: The Zero-Annotation Image Problem
- OOM on a vGPU: Why Your VRAM Math Is Wrong for Large Medical Images
- DVC + MLflow: The Two-Command Reproducibility Guarantee
- Why Pixi Instead of Conda for Reproducible ML Environments
- Git Submodules in a Python ML Monorepo
- PULSE: Turning Unpredictable Intelligence Into a Reliable System

## Tone and copywriting
Voice:
- calm
- senior
- precise
- technical but readable
- confident without hype
- no "AI enthusiast" language
- no inflated metrics that cannot be proven

Preferred phrases:
- production-ready AI systems
- AI infrastructure
- MLOps pipelines
- model promotion
- validation gates
- reproducible ML workflows
- reliable systems after the demo
- evidence-backed engineering

Avoid:
- "passionate AI enthusiast"
- "rockstar"
- "ninja"
- over-selling
- unverifiable percentages
- generic portfolio copy

## Build constraints
- Keep the app static for now.
- No backend in V1.
- No CMS in V1.
- No database in V1.
- No auth.
- No unnecessary complexity.
- Use accessible semantic markup.
- Keep performance good.
- Make the content easy to edit.

## Suggested technical direction
Current project stack:
- React
- TypeScript
- Vite
- CSS or Tailwind-style utility classes
- Framer Motion if already installed / useful
- Lucide React icons where available

Important: do not import non-existing lucide brand icons like `Github` or `Linkedin`; use safe icons or text buttons unless verified.

## Immediate next tasks
1. Run the project and fix any blank-screen/runtime errors.
2. Improve homepage visual hierarchy: make Recent Insights more prominent.
3. Ensure light theme looks premium, not washed out.
4. Improve detail drawer/page content structure.
5. Add realistic content from the known projects and insights.
6. Add clear external proof buttons for GitHub, LinkedIn, CV, demo.
7. Improve responsive layout.
8. Prepare for Vercel deployment.

## Quality bar
The site is good enough for V1 when:
- a recruiter understands Mario's positioning in under 10 seconds
- technical depth is visible without reading everything
- filtering by skill makes the portfolio feel useful
- projects and insights reinforce each other
- the site feels more like a professional engineering dashboard than a generic portfolio template

---
> Source: [daytona675r/mario-ai-controlcenter](https://github.com/daytona675r/mario-ai-controlcenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
