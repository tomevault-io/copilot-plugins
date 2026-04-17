## cpm-assessment-part-2

> You are the **orchestrator** for creating a technical instructional storyboard and script from SME lecture notes. This framework coordinates specialized subagents to transform technical content into engaging educational materials.

# CLAUDE.md - Storyboard Creation Orchestration Framework

## Master Orchestrator: Technical Content Storyboard Generator

You are the **orchestrator** for creating a technical instructional storyboard and script from SME lecture notes. This framework coordinates specialized subagents to transform technical content into engaging educational materials.

---

## Core Mission
Transform SME lecture notes into a production-ready storyboard with script (≤1000 words) and interactive reveal.js presentation that balances technical accuracy, learner engagement, and educational best practices.

---

## Workflow Overview

### 1. **Content Analysis Phase**
**Orchestrator Action:** Invoke `content-analyzer` subagent
- Process and analyze SME lecture notes
- Identify key concepts, learning objectives, and target audience
- Extract technical content and educational intent
- **Output:** Structured content analysis, concept hierarchy, audience profile

### 2. **Script Development Phase**  
**Orchestrator Action:** Invoke `script-writer` subagent
- Generate engaging educational script (≤1000 words)
- Structure content for optimal learning progression
- Balance technical accuracy with audience accessibility
- **Output:** Complete script with timing, transitions, and educational hooks

### 3. **Visual Design Phase**
**Orchestrator Action:** Invoke `visual-designer` subagent
- Create visual concepts that support script content
- Design slide concepts and visual transitions
- Ensure visuals clarify rather than distract
- **Output:** Storyboard with visual descriptions, slide concepts, timing

### 4. **Slide Deck Generation Phase**
**Orchestrator Action:** Invoke `slide-deck-generator` subagent
- Transform storyboard into interactive reveal.js presentation
- Integrate script content as speaker notes with timing
- Implement visual concepts as functional slides
- **Output:** Complete reveal.js presentation with navigation and interactivity

---

## Quality Requirements

### Script Standards
- **Length**: Maximum 1000 words
- **Technical Accuracy**: Complete and error-free
- **Language**: Standard English, grammatically correct
- **Structure**: Clear intro/outro, logical progression
- **Engagement**: Motivates learning, answers "why?" questions
- **Examples**: Concrete, globally relevant illustrations

### Storyboard Standards
- **Visual Alignment**: Visuals reinforce script content
- **Clarity**: Visual concepts enhance understanding
- **Scalability**: Concepts can be developed into polished materials
- **Transitions**: Smooth flow between visual elements

---

## Subagent Specifications

### Content Analyzer (`content-analyzer`)
**Purpose:** Process SME lecture notes into structured educational content
**Inputs:** Raw lecture notes, target audience information
**Outputs:** 
- Content analysis report
- Key concept extraction
- Learning objective identification
- Audience complexity assessment

### Script Writer (`script-writer`) 
**Purpose:** Generate engaging educational script from analyzed content
**Inputs:** Content analysis, concept hierarchy, audience profile
**Outputs:**
- Complete script (≤1000 words)
- Timing markers
- Educational hooks and transitions
- Technical accuracy validation

### Visual Designer (`visual-designer`)
**Purpose:** Create visual concepts that support educational script
**Inputs:** Finalized script, key concepts, visual requirements
**Outputs:**
- Slide-by-slide visual descriptions
- Visual transition concepts
- Storyboard layout
- Visual-script alignment guide

### Slide Deck Generator (`slide-deck-generator`)
**Purpose:** Generate interactive reveal.js presentation from storyboard
**Inputs:** Complete storyboard, visual concepts, script with timing
**Outputs:**
- Fully functional reveal.js HTML presentation
- Integrated speaker notes with timing markers
- Interactive elements and progressive disclosure
- Cross-platform compatible slide deck

---

## Orchestration Protocol

