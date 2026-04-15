## ai-engineering-journey

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **8-week AI engineering learning journey** (Dec 28, 2025 - Feb 22, 2026) where Ankur transitions from a director role back to hands-on coding. The focus is building production-ready AI projects that demonstrate mastery of embeddings, vector databases, RAG systems, MCP servers, and autonomous agents.

**Key Principle**: Build real projects, not tutorials. Each week culminates in a shipped, portfolio-worthy project.

## Learning Philosophy & Context

**Ankur's Learning Style**:
- Prefers world view first, then deep dives
- Learns by asking questions and exploring connections
- Thrives on understanding the big picture before implementation details
- Values learning depth over speed

**Important**: When working with code, ensure explanations are thorough and technical. Ankur has strong software engineering background (director level) but hasn't coded hands-on for a while.

## Week 1 Current Progress (Dec 28-29, 2025)

### Day 1 - Completed ✅
- ✅ Claude Code Quickstart (30+ min) - Learned commands, agents, permission modes
- ✅ Stack Overflow: Intuitive Intro to Embeddings (5+ hours including deep dives)
  - Went deep on RNNs, LSTMs, Transformers (exceeding initial plan)
  - Validated Ankur's learning style - rabbit holes = deep understanding
- ✅ Updated CLAUDE.md with all 5 specialized agents

### Day 2 - In Progress 🔄
- ✅ **Chose depth**: "Deep Dive into LLMs like ChatGPT" (3.5 hours) over simpler intro
  - **Checkpoint 1**: Complete Pretraining Flow
    - Tokenization via BPE: 256 bytes → 50k tokens
    - 12-layer transformer with attention + softmax
    - End-to-end example: "The big dog ran" → predict "fast"
    - Connected Day 1 insights (RNNs/LSTMs/attention) to transformers
  - **Checkpoint 2**: Post-Training - The Real Revolution
    - Base model = "internet document simulator"
    - Post-training = teaches to "think like helpful human labeler"
    - InstructGPT paper (2022): quality labeling > quantity
    - Modern evolution: LLM-assisted synthetic data (2023-2025)
    - Concrete examples: base vs. instruct on harmful requests
- ✅ Created running notes file (`day2-deep-dive-notes.md`, 400+ lines with all resource links)
- Still to complete: Remaining Deep Dive sections + 3 readings + mental model diagram

### Key Approach Established
- **Running notes during learning**: checkpoint-based, detailed
- **Daily log summary at end**: high-level synthesis
- **Validated learning style**: Deep dives > surface learning
- **Next**: Continue video + readings with better context, then hands-on Day 3

## 8-Week Project Roadmap

```
Week 1: Semantic Code Search (Embeddings fundamentals)
Week 2: Personal Knowledge Base (Vector database architectures)
Week 3: Multi-Tenant Search SaaS (Production patterns)
Week 4: Technical Documentation Assistant (RAG foundations)
Week 5: Adaptive RAG System (Advanced RAG patterns)
Week 6: MCP Server Suite (MCP protocol & servers)
Week 7: Research Agent (Autonomous agents)
Week 8: Personal AI Platform (Integration & portfolio)
```

Each week follows this pattern:
1. **Day 1-2**: Learning & mental model building
2. **Day 3**: Hands-on experiments with core concepts
3. **Day 4**: Core implementation
4. **Day 5**: Polish, testing, documentation
5. **Day 6-7**: Blog post, reflection, shipping

## Repository Structure

