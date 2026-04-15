## dax-orchestrator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DAX Catalog v1.0.0** is a production-ready AI-powered Power BI documentation system that transforms raw Power BI DAX metadata into business-friendly documentation using a **7-agent AI pipeline architecture**. The system processes Power BI INFO.VIEW exports and generates comprehensive stakeholder-ready documentation across any business domain with **domain-agnostic intelligence**.

### Key Features
- ✅ **7-Agent Pipeline**: Complete CSV → Domain → Parallel Analysis → Synthesis → Polish workflow
- ✅ **Domain-Agnostic**: Works reliably with any Power BI model without dataset-specific assumptions
- ✅ **Production-Ready**: Clean codebase, v1.0.0, comprehensive error handling and logging
- ✅ **Advanced DAX Linting**: Pattern-based code quality analysis for any DAX formula
- ✅ **Confidence Gating**: Intelligent fallbacks to generic domains when uncertain
- ✅ **Artifact Versioning**: Complete metadata tracking with schema versioning
- ✅ **Web Interface**: React frontend with real-time progress tracking
- ✅ **Stakeholder-Ready Output**: Immediate sharing without additional editing required 

## Essential Commands

### Development & Build
```bash
# Build the project (required for web server)
npm run build

# Development mode CLI
npm run cli <command>

# Run tests
npm run test

# Type checking and linting
npm run typecheck
npm run lint:check
```

### CLI Usage Patterns
```bash
# Full 7-agent pipeline analysis (recommended)
npm run cli orchestrate ./sample-data/test2

# Limit measures for faster testing
npm run cli orchestrate ./sample-data/test2 --max-measures 5

# Legacy single DAX analysis (still supported)
npm run cli analyze 'SUM(Sales[Amount])' --name 'Total Sales'

# File discovery and validation
npm run cli discover ./sample-data/test2 --preview

# Process single CSV (legacy compatibility)
npm run cli process-csv measures.csv --output documentation.md

# Generate sample test data
npm run cli create-samples
```

### Web Application
```bash
# Start both server and client (from web-app directory)
cd web-app && npm run dev

# Individual components
cd web-app && npm run server  # Express server (dynamic port, starts at :3001)
cd web-app/client && npm start  # React client on :3000

# Production mode (build + serve)
cd web-app && npm run build && npm start  # Built React app + server
```

## Architecture Overview

### 7-Agent AI Pipeline Architecture
The core innovation is a sophisticated agent orchestration system that processes Power BI data through specialized AI agents:

**Pipeline Flow:**
```
Agent 0 (CSV Parser) → Agent 1 (Domain Classifier) → [Agents 2,3,4 Parallel] → Agent 5 (Report Synthesis) → Agent 6 (Content Polish)
```

**Agent 0 (CSV Parser)**: Data ingestion and validation
- Parses INFO.VIEW exports with dynamic content detection
- Schema enrichment with format string inference and role detection
- Filters out INFO.VIEW metadata tables (Measures_Table, Column_Table, etc.)
- Builds dynamic ID mappings for relationships

**Agent 1 (Domain Classifier)**: Business context identification with confidence gating
- Identifies industry domain (transportation, finance, compliance, etc.)
- **NEW**: Falls back to "Analytics Model" with generic stakeholders when uncertain
- Determines key stakeholders and regulatory context
- Sets context for subsequent agents

**Agents 2, 3, 4 (Parallel Processing)** with concurrency limiting:
- **Agent 2**: Business glossary and terminology extraction
- **Agent 3**: Data architecture transformation to business processes  
- **Agent 4**: DAX analysis with **NEW** generic DAX linting and business explanations

**Agent 5 (Report Synthesis)**: Unified documentation generation
- Integrates heuristic measure descriptions for complete coverage
- Merges parallel agent outputs into coherent documentation

**Agent 6 (Content Polish & Review)**: Publication-ready content creation
- Comprehensive grammar, style, and readability review
- Professional tone optimization for stakeholders
- Single-page formatted output for immediate sharing
- Quality assurance and consistency validation

