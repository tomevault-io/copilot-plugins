## cursor

> > **📝 File Purpose**: This file contains AI assistant rules for personal workspace management and organization.

# Personal Workspace - AI Assistant Rules

> **📝 File Purpose**: This file contains AI assistant rules for personal workspace management and organization.

## Workspace Overview
You are working in a personal workspace designed for organizing projects, notes, research, and ideas using AI assistance.

## Current Workspace Structure
```
Personal Workspace/
├── .cursorrules              # AI assistant configuration (this file)
├── README.md                 # Main documentation
├── scripts/                  # 🔧 Utility scripts and tools
│   ├── check_range.py        # Number range validation script
│   └── README.md             # Scripts documentation
└── Docs/                     # Personal documentation
    ├── Health/               # 🏥 Health tracking & medical records
    │   ├── Medical-Records/  # Tests, checkups, prescriptions
    │   ├── Fitness/          # Workouts, progress tracking
    │   └── Goals/            # Health objectives
    ├── Finance/              # 💰 Personal finance management
    │   ├── Budget/           # Monthly budgets, expenses
    │   ├── Investments/      # Portfolio, stocks, crypto
    │   └── Goals/            # Financial objectives
    ├── Learning/             # 🎓 Education & skill development
    │   ├── Courses/          # Online courses, notes
    │   ├── Books/            # Reading lists, summaries
    │   └── Skills/           # Skills practice, progress
    ├── Home/                 # 🏠 Home management
    │   ├── Projects/         # Renovations, improvements
    │   ├── Maintenance/      # Schedules, tasks
    │   └── Tech/             # Devices, smart home
    └── My Company Example/   # Example business structure template
```

## 🚨 CRITICAL RULE: Number Range Validation

**ALWAYS use the range checking script for numerical comparisons - NEVER do mental math for ranges:**

When you need to check if any number falls within a range:
1. **STOP** - Do not attempt to compare numbers mentally
2. **USE THE SCRIPT**: `python3 scripts/check_range.py [value] [min] [max] [value] [min] [max] ...`
3. **EXAMPLE**: `python3 scripts/check_range.py 185 125 200 92 70 100`
4. **TRUST THE RESULT**: Use the script output, not your own calculation

**Why this rule exists**: LLMs frequently make errors with numerical comparisons. The script provides 100% accurate mathematical results.

**Common use cases**:
- Health metrics (cholesterol, blood sugar, blood pressure)
- Budget categories (spending within planned ranges)
- Financial targets (investment performance ranges)
- Any situation requiring "is X between Y and Z?"

## General Guidelines
- Help organize and manage personal projects and documentation
- Maintain consistent structure and naming conventions  
- Assist with content creation, research, and analysis
- Provide helpful suggestions for workspace organization
- Use web search when research is needed
- Connect information across different areas (health, finance, learning, etc.)

## File Management
- Use clear, descriptive file and folder names
- Maintain logical organization structure
- Keep related materials grouped together
- Regular cleanup and organization of outdated content
- Always reference actual file paths when suggesting connections

## AI Assistant Behavior
- Be helpful and proactive in suggesting improvements
- Ask clarifying questions when tasks are unclear
- Provide context and explanations for recommendations
- Respect user preferences and working style
- **NEVER** perform numerical range comparisons without the script
- Document sources when conducting research
- Create cross-references between related content

## Integration Rules
- Link related information across Health, Finance, Learning, and Home categories
- Use examples from actual files when providing guidance
- Suggest automation opportunities with scripts
- Maintain consistency in documentation format
- Always verify numerical data with appropriate tools

---

*Personal workspace template - customize as needed*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivan121178) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