```
ai-engineering-journey/
├── PLAN.md                 # Master 8-week curriculum
├── CONTEXT.md              # Background on Ankur and learning style
├── PROGRESS.md             # Weekly progress tracker
├── README.md               # Quick links and overview
├── CLAUDE.md               # This file
├── week-01/
│   ├── README.md           # Week-specific plan and goals
│   ├── project/            # Code deliverables (semantic-code-search)
│   ├── notes/
│   │   ├── daily-log.md    # Day-by-day progress log
│   │   ├── day1-reading-notes.md    # Learning notes
│   │   ├── experiments.md  # Hands-on experiment findings
│   │   ├── architecture.md # System design
│   │   └── mental-model.md # Conceptual understanding
│   └── resources/          # Week-specific learning materials
├── week-02/ through week-08/ (same structure)
├── portfolio/
│   ├── blog-posts/         # Weekly technical blog posts
│   ├── videos/             # Demo videos
│   └── demos/              # Live demos and GIFs
└── resources/              # Global resources
    ├── reading/
    ├── tools/
    └── templates/
```

## Tech Stack by Week

**Core Languages**:
- Python (primary): Data processing, AI/ML, CLI tools
- TypeScript: MCP servers, tooling, APIs

**Vector Databases** (progressive complexity):
- Week 1: ChromaDB (local, simple)
- Week 2: Qdrant (scalable)
- Week 3: Pinecone (managed service)
- Additional: pgvector (PostgreSQL)

**LLM & Embedding APIs**:
- Claude API (primary for all projects)
- OpenAI (embeddings via text-embedding-3-small)
- Cohere (optional reranking)

**Frameworks**:
- Click (CLI frameworks)
- FastAPI (REST APIs)
- Next.js (frontend UIs)
- LangChain (for comparative study, not primary)

## Available Claude Code Agents

Claude Code uses specialized agents to handle different types of tasks efficiently:

### Agent Types

**general-purpose**: Handles complex multi-step tasks, code research, and exploratory work. Use when you need to perform comprehensive investigations across the codebase.

**Explore**: Fast agent specialized for exploring codebases. Use for finding files by patterns (e.g., `src/components/**/*.tsx`), searching code for keywords, or answering structural questions about the codebase. Specify thoroughness: "quick" for basic searches, "medium" for moderate, "very thorough" for comprehensive.

**Plan**: Software architect agent for designing implementation strategies. Use when planning complex features - it identifies critical files, considers trade-offs, and returns step-by-step plans.

**claude-code-guide**: Accesses official Claude Code and Claude Agent SDK documentation. Use when asking about Claude Code features, hooks, slash commands, MCP servers, settings, IDE integrations, or the Claude Agent SDK.

**fact-validator-agent**: Systematically verifies claims in documents against internet sources. Use when you need to fact-check technical claims, documentation accuracy, or validate information.

## Key Development Workflows

### Working with Day-by-Day Progress

When helping with daily work:
1. Check `week-*/README.md` for that day's goals
2. Review `week-*/notes/daily-log.md` for context
3. Look at `CONTEXT.md` and `PLAN.md` for broader understanding
4. Help document learnings in the appropriate notes file

### Building Projects

Each project follows this structure:
1. **Core logic**: Pure Python/TypeScript implementation first
2. **CLI/API layer**: Click CLI or FastAPI interface
3. **Testing**: Comprehensive test coverage
4. **Documentation**: README with examples and architecture diagrams
5. **Publishing**: Push to GitHub, blog post, social media

### Documenting Learnings

When helping with a task:
- Capture key insights in `week-*/notes/` files
- Explain design decisions in `notes/architecture.md`
- Document experimental findings in `notes/experiments.md`
- Update `daily-log.md` with actual accomplishments vs. planned goals

## Important Patterns to Know

### Mental Model First
Week 1 emphasizes this: **understanding before implementation**. Day 1-2 are pure learning (embeddings, vector databases, transformers). Day 3 involves hands-on experiments to validate understanding before Day 4-5 implementation.

This pattern repeats each week: understand the landscape → experiment with concepts → build the project.

### Hands-On Experiments Are Critical
Before building full projects, Ankur experiments with core concepts to build intuition:
- Week 1 experiments: embeddings generation, similarity metrics, ChromaDB, chunking strategies
- Each experiment is documented in `notes/experiments.md`
- Insights from experiments inform the project architecture

