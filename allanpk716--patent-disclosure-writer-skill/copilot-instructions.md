## patent-disclosure-writer-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication

- **使用中文回答问题和进行所有沟通**

## Project Overview

This is a **Claude Code skill** that automatically generates Chinese patent disclosure documents (专利申请技术交底书) compliant with the IP-JL-027 standard. Users provide an innovative idea, and the skill performs patent retrieval, technical analysis, figure generation, and document writing.

**Key capabilities**: Patent search, technical analysis, Mermaid diagram generation, Markdown/DOCX output, checkpoint/resume support, and optional 5-expert review system.

## Entry Points

### Slash Commands
- `/patent` - Main patent generation command with checkpoint/resume and selective regeneration
- `/patent-md-2-docx` - Convert Markdown disclosure to Word format
- `/patent-update-diagrams` - Intelligently supplement missing diagrams in existing documents

### Skill Triggers
The skill auto-triggers on keywords: "写专利交底书", "生成专利文档", "专利申请", "技术交底书", "申请发明专利"

## Agent Architecture

The system uses **31 specialized subagents** organized in three categories:

### Generation Agents (01-10) - Sequential Chapter Generation
Each agent generates a specific chapter of the patent disclosure:

| Agent | Chapter | Output File |
|-------|---------|-------------|
| title-generator | 1. Invention Name | 01_发明名称.md |
| field-analyzer | 2. Technical Field | 02_技术领域.md |
| background-researcher | 3. Background Technology | 03_背景技术.md |
| problem-analyzer | 4.(1) Technical Problems | 04_技术问题.md |
| solution-designer | 4.(2) Technical Solution | 05_技术方案.md |
| benefit-analyzer | 4.(3) Beneficial Effects | 06_有益效果.md |
| implementation-writer | 5. Implementation | 07_具体实施方式.md |
| protection-extractor | 6. Protection Points | 08_专利保护点.md |
| reference-collector | 7. References | 09_参考资料.md |
| document-integrator | Document Integration | 专利申请技术交底书_[name].md |

### Document Processing Agents (11-21)
Handle diagram generation, DOCX conversion, validation, and figure management.

### Review System Agents (22-31) - Optional Expert Review
**5-expert team** with weighted voting (7 total votes):

| Role | Weight | Responsibility |
|------|--------|----------------|
| Senior Technical Expert | 2 votes | Technical feasibility, innovation |
| Technical Expert | 1 vote | Technical details, implementation |
| Legal Expert | 1 vote | Legal compliance |
| Senior Patent Agent | 2 votes | Protection scope, strategy |
| Patent Agent | 1 vote | Document formatting |

**4-stage review process**: Pre-review → Mid-review 1 → Mid-review 2 → Final review

**Voting thresholds**: Major issues ≥6/7 votes, Minor issues ≥5/7 votes

## Execution Modes

### Review Mode (Recommended for important patents)
Chapters generated in stages with expert validation after each stage:
- Stage 1: Chapters 01-02 → Pre-review
- Stage 2: Chapters 03-05 → Mid-review 1
- Stage 3: Chapters 06-07 → Mid-review 2
- Stage 4: Chapters 08-09 → Final review

### Quick Mode (For drafts/internal use)
All chapters 01-09 generated sequentially without review, then integrated.

## MCP Service Dependencies

The skill requires these MCP services configured in `~/.claude/settings.json`:

| Service | Purpose | API Required |
|---------|---------|--------------|
| web-search-prime | Web search | Zhipu AI API |
| web-reader | Web content extraction | Zhipu AI API |
| google-patents-mcp | Patent search via SerpApi | SerpApi API |
| exa | Technical document search | Exa API |

## Validation Scripts

Located in `.claude/skills/patent-disclosure-writer/scripts/`:

```bash
# Validate disclosure completeness
python scripts/validate_disclosure.py --dir .

# Validate Mermaid diagram syntax
python scripts/validate_mermaid.py --dir . --verbose

# Check figure numbering continuity
python scripts/check_figures.py --dir .

# Check generation status
python scripts/state_manager.py --status

# Check review status
python scripts/review_state_manager.py --status pre1
```

### Exit Codes
- `validate_disclosure.py`: 0=success, 10=failure
- `validate_mermaid.py`: 0=success, 11=failure
- `check_figures.py`: 0=continuous, 12=gaps found

## Patent Types

| Type | Innovation Requirement | Review Period | Protection Term |
|------|------------------------|---------------|-----------------|
| Invention Patent | Outstanding substantive features & significant progress | 2-3 years | 20 years |
| Utility Model Patent | Substantive features & progress | 6-12 months | 10 years |

## Template System

- **Markdown template**: `templates/IP-JL-027(A／0)专利申请技术交底书模板.md` - Standard Chinese patent disclosure template
- **DOCX template**: Used for final Word document conversion

Template follows the IP-JL-027 standard with sections:
1. Invention Name
2. Technical Field
3. Background Technology
4. Invention Content (Problems, Solutions, Benefits)
5. Implementation
6. Key Protection Points
7. References

## State Management

- **Generation state**: Managed by `state_manager.py` - tracks chapter completion, supports checkpoint/resume
- **Review state**: Managed by `review_state_manager.py` - tracks review stage progress

State files are created in the working directory during generation.

## Key Architectural Patterns

### Checkpoint/Resume Support
The system detects existing chapter files and offers options:
- Continue from last checkpoint
- Regenerate specific chapters
- Start fresh

### Dispute Resolution
When expert votes fail thresholds:
1. Dispute report generated
2. Majority and minority opinions shown
3. Recommended solution provided
4. User chooses final approach

### Diagram Generation
- Mermaid format embedded in Markdown
- 12 standard diagram types (flowcharts, sequence diagrams, architecture diagrams, protocol format diagrams)
- Automatic diagram validator checks syntax

### Script Editing
- **修复脚本时优先在原文件上编辑，非必需不新建脚本**

## Windows Compatibility

- **开发环境为 Windows 系统**
- 所有脚本使用 UTF-8 编码
- Python 脚本使用 `python` 命令（非 `python3`）
- **BAT 脚本不允许使用中文**（避免编码问题）
- 路径分隔符使用反斜杠 `\` 或正斜杠 `/` 均可

## Documentation Structure

- `README.md` - Main documentation with execution flow diagrams
- `docs/review-system-quickstart.md` - Review system guide
- `.claude/skills/patent-disclosure-writer/references/configuration.md` - MCP configuration
- `.claude/skills/patent-disclosure-writer/references/agents.md` - Agent documentation
- `.claude/skills/patent-disclosure-writer/references/troubleshooting.md` - Troubleshooting guide

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2025-01-15 | Added 5-expert review system with 4-stage review, voting, dispute resolution |
| 1.1.0 | 2025-01-14 | Added validation scripts, state management, evolution analysis |
| 1.0.0 | 2025-01-14 | Initial version, invention and utility model patent support |

---
> Source: [allanpk716/patent-disclosure-writer-skill](https://github.com/allanpk716/patent-disclosure-writer-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
