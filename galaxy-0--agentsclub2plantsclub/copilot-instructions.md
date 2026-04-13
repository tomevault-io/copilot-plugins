## agentsclub2plantsclub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI course curriculum repository (AgentSClub2PlantSClub) focused on creating educational materials about AI agents and cognitive architectures. The project is primarily in Chinese and contains course designs, teaching resources, and content integration materials.

## Repository Structure

- `20250605/` - Contains materials for a specific course session on CoALA (Cognitive Architectures for Language Agents)
  - `coala_presentation.html` - Interactive HTML presentation with audio player
  - `coala.md` - Detailed 90-minute course curriculum and teaching plan
  - `语言智能体的认知架构.wav` - Audio file for the presentation
- `CHANGELOG.md` - Project change history following semantic versioning
- `README.md` - Basic project description

## Key Technologies

- **HTML/CSS/JavaScript**: The presentation is a standalone HTML file with embedded styles and interactive features
- **Audio Integration**: Custom audio player with speed controls and progress tracking
- **Responsive Design**: Presentation optimized for different screen sizes

## Architecture Notes

### CoALA Presentation (`coala_presentation.html`)
- Self-contained HTML presentation with 10 slides
- Features a custom audio player with speed controls (0.75x to 1.5x)
- Keyboard navigation (arrow keys, spacebar)
- Slide transition animations with CSS transforms
- Progress indicator and slide counter
- Dark theme with minimalist design

### Content Structure
- Bilingual content (primarily Chinese with some English headings)
- Covers cognitive architectures for language agents based on arXiv:2309.02427v3
- Educational material designed for PhD students and advanced learners

## Common Issues to Watch For

When working with the presentation file:
- Audio file path is hardcoded as `"语言智能体的认知架构.wav"` - ensure the audio file exists in the same directory
- Mixed Chinese/English content may need language consistency checks
- Animation timing depends on CSS transitions - be careful when modifying animation properties
- Audio player fallback logic uses hardcoded duration "07:25" if metadata loading fails

## Content Guidelines

- Maintain Chinese as the primary language for educational content
- Keep technical terms consistent with the CoALA framework terminology
- Preserve the academic tone and research paper references
- Ensure audio synchronization if modifying presentation timing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galaxy-0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Galaxy-0)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