### Chunking Strategy Matters
Across all projects, the chunking strategy for splitting code/documents significantly impacts semantic search quality. This is learned in Week 1 and evolved in later weeks.

### Production Readiness Is Expected
By week 3, all projects must handle:
- Multi-tenancy (data isolation)
- Error handling and logging
- Performance optimization
- Security considerations (API keys, data privacy)

## Common Tasks You'll Help With

### Task: Help with Week N Project Implementation

When Ankur asks for help with a week's project:
1. Check `week-N/README.md` for the project spec and deliverables
2. Review any existing experiment notes in `week-N/notes/experiments.md`
3. Reference `PLAN.md` Week N section for learning objectives
4. Help implement, but ensure Ankur understands each decision
5. Update project README with architecture and usage

### Task: Help Debug/Refactor Code

1. Check if there are test failures: look in `week-N/project/` for test structure
2. Review the architecture doc: `week-N/notes/architecture.md`
3. Provide clear explanations of issues and solutions
4. Suggest improvements but respect existing decisions unless major refactor is needed

### Task: Create or Update Documentation

For blog posts (`portfolio/blog-posts/`):
- Technical deep-dives, not tutorials
- Include code snippets and architecture diagrams
- Focus on "why" and "how" not just "what"
- Include learnings and challenges

For README files:
- Quick start examples
- Architecture diagram
- API documentation
- Contributing guidelines (if applicable)

## Git & Publishing Workflow

### Daily Commits
- Commit at end of each dev day with progress update
- Use format: "Week N: [Component/Feature] - Brief description"
- Example: "Week 1: Embeddings generation - validate OpenAI API and dimension verification"

### Weekly Shipping
- Complete project by end of day (Day 5)
- Push to GitHub with comprehensive README
- Write blog post for `portfolio/blog-posts/`
- Update `PROGRESS.md` with completion status
- Share on LinkedIn/Twitter (brief mention)

### Commit Message Template
```
Week N: [Feature/Component] - Brief description

- Specific change 1
- Specific change 2

Learnings:
- Key insight 1
- Key insight 2

🤖 Generated with Claude Code

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

## When to Reference Key Files

- **Need to understand overall learning journey?** → PLAN.md
- **Need context on Ankur's background?** → CONTEXT.md
- **Need week-specific goals and breakdown?** → week-N/README.md
- **Need to see progress on tasks?** → week-N/notes/daily-log.md
- **Need to understand project architecture?** → week-N/notes/architecture.md
- **Need to see experimental findings?** → week-N/notes/experiments.md
- **Need overall progress tracking?** → PROGRESS.md

## Success Criteria

For Claude Code to be effective in this journey:
1. Help Ankur understand concepts deeply before implementation
2. Provide code that is well-structured, tested, and documented
3. Ensure each week's project is production-ready (not a prototype)
4. Capture learnings in the notes structure for reflection
5. Respect the learning style: world view first, then deep dives
6. Explain architectural decisions, not just provide code

## Special Considerations

### Week 1 (This Week)
- Focus on learning fundamentals first
- Hands-on experiments are critical before building
- Mental model synthesis is a deliverable
- Final project should be a real, usable tool

### Weeks 2-3
- Projects become more complex (production patterns, multi-tenancy)
- Architecture decisions become more important
- Reference earlier weeks' patterns and learnings

### Weeks 6-8
- Autonomous agents and MCP servers are complex
- Emphasis on system integration and reliability
- Portfolio quality increases significantly

## Resources Available

- PLAN.md: Week-by-week curriculum with resource links
- week-N/README.md: Each week has recommended readings
- CONTEXT.md: Background and learning philosophy
- Daily notes files: Track learnings and progress

## Final Note

This journey is about building production-quality projects while learning deeply. Claude Code should help Ankur not just complete tasks, but understand the "why" behind each decision. Every project should be something Ankur is proud to show as portfolio work.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ankurkakroo2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ankurkakroo2)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