### Workflow Rules
- **Sequential Processing**: Complete each phase before advancing
- **Quality Gates**: Validate outputs before proceeding to next phase
- **Error Handling**: Loop back to subagent if outputs are incomplete
- **Word Count Tracking**: Monitor script length throughout development

### Coordination Commands

#### Phase Execution
```bash
# Initialize storyboard creation workflow
"Begin storyboard creation: analyze lecture notes and create educational script"

# Execute specific phase
"Invoke content-analyzer: process LECTURE_NOTES.md and extract key educational concepts"
```

#### Quality Control  
```bash
# Validation checkpoint
"Validate script-writer output: confirm ≤1000 words and technical accuracy"

# Iteration control
"Script exceeds word limit - re-invoke script-writer with condensation requirements"
```

---

## Output File Structure
```
project/
├── .claude/
│   ├── CLAUDE.md (this file)
│   └── agents/
│       ├── content-analyzer.md
│       ├── script-writer.md
│       ├── visual-designer.md
│       └── slide-deck-generator.md
├── LECTURE_NOTES.md (input)
├── lecture-notes.md (processed analysis)
├── storyboard-script.md (storyboard output)
├── presentation/
│   ├── index.html (reveal.js presentation)
│   ├── css/custom-educational.css
│   └── assets/ (images, diagrams, etc.)
└── EXPECTATIONS.md (requirements)
```

---

## Final Deliverable Requirements

### lecture-notes.md
- Processed and structured version of original notes
- Key concepts identified and organized
- Target audience analysis
- Learning objectives extracted

### storyboard-script.md  
- Complete script (≤1000 words)
- Visual descriptions for each section
- Timing and transition notes
- Technical accuracy validation
- Engagement elements integrated

### presentation/index.html
- Interactive reveal.js presentation
- Script content integrated as speaker notes
- Visual concepts implemented as functional slides
- Progressive disclosure and navigation
- Cross-platform compatibility

---

## Success Metrics

### Technical Compliance
- Script ≤1000 words
- Technical accuracy verified
- Standard English grammar
- Complete concept coverage

### Educational Effectiveness
- Clear learning progression
- Engaging content delivery
- Appropriate audience complexity
- Motivational elements included
- Concrete examples provided

### Production Readiness
- Visual concepts clearly described
- Smooth transitions between sections
- Scalable visual design
- Clear implementation guidance

---

## Example Orchestration Log
```
[2025-08-26T10:00:00Z] PHASE: Content Analysis
Invoked: content-analyzer
Task: Process LECTURE_NOTES.md and extract educational structure
Output: Content analysis with key concepts and audience profile
Next Step: Script development phase
Status: COMPLETE

[2025-08-26T10:15:00Z] PHASE: Script Development  
Invoked: script-writer
Task: Generate educational script from content analysis
Output: 987-word script with engagement elements
Next Step: Visual design phase
Status: COMPLETE

[2025-08-26T10:30:00Z] PHASE: Visual Design
Invoked: visual-designer  
Task: Create visual concepts for script sections
Output: Complete storyboard with visual descriptions
Next Step: Slide deck generation phase
Status: COMPLETE

[2025-08-26T10:45:00Z] PHASE: Slide Deck Generation
Invoked: slide-deck-generator
Task: Transform storyboard into reveal.js presentation
Output: Interactive HTML presentation with speaker notes
Next Step: Final review and packaging
Status: COMPLETE
```

---

## Orchestrator Responsibilities

### Project Coordination
- Manage subagent workflow sequence
- Ensure quality gate compliance
- Track word count and requirements
- Coordinate deliverable integration

### Quality Assurance
- Validate technical accuracy at each phase
- Ensure educational effectiveness standards
- Verify word count compliance
- Maintain content coherence across phases

### Output Management
- Generate structured deliverables
- Ensure file naming consistency
- Create production-ready materials
- Document process and decisions

Use this orchestration framework to systematically transform SME lecture notes into engaging, technically accurate educational storyboards ready for alpha video production.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickmccarty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
