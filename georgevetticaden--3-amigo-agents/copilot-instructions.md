## 3-amigo-agents

> Welcome to the **3 AI Amigos** workspace - a complete toolkit for orchestrating AI-powered Product Managers, UX Designers, and Claude Code to build production systems.

# CLAUDE.md

Welcome to the **3 AI Amigos** workspace - a complete toolkit for orchestrating AI-powered Product Managers, UX Designers, and Claude Code to build production systems.

## 🎯 What You're Looking At

This repository contains everything needed to implement the 3 AI Amigos methodology:
- **Agent configurations** to create your AI team
- **Technical patterns** that work for any domain  
- **Complete example** of a multi-agent health system
- **Setup guides** to get you started

## 🚀 Quick Navigation

### Want to implement 3 AI Amigos? Start here:
1. **README.md** - Your complete guide (start here!)
2. **agents/** - Copy these configs to set up your AI agents
3. **docs/po-document-checklist.md** - Documents you'll need

### Want to understand the patterns?
- **technical-guides/implementation-guide.md** - How to build multi-agent systems
- **technical-guides/multi-agent-patterns.md** - Orchestrator-worker architecture
- **docs/images/ai-amigos-diagram.png** - Visual overview

### Want to see a complete example?
- **use-cases/multi-agent-health-insight-system/** - Full health system implementation
  - Includes brand guidelines based on the mockups
  - Complete domain documentation
  - Visual examples in the PDF

## 📁 Repository Structure

```
3-AMIGO-AGENTS/
│
├── 🤖 agents/                    # COPY THESE to configure your agents
│   ├── pm-agent/                 
│   │   ├── config/              # PM Agent instructions
│   │   └── example-prompts/     # What to say to PM
│   ├── ux-agent/                
│   │   ├── config/              # UX Agent instructions  
│   │   └── example-prompts/     # What to say to UX
│   └── code-agent/              
│       └── config/              # CLAUDE.md template for your workspace
│
├── 📚 technical-guides/          # REUSABLE patterns (domain-agnostic)
│   ├── implementation-guide.md   # Multi-agent implementation
│   ├── multi-agent-patterns.md   # Based on Anthropic's research
│   ├── multi-agent-implementation-architecture.md # Backend structure patterns
│   └── streaming-patterns.md     # Real-time updates
│
├── 🏥 use-cases/                 # EXAMPLE: Complete health system
│   └── multi-agent-health-insight-system/
│       ├── [domain documents]    # Templates for your own domains
│       └── health-user-stories.pdf # UI/UX examples
│
├── 📖 docs/                      # HOW-TO guides
│   ├── demo-setup-guide.md      # Run the demo
│   ├── po-document-checklist.md # What docs you need
│   └── images/                  # Diagrams and visuals
│
└── README.md                     # START HERE - Complete guide
```

## 💡 The Big Picture

The 3 AI Amigos transforms development by orchestrating:

1. **Product Manager Agent** → Creates requirements & architecture
2. **UX Designer Agent** → Designs interfaces & experiences
3. **Claude Code** → Implements the complete system

You provide domain expertise, they build the system.

## 🎯 Your Next Steps

### Option 1: Build Something
1. Read **README.md** thoroughly
2. Set up your 3 AI agents using files in `agents/*/config/`
3. Create domain documents (use health example as template)
4. Run through the 5-phase process
5. Get a working system!

### Option 2: Explore First
1. Look at `docs/images/ai-amigos-diagram.png`
2. Browse the health example in `use-cases/`
3. Read technical guides to understand patterns
4. Then follow Option 1 when ready

## 🔑 Key Files You'll Need

### To Configure Agents:
- `agents/pm-agent/config/pm-agent-instructions.md`
- `agents/ux-agent/config/ux-designer-agent-instructions.md`  
- `agents/code-agent/config/minimal-claude-md-template.md`

### To Understand What to Submit:
- `agents/pm-agent/example/prompts/po-prompt-for-pm-agent.md` - Prompt + attached docs
- `agents/ux-agent/example/prompts/po-prompt-for-ux-agent.md` - Prompt + attached docs

### To See What Documents You Need:
- `docs/po-document-checklist.md`
- `use-cases/multi-agent-health-insight-system/` (examples)

## 📝 Important Notes

1. **Domain-Agnostic vs Domain-Specific**
   - `agents/` and `technical-guides/` work for ANY domain
   - `use-cases/` shows specific examples
   - Keep them separate!

2. **The Process is Sequential**
   - PM Agent first → UX Agent second → Claude Code last
   - Each builds on the previous outputs

3. **You're the Product Owner**
   - You provide domain knowledge
   - You guide the agents
   - You validate outputs

## 🎉 Success Looks Like

- ✅ 3 AI agents configured in Claude Desktop
- ✅ Your domain requirements ready
- ✅ PM creates comprehensive specs
- ✅ UX creates beautiful designs
- ✅ Claude Code builds working system
- ✅ System runs successfully!

---

**Ready?** Open README.md and let's build something amazing with your new AI development team!

*Remember: This isn't just about tools - it's about revolutionizing how we build software.* 🚀

---
> Source: [georgevetticaden/3-amigo-agents](https://github.com/georgevetticaden/3-amigo-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