### Domain-Agnostic Improvements
- **DAX Linting**: Pattern-based technical analysis (division operators, AVERAGEX issues, etc.)
- **Confidence Gating**: Generic fallbacks prevent hallucination
- **Schema Enrichment**: Smart format detection and role inference
- **No-Fabrication Guardrails**: All agents instructed not to invent domain-specific content
- **Enhanced Logging**: Real-time Claude API call timing and progress visibility

### Key Components

**AgentOrchestrator** (`src/agent-orchestrator.ts`)
- Coordinates the 7-agent pipeline with concurrency limiting (p-limit)
- Manages shared Claude client for API efficiency
- Handles parallel processing of Agents 2, 3, 4
- **NEW**: Artifact versioning with metadata (schema v1.0.0)
- Provides progress callbacks for web UI

**InfoViewParser** (`src/csv-parser.ts`)
- Robust CSV parsing with filename-agnostic detection
- **NEW**: Schema enrichment with format string inference
- Dynamic table/column ID mapping for relationships
- INFO.VIEW metadata filtering to exclude utility tables
- Handles Power BI relationship string parsing

**ClaudeClient** (`src/claude-config.ts`)
- Anthropic API integration with error handling
- **NEW**: Enhanced logging with request timing
- Retry logic and timeout management
- Shared across all agents for efficiency

**Domain-Agnostic Libraries** (`src/lib/`)
- `dax-lint.ts`: Generic DAX pattern analysis (technical, not domain-specific)
- `format-defaults.ts`: Smart schema enrichment and format inference
- `measure-heuristics.ts`: Model-agnostic measure classification
- `measure-enricher.ts`: Business context merging without domain assumptions

### Web Application Architecture

**Server** (`web-app/server.js`)
- Express server with file upload handling
- WebSocket support for real-time progress updates
- Agent orchestration integration
- Temporary file management

**Client** (`web-app/client/src/`)
- React application with file drop interface
- Smart file detection (content-based, not filename-dependent)
- Real-time progress tracking via WebSocket
- Results viewer with formatted documentation

## Critical Implementation Details

### CSV File Processing
The parser must handle any Power BI INFO.VIEW export format:
- Files can have any names (input1.csv, data.csv, etc.)
- Content-based detection determines file types
- Dynamic ID mapping resolves relationships between tables/columns
- INFO.VIEW metadata tables are automatically filtered

### Agent Context Passing
```typescript
interface AgentContext {
  measures?: any[];
  tables?: any[];
  columns?: any[];
  relationships?: any[];
  domain?: string;          // Set by Agent 1
  stakeholders?: string[];  // Set by Agent 1
  businessContext?: string; // User-provided context
}
```

### Data Quality Assurance
The system automatically filters INFO.VIEW metadata including:
- `Measures_Table`, `Column_Table`, `Relationship_table`, `Tables_table`
- Tables with `_TABLE` suffix
- Any table containing INFO.VIEW expressions

## Environment Requirements

### Required Environment Variables
```bash
ANTHROPIC_API_KEY=your_claude_api_key  # Required for AI analysis
```

### API Configuration
- Model: `claude-sonnet-4-20250514` (latest Sonnet 4)
- Default timeout: 60 seconds
- Max tokens: 2000-2500 depending on agent

## Testing Strategy

### Test Data Structure
```
sample-data/
├── test2/    # Sales Analytics domain (working dataset) - 7 measures, 6 tables
├── test3/    # Transportation domain - bike share analytics
├── test9/    # Empty dataset - tests domain fallback to "Analytics Model"
└── *.csv     # Legacy root-level exports (measures.csv, tables.csv, etc.)
```

### Test Commands
```bash
# Unit tests
npm run test

# Agent orchestration testing
npm run test:agents

# Legacy CLI testing
npm run test:legacy
```

## Web Deployment Considerations

### Build Dependencies
The web server depends on compiled TypeScript:
1. Run `npm run build` in main directory
2. Web server imports from `../dist/agent-orchestrator`
3. Any source changes require rebuild for web app

### File Upload Handling
- Supports drag-and-drop of any 4 CSV files
- Automatic content-type detection
- Temporary file cleanup after processing
- WebSocket progress updates during analysis

## Common Troubleshooting

