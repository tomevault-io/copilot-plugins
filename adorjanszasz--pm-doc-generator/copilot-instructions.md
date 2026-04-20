## pm-doc-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Product Management Documentation System

## System Overview
This workspace transforms product ideas and requirements into comprehensive product documentation:
1. Raw inputs (notes/transcripts/images/Jira) → Structured analysis → Multiple document formats
2. All voice guidelines and templates live in @context/
3. Agents handle document-type optimization

## Core Capabilities

### Input Processing
- Jira requirements → Feature roadmap entry, What's New doc, LinkedIn post, PRD update
- Raw notes/meeting minutes/transcripts/voice notes/images/screenshots → Jira requirements, PRD, Feature roadmap entry, What's New doc, LinkedIn post, PRD update

### Output Generation
- **Jira Requirements**: Well-structured user stories with acceptance criteria
- **Product Roadmap Entries**: External, strategic feature descriptions for stakeholder communication
- **What's New Documentation**: External, user-facing feature announcements
- **LinkedIn Posts**: Professional product update announcements
- **PRDs (Product Requirements Documents / Scope Documentation)**: Comprehensive product specifications, describing feature scope and boundary definitions

## Workflow Commands

### From Jira to Documentation
- `/parse-jira [ticket-id or paste content]` - Analyze Jira requirements and extract key information
- `/generate-roadmap` - Create roadmap entries from Jira requirements
- `/write-whats-new` - Generate What's New documentation from features
- `/create-linkedin-post` - Transform product updates into LinkedIn posts
- `/update-prd` - Update PRD with new requirements or changes

### From Raw Content to Jira
- `/extract-requirements` - Analyze raw notes/transcripts to identify requirements
- `/create-jira` - Generate Jira user stories from raw content or meeting notes
- `/create-prd` - Build comprehensive PRD from gathered requirements
- `/define-scope` - Create scope documentation from requirements

### Analysis and Planning
- `/analyze-feasibility` - Assess technical and business feasibility
- `/identify-dependencies` - Map feature dependencies and prerequisites
- `/stakeholder-summary` - Create executive summary for stakeholders

## Context Architecture
Always reference these for consistency:
- @context/pm-voice-samples.md - PM's writing style from presentations, blog posts, and Jira tickets
- @context/document-templates.md - Standard templates for different document types
- @context/company-guidelines.md - Company-specific terminology, branding, and style guides
- @context/jira-patterns.md - Standard patterns for Jira ticket structure

## Agent Specializations
- `jira-writer`: Expert in writing clear, actionable Jira user stories
- `prd-specialist`: Comprehensive PRD creation with technical depth
- `roadmap-formatter`: Strategic roadmap entries for stakeholder communication
- `whats-new-creator`: User-friendly release notes and feature announcements
- `linkedin-pm-repurposer`: Professional product updates for LinkedIn

## Voice Context System

The system learns from three types of PM voice samples:

1. **Presentation Voice**: How the PM communicates in slides and presentations
   - Strategic framing
   - Stakeholder language
   - Value proposition style

2. **Blog/Article Voice**: How the PM writes publicly
   - Thought leadership tone
   - Industry perspective
   - Storytelling approach

3. **Jira/Technical Voice**: How the PM writes requirements
   - Clarity and precision
   - Acceptance criteria style
   - Technical communication patterns

## Development Guidelines
- Never duplicate context information - always reference source files
- Maintain consistency across all generated document types
- Each agent should specialize in their document type's unique requirements
- Always ask for output type if not specified by user
- Quality gates should validate against company guidelines and templates
- Preserve PM's authentic voice across all document types

## File Management
- `/rawnotes/` - Unprocessed meeting notes, transcripts, voice notes, images
- `/jira-exports/` - Exported Jira tickets for processing
- `/research/` - Market research, competitive analysis, user feedback
- `/drafts/` - Work-in-progress documents
- `/wn-exports/` - Published What's New documentation
- Output files automatically saved with naming convention: `[doc-type]-[feature-slug]-[YYYY-MM-DD].md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adorjanszasz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