### CSV Parser Issues
- Ensure all 4 file types present (measures, tables, columns, relationships)
- Check for INFO.VIEW metadata contamination
- Verify column headers match Power BI export format

### Agent Pipeline Failures
- Confirm ANTHROPIC_API_KEY is set
- Check API rate limits and timeout settings
- Review agent confidence scores for quality assessment

### Web Application Issues
- Ensure main project is built (`npm run build`)
- Check WebSocket connection on port 3003
- Verify file upload size limits (10MB default)

## Domain Intelligence

The system is **designed to be domain-agnostic** and has proven performance across:
- **Sales Analytics**: E-commerce revenue and customer insights (test2 dataset)  
- **Transportation**: Bike share operations (4.8M trips)
- **Compliance**: UK social housing fire safety regulations
- **Finance**: Any DAX-based financial reporting
- **Generic Analytics**: Falls back gracefully for unknown domains

**Domain-Agnostic Design Principles:**
- Agent 1 classifies domains but falls back to generic "Analytics Model" when uncertain
- All agents avoid inventing domain-specific terminology or business processes
- Heuristic analysis works with any DAX formula regardless of business context
- Technical linting focuses on DAX patterns, not business rules
- Confidence gating prevents hallucination across all domains

## File Organization Patterns

### Source Structure  
- `src/agents/`: Specialized AI agent implementations (0-6)
- `src/lib/`: Domain-agnostic utility libraries
- `src/helpers/`: Shared utilities (retry logic, etc.)
- `src/types.ts`: Core data type definitions
- `src/csv-parser.ts`: Power BI data ingestion
- `src/claude-config.ts`: API client management
- `src/agent-orchestrator.ts`: Pipeline coordination
- `docs/`: Organized documentation and test outputs
- `sample-data/`: Clean test datasets with output directories

### Output Patterns & Artifacts
The system generates comprehensive artifacts in `{dataset}/out/`:
- `final_report.json`: Complete analysis results with lint findings
- `meta.json`: Schema v1.0.0 metadata with generation timestamp
- `model_documentation.html`: Formatted HTML for immediate sharing
- `model_documentation.md`: Markdown for technical teams
- `model_kpis.csv`: Structured data export
- `synthesis.json`: Pre-polish intermediate analysis

**Each output includes:**
- Business-friendly language optimized for stakeholders
- Executive summaries with key insights and confidence scores
- Technical DAX analysis with generic pattern-based linting
- Domain-appropriate stakeholder identification (or generic fallback)
- Immediate sharing capability without additional editing

## Important Instructions

### Development Guidelines
When working in this repository:
- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User
- Always run `npm run build` after making changes to TypeScript files if the web application is being used
- When updating agent logic, maintain domain-agnostic principles and avoid dataset-specific assumptions
- When updating CSV parsing logic, ensure INFO.VIEW metadata tables are properly filtered out

### Production System Status
**DAX Catalog v1.0.0** is production-ready with:
- ✅ Clean, organized codebase with comprehensive cleanup completed
- ✅ Domain-agnostic architecture working across any Power BI model  
- ✅ All 7 agents functional with proper error handling and logging
- ✅ Advanced features: DAX linting, confidence gating, artifact versioning
- ✅ Web interface with real-time progress tracking
- ✅ Complete test coverage across multiple domains

### User Value Proposition
**"After using this tool, users can immediately share meaningful insights about their Power BI model with stakeholders without additional editing"**

**Users want to:**
1. **Share findings** - "Here's what our dashboard does" 
2. **Train users** - "Here's how to interpret these metrics"
3. **Make decisions** - "These are our key performance indicators"
4. **Improve dashboard** - "Here are optimization opportunities" 
5. **Document compliance** - "Here's our data governance"

**What users can DO with the output:**
1. Share with stakeholders ("Here's what our dashboard does")
2. Train new team members ("Here's how to use our reports")
3. Document for compliance ("Here's our data model documentation") 
4. Optimize dashboard ("Here's what we can improve")
5. Make business decisions ("Here are our key metrics and what they mean")

The system delivers **immediate stakeholder value** across any business domain without requiring domain expertise or additional editing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kittypony123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
